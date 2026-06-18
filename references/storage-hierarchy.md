# Three-Layer Storage Hierarchy — What Goes Where

> Decided 2026-06-18 after a system-wide audit of memory, fact store, and KB.
> Goal: each class of information lands in exactly one tier.
>
> See also: `work-closure-check` §5 信息存储检查

## Decision Tree

```
本次产生的信息
  │
  ├─ 流程 / 排查路径 / 修复步骤 / 配置方法
  │   └─ SKILL（skill_manage action=create/patch）
  │      Class-level 名称，session 细节走 references/
  │
  ├─ 可自动化同步的配置信息（MCP/服务/端口/约定）
  │   └─ 知识库 KB（kb_add）
  │      ├─ hermes_config  ← 服务/MCP/工具配置（由 deploy hook 监督）
  │      ├─ dynamic_ref    ← 会变的参考数据（API 端点、限制表）
  │      └─ sre_incidents  ← 事故记录（需要推理根因，无法自动化）
  │
  ├─ 用户偏好 / 沟通习惯 / 工具链选择
  │   └─ Fact Store (category=user_pref)
  │      用 fact_store(action='add', category='user_pref', ...)
  │
  ├─ 稳定环境事实（路径、IP、项目结构、版本号）
  │   └─ MEMORY.md (陈述句，不用指令句)
  │
  └─ 临时状态 / PR 号 / commit SHA / 会话产物
      └─ 不存任何地方 — 仅用 session_search 回溯
```

## Tier Boundaries

| 例子 | 正确位置 | 常见误放 | 根因 |
|:-----|:---------|:---------|:-----|
| "智谱用 zai provider" | **MEMORY.md** | Fact Store | 稳定环境事实，非偏好 |
| "zypper 优先于 Homebrew" | **Fact Store (user_pref)** | MEMORY.md | 是偏好，非环境事实 |
| "SearXNG 在 127.0.0.2:8931" | **hermes_config KB** | Fact Store / MEMORY.md | 配置信息，应可被自动化同步覆盖 |
| "Gateway 消息长度上限" | **dynamic_ref KB** | MEMORY.md | 会变的参考数据 |
| "部署新 MCP → 写 hermes_config" | **Skill (deploy-register)** | — | 流程性知识 |
| "E2EE OTK 修复步骤" | **sre_incidents KB** | Fact Store | 事故记录，有根因分析 |
| "@hermes00 token invalidated" | **sre_incidents KB** | Fact Store（被删） | 事故记录，非持久事实 |
| "用户喜欢 British English" | **Fact Store (user_pref)** | — | 明确的沟通偏好 |

## Audit Checklist (run quarterly or after 5+ new facts)

1. **Fact Store**: scan for duplicates (same info recorded multiple times in different sessions)
2. **Fact Store**: scan for TODO/temp items that should have been deleted
3. **Fact Store**: verify category field — user_pref vs general
4. **MEMORY.md**: verify no plaintext passwords remain
5. **MEMORY.md**: verify all entries are declarative statements, not instructions
6. **KB ↔ MEMORY** boundary: any MEMORY entry that describes a configurable service → belongs in hermes_config KB
