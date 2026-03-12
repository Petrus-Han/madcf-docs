# Deck Event Bus — Real-time State Sync

## Overview

Each deck maintains a persistent SSE connection from the frontend to PP backend. This connection serves as a **real-time event bus** for all deck-level state changes, eliminating the need for polling.

```
Arx (state change occurs)
  → POST /webhooks/deck-events  (to PP backend)
    → PP backend pushes via SSE
      → Frontend updates UI in real-time
```

## Architecture

### Components

```
┌──────────────┐   webhook    ┌──────────────┐   SSE (persistent)   ┌──────────────┐
│     Arx      │ ──────────→  │  PP Backend   │ ──────────────────→  │  PP Frontend  │
│              │              │              │                       │              │
│ - reaper     │              │ - event hub  │                       │ - EventSource│
│ - health     │              │ - per-deck   │                       │ - state mgmt │
│ - chat       │              │   SSE conns  │                       │ - UI update  │
│ - provider   │              │              │                       │              │
└──────────────┘              └──────────────┘                       └──────────────┘
```

### Data Flow

1. **Arx** detects a state change (agent destroyed, artifact created, etc.)
2. **Arx** sends a webhook POST to PP backend with event payload
3. **PP backend** looks up which deck(s) are affected, pushes event to their SSE connections
4. **PP frontend** receives the event and updates UI accordingly

## Event Types

### 1. `agent_status`

Agent lifecycle changes — online, offline, destroying.

**Trigger points in Arx:**
- `agent_service::destroy_instance()` → status: `offline`
- `create_agent` handler → status: `online`
- `health_probe` detects crash → status: `offline`
- `idle_reaper` timeout → status: `offline`

**Payload:**
```json
{
  "type": "agent_status",
  "agent_id": "abc-123",
  "status": "online | offline | destroying",
  "tulpa_name": "pp-planner",
  "reason": "idle_timeout | user_destroy | crashed | created"
}
```

**Frontend action:** Update status dot (green/gray), show toast if unexpected crash.

### 2. `artifact_synced`

Any phase's artifact was created or updated by the agent or user action.

**Phase artifacts:**

| Phase | artifact_key | Description |
|-------|-------------|-------------|
| gather | `audience` | Audience profile, key messages, meeting context |
| plan | `storyboard` | Cards, narrative arc, audience summary |
| generate | `slides` | Slide HTML/content, visual assets |
| preview | `presentation` | Final assembled presentation |

**Trigger points:**
- Agent uploads artifact via `arx_upload` tool
- PP BFF `syncArtifactsToDeck()` after chat stream
- User edits storyboard cards manually

**Payload:**
```json
{
  "type": "artifact_synced",
  "deck_id": "deck-456",
  "phase": "plan",
  "artifact_key": "storyboard",
  "data": {
    "cards": [...],
    "audience": { "name": "...", "profile": "..." },
    "narrative": "..."
  }
}
```

**Frontend action:** Update the corresponding panel based on `phase` + `artifact_key`.

### 3. `phase_changed`

Deck phase was advanced (by user action or agent recommendation).

**Trigger points:**
- PP BFF `POST /api/decks/:id/phase`
- Future: agent auto-advance when sufficient context gathered

**Payload:**
```json
{
  "type": "phase_changed",
  "deck_id": "deck-456",
  "from": "gather",
  "to": "plan"
}
```

**Frontend action:** Update Header phase tabs, transition preview panel.

### 4. `context_updated`

Context/memory file was added, edited, or deleted.

**Trigger points:**
- User uploads file via ChatPanel
- User edits memory in EditModePage
- Agent saves context via `arx_memory` tool
- PP BFF `POST /api/decks/:id/context`

**Payload:**
```json
{
  "type": "context_updated",
  "deck_id": "deck-456",
  "action": "added | updated | deleted",
  "key": "competitor-analysis.md",
  "content_type": "text/markdown",
  "actor": "user:alice | agent:pp-planner"
}
```

