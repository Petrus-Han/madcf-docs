# MADCF Documentation & Research

Documentation, research reports, architecture diagrams, and presentation materials for the MADCF (Multi-Agent Distributed Collaboration Framework) project.

## Structure

```
research/            — Research reports and analysis
  madcf-distributed-positioning.md   — MADCF distributed vs decentralized positioning
  competitive-analysis.md            — Multi-agent framework competitive landscape
  PitchPilot-Project-Charter.md      — PitchPilot project charter

architecture/        — Technical architecture documents
  deck-event-bus.md                  — Real-time event bus architecture

presentations/       — HTML presentation materials
  madcf-intro.html                   — MADCF introduction (5-section narrative)
  madcf-architecture.html            — 3-layer architecture diagram
  PitchPilot-Project-Charter.html    — Project charter (styled)
  pitchpilot-ui-mockup.html          — UI mockup/wireframe
```

## Related Repositories

| Repository | Description |
|-----------|-------------|
| [MADCF](https://github.com/Petrus-Han/MADCF) | Protocol specification + Python reference implementation |
| [Arx](https://github.com/Alai-sudo/arx) | Master node implementation (Rust) |
| [Oasis](https://github.com/Alai-sudo/oasis) | Agent sandbox runtime (Docker) |
| [deck-viber](https://github.com/Petrus-Han/deck-viber) | PitchPilot application (Node.js + React) |

## Key Concepts

- **MADCF** = Multi-Agent **Distributed** Collaboration Framework
- **Arx** = Infrastructure node (Latin: "citadel")
- **Tulpa** = Agent definition (Engine + Credential + Talents)
- **Oasis** = Sandboxed execution environment (Docker container)
