# PitchPilot 项目立项书 / Project Charter

> **做 Pitch 的本质是叙事，而不是排版。**
> *The essence of a pitch is storytelling, not formatting.*

---

## 1. 项目概述 / Project Overview

**项目名称 / Project Name：** PitchPilot

**发起人 / Sponsor：** 张路宇

**项目负责人 / Project Lead：** Peter Han

**立项日期 / Initiation Date：** 2026-03-05

**项目定位 / Positioning：**

PitchPilot 是一个 Pitch 材料自动生成系统。它将 Confluence 中沉淀的行业洞察、客户案例、产品路线图（Product Roadmap）、品牌故事、市场背书、技术趋势等素材，转化为结构化的内容资产（Content Assets）。当需要向客户 Pitch 时，用户只需通过可对话的向导式界面（Wizard-style UI），系统 与用户相互启发式对话，系统便会自动调取最新素材、遵循品牌规范、即时编排生成一份专业的、客制化的 Pitch 材料。

---

## 2. 项目背景与动机 / Background & Motivation

**痛点 / Pain Points：**

- 每次客户 Pitch 都需要从零或半成品开始制作材料，耗费大量时间（通常数小时甚至数天）
- 素材散落在各处，缺乏统一管理和实时更新机制

**机会 / Opportunity：**

- 现有技术条件（大模型能力 + 知识库结构）已初步满足自动化生成的要求
- 已验证品牌规范可被自动遵循（田磊验证 Join Us 页面，馨睿的 Investor Deck 改版）
- 从零生成一份完整 Pitch 文档已被验证可在 32 分钟内完成，PitchPilot 目标是将此压缩到 15 分钟以内
- Confluence 素材库已有一定积累，具备结构化消费的基础

---

## 3. 项目目标 / Objectives

### 核心目标 / Primary Goals

1. **15 分钟生成客制化 Pitch 材料** — 任何同事或合作伙伴，在客户会议前 15 分钟即可完成一份高质量、客制化的 Pitch 内容
2. **素材实时动态化** — 系统自动拉取 Confluence 最新状态的素材，确保每次生成的内容都是最新的
3. **品牌规范零偏差** — 自动遵循公司品牌视觉规范，无需人工调整
4. **人人可操作** — 向导式界面，无技术门槛，覆盖所有区域的同事和伙伴

### 关键成果 / Key Results

| 指标 / Metric | 目标值 / Target |
|---|---|
| 单次生成时间 / Generation Time | ≤ 15 分钟 |
| 品牌规范符合率 / Brand Compliance | 100% |
| 用户操作学习成本 / Onboarding Time | ≤ 1 分钟 |
| 制品存档可追溯 / Archive & Traceability | 100% |

---

## 4. 产品范围 / Scope

### 4.1 内容模块 / Content Modules

用户可按需选择以下章节模块进行排列组合：

| 模块 / Module | 说明 / Description |
|---|---|
| 对话框 | Agent 与人的交互接口 |
| 参考资料（上下文）面板 | 提供给 AI 的参考资料 |
| 制成品预览 | 展示效果 |
| Agent 内核适配 | Agent Sandbox（兼容 claude code, codex, gemini cli） |

### 4.2 功能范围 / Feature Scope

**Phase 1 — MVP：**

- 对话：说明需求
- 支持 skill 预设注入和管理
- 支持上传文件作为参考资料
- 支持预览制成品
- Pitch 材料生成
- 支持发布、导出 HTML
- 单租户

**Phase 2 ：**

- 富文本、交互式需求沟通界面
- 可交互的制成品预览、修改页面
- 替换 Claude Code 为开源 coding agent
- 多租户

### 4.3 不在范围内 / Out of Scope

- 不基于 Dify 构建（可后续融入）
- 不做 PPT / PDF 导出（仅输出 HTML）
- 不做移动端适配

---

## 5. 技术方案概要 / Technical Approach

### 架构思路 / Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    用户界面 / UI Layer                    │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────────┐ │
│  │  AI 对话框    │ │ 文件管理面板  │ │  制成品预览       │ │
│  │  启发式交互   │ │ 参考资料上传  │ │  实时渲染 & 发布  │ │
│  └──────┬───────┘ └──────┬───────┘ └────────▲─────────┘ │
└─────────┼────────────────┼──────────────────┼───────────┘
          │                │                  │