**Frontend action:**
- Refresh Sidebar context file list
- If edited by another user/tab → show notification: "Memory updated, reload?"
- If edited by self → silent refresh

**Agent sync:**
When context is updated by a user, the running agent's sandbox also needs the new files. PP backend calls Arx API to update the agent's files:

```
User edits memory
  → PP saves to DB
  → broadcast SSE: context_updated (other browsers refresh)
  → PUT /api/agents/:id/files (Arx API)
    → Arx provider writes file into sandbox + recompiles CLAUDE.md
```

The Arx provider handles all container-level operations internally. PP never calls Docker commands directly — it only uses Arx APIs.

### 5. `deck_updated`

Deck metadata changed (title, audience, meeting notes, etc.).

**Trigger points:**
- PP BFF `PATCH /api/decks/:id`
- Agent auto-rename on first message
- Post-stream artifact sync updates audience_json

**Payload:**
```json
{
  "type": "deck_updated",
  "deck_id": "deck-456",
  "fields": ["title", "audience_json"],
  "title": "Series A Pitch — AI Compliance"
}
```

**Frontend action:** Update Header title, PreviewPanel audience display.

## Multi-tab Sync

All event types are automatically broadcast to **every SSE connection** for the same deck. This provides multi-tab/multi-user sync out of the box:

- User A edits memory in Tab 1 → Tab 2 and User B both receive `context_updated`
- Agent creates storyboard → all tabs receive `artifact_synced`
- User A clears conversation → all tabs receive `agent_status: offline`

**Conflict handling:**
- **Same user, self-edit:** Silent refresh (no confirmation needed)
- **Other user/agent edit:** Show notification bar: "Memory updated by XX — Reload?"
- Events carry `actor` field to distinguish origin

## Implementation Plan

### Step 1: PP Backend — Event Hub

**File:** `PitchPilot/app/server/services/event-hub.mjs`

```javascript
// In-memory registry of SSE connections per deck
// Map<deckId, Set<Response>>
const connections = new Map();

export function addConnection(deckId, res) { ... }
export function removeConnection(deckId, res) { ... }
export function broadcast(deckId, event) {
  const conns = connections.get(deckId);
  if (!conns) return;
  const data = `data: ${JSON.stringify(event)}\n\n`;
  for (const res of conns) {
    res.write(data);
  }
}
```

### Step 2: PP Backend — SSE Endpoint

**Route:** `GET /api/decks/:deckId/events`

