---
name: work-closure-check
description: "Mandatory closure checklist when wrapping up a task: confirm success, clean temporary files, evaluate skill/documentation updates, verify information storage."
category: devops
version: 1.1.0
---

# Work Closure Check

> Enforcement of the closure discipline. When a task transitions from execution to wrap-up, **this skill must be loaded first** to run the systematic checks before notifying the user to confirm success.

## Trigger Keywords

This skill is auto-loaded when the following keywords appear:

- wrap up, finish, summarise, clean up, end
- verified, tested, delivered
- "can you check if it's done", "confirm success"

## Checklist (execute in order)

### ① Credential Leak Scan

**——Did this task involve any credentials (passwords, tokens, API keys, SSH private keys)?**

If yes → check the following locations for plaintext residuals:

| Location | Pass criterion | Violation action |
|:---------|:---------------|:-----------------|
| **Memory** (memory tool) | No passwords/tokens/private keys | Delete immediately, replace with `→secure store` reference |
| **Terminal output** (session history) | No credentials exposed in command arguments | If already exposed, inform the user of the risk |
| **Skill content** | No hardcoded credentials | Move to encrypted storage |
| **File residuals** (temp scripts, config backups) | No credential-containing temp files | Delete or sanitise sensitive fields |

**Core principle:** Credentials must never be stored in memory. All passwords/tokens live only in their designated encrypted storage; memory holds only references.

If none or cleaned → proceed.

### ② Skill Update Check

**——Does this session contain any notable problems, workflows, or techniques worth recording?**

Ask each:

| Question | If yes |
|:---------|:-------|
| Did you solve a specific problem worth recording? | **Save as a new skill** |
| Did you find an existing skill that is outdated, incomplete, or has pitfalls? | **Patch it immediately** |
| Did you identify a class-level pattern worth abstracting? | Create a class-level umbrella skill |
| Are there session-specific details worth archiving? | Add a reference file under the existing skill's `references/` |

**Naming principle:** New skills must use class-level names, not session-specific artefacts (`fix-something-today` ✗ → `something-diagnostics` ✓).

### ③ Decision Record Check

**——Were any trade-off decisions made during this session that are worth documenting?**

If yes → record in the relevant skill reference documentation, or check whether `astra-lifecycle-sync --check` reports matching hooks (requires the lifecycle tool to be deployed):

```markdown
## Decision Record (YYYY-MM-DD)
- **Context:** [problem description]
- **Options:** [A: approach / B: approach / C: approach]
- **Chosen:** [key trade-off]
- **Cost of rejected options:** [what was sacrificed]
```

> Example: "Use IPMI SEL instead of Redfish EventLog — EventLog may be empty, SEL is the hardware black box."

### ④ Service/Device Registration Check

**——Was any new service, MCP, CLI tool, or device connected during this session?**

If yes → register accordingly:

| Object | Registration target |
|:-------|:--------------------|
| New service (MCP / CLI / HTTP) | Service inventory |
| New device (SSH-reachable) | Device inventory |
| New credential category | Credential storage convention |

If none → proceed.

### ⑤ Information Storage Check

This stage is executed dynamically by the agent based on available tools.

**Decision tree: whether to store information and where**

```
Information produced
  │
  ├─ Type: preference / workflow / reference / environment fact / ephemeral state
  │
  ├─ Tool survey: what storage tools does this Hermes instance have?
  │   (agent enumerates available tools — no specific tool is presumed to exist)
  │
  └─ Placement decision:
      ├─ Reusable workflow / solution / lesson
      │   → skill (skill_manage), never into persistent memory
      ├─ Long-lived reference / decision record
      │   → knowledge-base-type tools preferred, local files as fallback
      ├─ Preference / corrected assumption / environment fact
      │   → external persistent memory preferred (avoids context bloat)
      │     fall back to memory when no external persistent storage is available
      └─ Ephemeral state / PR number / commit SHA / one-off output
          → store nowhere, retrieve via session_search
```

**Agent self-check list:**

- [ ] Enumerate all storage tools available to this session
  (agent lists them — no specific tool is presumed to exist)
- [ ] For each available tool, check for **information that should have been recorded** or **information that was recorded incorrectly**
- [ ] Confirm that workflow/solution information belongs in skills, not in any persistent memory
- [ ] Confirm no credential residuals (credentials belong in encrypted storage, not in any tool listed above)
- [ ] Confirm that **no TODO / task progress / commit SHA / ephemeral session state** was written to memory
  (These belong to `session_search` recovery, not persistent storage)
- [ ] Confirm that all persistent memory entries use **declarative sentences** (`user prefers concise responses` ✓),
  never **imperative sentences** (`must respond concisely` ✗)

### ⑥ Environment Baseline Comparison (if modification was involved)

**——If a baseline was recorded before the change, has every item been compared after the change?**

| Item | Difference is a problem? |
|:-----|:-------------------------|
| systemd service list | ✅ New services should be registered; leftovers should be explained |
| Listening ports | ✅ Closed ports should be traceable |
| Docker containers | ✅ Adds/removes should match this task |
| Mount points | ✅ No unexpected mounts or residuals |
| crontab | ✅ New cron entries should be registered |
| Network configuration | ✅ Interface changes should have a reason |

## Standard Closure Procedure

After the checklist is complete, follow this order:

1. ✅ **Notify the user** — "Task complete, please confirm success"
2. ⏳ **Wait for confirmation** — Do **not** skip confirmation and clean up directly
3. 🧹 **After user confirms** — Clean up temporary files/scripts
4. 📝 **Full summary** — What was done, results, outstanding issues
5. 📚 **Final offer** — "Is there anything else I can help with?"

> **Important:** Even if all checklist items pass, the final notification to the user is still required. The user may spot issues you missed.

## Pitfalls

1. **Do not skip "waiting for confirmation" and clean up directly.** The user may want to see test results or temporary file contents.
2. **Do not confuse "skill update" with "store in memory".** Skills capture *how to do something*; memory captures *who the user is / what the environment looks like*. The same technique should not live in both places.
3. **"No credentials involved this time" does not mean "memory is clean".** Memory may still contain credentials from previous sessions — stage ① must be executed regardless.
4. **"Ask one more question" is not optional, it's mandatory.** Even if you think "there's nothing special this time", execute the checklist — you may have missed something worth recording.
5. **Closure-stage skills must not trigger a new closure.** Load this skill → run checks → complete ✓. Do not trigger another round of self-checks after finishing.