┌─────────▼────────────────▼──────────────────┼───────────┐
│              Agent 内核 / Agent Core                     │
│  ┌─────────────────┐  ┌────────────────────────────┐    │
│  │  Claude Code     │  │  Skills 管理                │    │
│  │  对话理解 · 编排  │  │  品牌规范 · 内容模板 · 叙事  │    │
│  └─────────────────┘  └────────────────────────────┘    │
└──────────────────────────────────────────────────────────┘
```

**MVP 技术选型：**

| 层级 / Layer | 技术选型 / Technology | 说明 / Notes |
|---|---|---|
| 前端 | React + Vite | 对话框 + 文件管理 + 制成品预览，单一 SPA |
| Agent Sandbox | 独立沙盒环境 | 运行 Coding Agent（Claude Code CLI 等），Skill 预设注入其中，产出物也生成在沙盒内 |
| Agent 内核 | Claude Code CLI | 对话理解、内容编排、代码生成 |
| Skill 体系 | dify-deck skill（已有，须迭代） | 品牌规范、Slide 模板引擎（React + TypeScript） |
| 制成品渲染 | dify-deck slide engine | 基于现有引擎，HTML 输出 |
| 文件管理 | 本地文件系统 | MVP 阶段用户手动上传参考资料 |
| 制成品发布 | 静态部署（沙盒内） | 生成后可发布为可访问的网页 |

**Phase 2 演进方向：**

| 层级 / Layer | 演进 / Evolution |
|---|---|
| Agent 内核 | 替换为开源 coding agent 或 Dify Agent，降低依赖 |
| 素材来源 | 接入 Confluence API，自动拉取结构化素材 |
| 交互界面 | 富文本、可交互的需求沟通与制品编辑 |
| 制成品 | 支持在线交互式修改 |
| 多租户 | 支持 x 路并发 |
| 自定义凭证 | 支持用户配置自己的 LLM crediential |

---

## 6. 节奏 / Cadence

| 阶段 / Phase | 时间 / Timeline | 交付物 / Deliverable | 状态 / Status |
|---|---|---|---|
| **M0 — 立项启动** | Day 0 (2026-03-05) | 立项书 | ✅ 完成 |
| **M1 — Agent 跑通** | Day 0 (2026-03-05) | Claude Code + dify-deck skill 端到端生成 Deck | ✅ 完成 |
| **M2.1 — Demo 跑通** | Day 1 (03-06 周五) | 前端骨架 + 对话 + Agent 本地进程运行 + 产出物 | ✅ 完成 |
| **M2.2 — 核心闭环** | Day 2 (03-09 周一) | Skill 注入 + 制成品预览 + HTML 发布 | 进行中（制成品预览 ✅） |
| **M2.3 — 完善体验** | Day 3 (03-10 周二) | 文件上传参考资料 + Skill 管理界面 + 交互打磨 |  |
| **M2.4 — Confluence 对接** | Day 4 (03-11 周三) | Confluence API 接入，素材自动拉取与解析 |  |
| **M3 — 迭代** | Day 5-7 (03-12 ~ 03-14) | 核心用户试用、收集反馈、快速迭代 |  |

---

## 7. 风险与应对 / Risks & Mitigation

| 风险 / Risk | 影响 / Impact | 应对 / Mitigation |
|---|---|---|
| Claude Code CLI 调用稳定性与成本 | 生成体验不可控、运营成本高 | MVP 验证价值后，Phase 2 迁移至开源 Agent 或 Dify Agent |
| 生成内容叙事深度不够 | 制品"看着像 AI 生成的" | Andy 提供 Storytelling 框架，优化叙事 Prompt |
| 品牌规范细节遗漏 | 视觉一致性不达标 | 田磊 Review 每个版本的品牌合规性 |
| Skill 体系灵活性不足 | 难以适配多样化的 Pitch 场景 | 设计可扩展的 Skill 注入机制，支持自定义模板 |

---

## 8. 成功标准 / Definition of Success

PitchPilot 成功的标志是：

1. **任何一位同事或合作伙伴**，面对任何客户，都能在会议前 **15 分钟内**完成一份客制化的、品牌合规的 Pitch 材料
2. 生成的制品质量达到或超过人工精心制作的水平——**效果要炸裂**（内部盲测中 ≥ 80% 的评审认为质量不低于人工制作）
3. 用户上手时间 **≤ 1 分钟**，无需培训，对话即用
4. 制品可一键发布为网页，随时可分享、可回溯