- Auth via JWT (query param `?token=xxx` since EventSource doesn't support headers)
- 30s heartbeat to keep alive through proxies
- Cleanup on disconnect

### Step 3: PP Backend — Webhook Receiver

**Route:** `POST /api/webhooks/arx`

- Auth via shared `ARX_SERVICE_KEY`
- Look up affected deck by `instance_id` → `agents_json`
- Broadcast to deck's SSE connections

### Step 4: Arx — Webhook Sender

**File:** `arx-services/src/webhook_service.rs`

Lightweight fire-and-forget HTTP POST to PP backend:

- `destroy_instance()` → `agent_status: offline`
- `record_start()` → `agent_status: online`
- `health_probe` crash → `agent_status: offline`

Configured via `ARX_WEBHOOK_URL` env var. Best-effort, non-blocking.

### Step 5: Arx — Agent File Update API

**Route:** `PUT /api/agents/:id/files`

Allows PP to push updated memory/context files into a running sandbox:

- Receives file key + content
- Provider writes to container filesystem
- Recompiles CLAUDE.md from instruction files

This keeps all container operations inside Arx's provider abstraction.

### Step 6: PP Backend — Internal Events

For events originating inside PP itself (no webhook needed):

- `POST /api/decks/:id/phase` → broadcast `phase_changed`
- `POST /api/decks/:id/context` → broadcast `context_updated`
- `PATCH /api/decks/:id` → broadcast `deck_updated`
- `syncArtifactsToDeck()` → broadcast `artifact_synced`

### Step 7: PP Frontend — useDeckEvents Hook

**File:** `src/hooks/useDeckEvents.ts`

```typescript
export function useDeckEvents(deckId: string | null, handlers: {
  onAgentStatus?: (status: string, reason: string) => void;
  onArtifactSynced?: (phase: string, key: string, data: any) => void;
  onPhaseChanged?: (from: string, to: string) => void;
  onContextUpdated?: (action: string, key: string, actor: string) => void;
  onDeckUpdated?: (fields: string[], data: any) => void;
}) {
  useEffect(() => {
    if (!deckId) return;
    const token = localStorage.getItem('pp_token');
    const es = new EventSource(`/api/decks/${deckId}/events?token=${token}`);
    es.onmessage = (e) => {
      const event = JSON.parse(e.data);
      switch (event.type) {
        case 'agent_status':
          handlers.onAgentStatus?.(event.status, event.reason);
          break;
        case 'artifact_synced':
          handlers.onArtifactSynced?.(event.phase, event.artifact_key, event.data);
          break;
        case 'phase_changed':
          handlers.onPhaseChanged?.(event.from, event.to);
          break;
        case 'context_updated':
          handlers.onContextUpdated?.(event.action, event.key, event.actor);
          break;
        case 'deck_updated':
          handlers.onDeckUpdated?.(event.fields, event);
          break;
      }
    };
    return () => es.close();
  }, [deckId]);
}
```

## Sequence Diagrams

### Agent Idle Timeout → Frontend Notification

```
Time 0:00  User sends last message
           └→ touch_activity() updates last_activity_at

Time 10:00 Arx idle_reaper fires
           └→ destroy_instance() via provider
              └→ POST /webhooks/arx { agent_status: offline, reason: idle_timeout }
                 └→ PP broadcast(deckId, { type: agent_status, status: offline })
                    └→ Frontend SSE → green dot turns gray
```

### User Clears Conversation

```
Frontend: clicks trash icon
  └→ DELETE /api/decks/:id/messages
     └→ archiveConversation()
     └→ destroyAgent() × N via Arx API
     └→ broadcast(deckId, { type: agent_status, status: offline })
     └→ All tabs → gray dot + empty messages
```

### Agent Creates Storyboard (Plan Phase)

```
Agent inside sandbox: writes storyboard.json via arx_upload tool
  └→ Arx stores artifact
  └→ Chat stream ends → PP syncArtifactsToDeck()
     └→ Updates deck DB
     └→ broadcast(deckId, { type: artifact_synced, phase: plan, artifact_key: storyboard })
        └→ All tabs → StoryboardPanel updates in real-time
```

### User Edits Memory → Agent Sync

```
User edits memory in EditModePage
  └→ POST /api/decks/:id/context { key, content }
     └→ PP saves to DB
     └→ broadcast(deckId, { type: context_updated, action: updated, actor: user:alice })
        └→ Other tabs → refresh sidebar (silent or with notification)
     └→ PUT /api/agents/:id/files { key, content } (Arx API)
        └→ Arx provider writes to sandbox + recompiles CLAUDE.md
        └→ Agent now has updated memory on next interaction
```

## Auth & Security

- **Arx → PP webhook**: Authenticated via shared `ARX_SERVICE_KEY` in POST body
- **PP → Frontend SSE**: Authenticated via JWT token in query param (`?token=xxx`)
- **PP → Arx API**: Authenticated via user JWT (forwarded from frontend)
- **Multi-tab**: All connections for the same deck receive the same events; no cross-deck leakage
- **Actor tracking**: Events carry `actor` field to distinguish self vs. others

## Error Handling

- **SSE disconnect**: Frontend auto-reconnects with exponential backoff
- **Webhook failure**: Arx logs warning, does not retry (fire-and-forget). State self-corrects on next user interaction
- **Stale connection**: 30s heartbeat keeps alive through proxies. Dead connections cleaned up on write failure
- **Agent file update failure**: Arx API returns error; PP logs but does not block the user edit. Agent gets updated files on next container recreation.
