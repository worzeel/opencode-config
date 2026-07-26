---
name: flerwin
description: Spin up the flerwin agent team — a tech lead, developers (one per domain), a database specialist, QA, critic, and documenter that work together using opencode-ensemble for parallel execution and peer-to-peer messaging. Use when the user wants to plan, build, or review non-trivial work and wants the full team on it (the default for real work).
---

# /flerwin — launch the agent team

Spin up the **flerwin** team and hand it the user's goal. The members coordinate
through opencode-ensemble's messaging and shared task board.

## Step 1 — right-size the team

**Don't always spawn all six.** Pick the members the task actually needs — every
extra teammate is another full context window of tokens.

| Teammate | Agent type | Spin up when… |
| :-- | :-- | :-- |
| `flerwin-lead` | build | almost always (the coordinator) |
| `flerwin-dev` | build | code needs writing/changing (spawn one per domain: frontend, backend, auth, etc.) |
| `flerwin-database` | build | data layer work — schemas, migrations, queries, seed data |
| `flerwin-qa` | build | code changed and needs testing (spawn one per domain for multi-domain projects) |
| `flerwin-smoke` | build | need to verify running system — starts services, hits endpoints with curl, checks pages load |
| `flerwin-critic` | explore | any plan/code/doc worth reviewing (usually) |
| `flerwin-documenter` | build | structure/behaviour changed enough to document |

> **Models:** Resolution order: explicit `model` on `team_spawn` → `modelsByAgent` in
> `~/.config/opencode/ensemble.json` → agent frontmatter `model` → opencode default.
> All flerwin agents are mapped in `modelsByAgent` — that's the single source of
> truth for team models. Frontmatter `model` only applies to agents with no
> `modelsByAgent` entry (the plugin omits the model and opencode falls back).

Common shapes:
- **Pure planning / architecture** → `flerwin-lead` + `flerwin-critic`
- **Bugfix / feature** → `flerwin-lead` + `flerwin-dev` + `flerwin-qa` + `flerwin-critic`
- **Multi-domain feature** → `flerwin-lead` + `flerwin-dev(s)` per domain + `flerwin-qa(s)` per domain + `flerwin-critic`
- **Data-heavy feature** → `flerwin-lead` + `flerwin-database` + `flerwin-dev(s)` + `flerwin-qa(s)` + `flerwin-critic`
- **Full stack with smoke** → add `flerwin-smoke` to verify running system end-to-end
- **Docs / memory files only** → `flerwin-lead` + `flerwin-documenter` + `flerwin-critic`
- **Big greenfield build** → all roles (lead spawns as many devs and QAs as domains need, plus database if data layer is involved, plus smoke to verify it all works running)

Start lean. If the work grows, the lead can message `main` to spawn additional members.

## Step 2 — spawn and orchestrate

1. Spawn `flerwin-lead` with the user's goal — it creates the team and tasks
2. The lead spawns teammates via `team_spawn` with `claim_task` for each role
   - Reference the agent by name (e.g. `agent: "flerwin-dev"`) — its model
     comes from `modelsByAgent` in `ensemble.json` automatically
   - For multi-domain projects, spawn domain-specific QAs (e.g. `qa-frontend`,
     `qa-backend`) just like you spawn domain-specific devs
3. Teammates communicate via `team_message` — peer-to-peer, not through main
4. Tasks are tracked on the shared board — use `team_tasks_complete` when done
5. When everything passes, the lead reports a summary back

## Step 3 — cleanup

When work is done:
1. **Agents self-shutdown.** Each agent calls `team_shutdown` on themselves after completing their task.
2. **Lead verifies completion.** Check `team_status` and `team_tasks_list` to confirm all work is done.
3. **Lead uses `team_merge`** to bring any remaining branches back.
4. **Lead uses `team_cleanup`** to remove the team.
5. **Lead reports a tight summary** to the user.

**Note:** If agents don't self-shutdown (crash, hang), the lead or main session
must intervene using the watchdog protocol.

## Memory files

Agents can write `AGENTS.md` files to persist state between sessions. This is
useful for:
- Recording architecture decisions
- Documenting API contracts
- Tracking schema versions
- Noting known issues

**Convention:** Always named `AGENTS.md`, placed in relevant directories,
minimal content with links to deeper notes.

**Who writes them:**
- **Documenter**: places initial AGENTS.md files
- **Any agent**: can append to existing memory files
- **Lead**: coordinates memory file placement

## Watchdog (main session as backstop)

The lead handles watchdog duties by default, but if the lead itself dies or the
team goes quiet, **the main session must step in**.

### When to check
- If you haven't heard from the lead in a while after spawning
- When `team_results` returns nothing for a long stretch
- When the user asks "what's going on?" or "are they still working?"
- Proactively if you notice the session has been idle

### How to check
1. `team_status` — see who's alive, who's dead, what's claimed
2. `team_results` — pull messages from any agent, check for blockers
3. `team_view` with a specific member name — jump into their session

### How to recover
1. Message the lead directly: `team_message(to: "lead", text: "...")`
2. If the lead is dead, message surviving agents to assess progress
3. If you need a new lead, spawn `flerwin-lead` with the project context and
   current task board state
4. If an agent is dead but the lead is alive, let the lead handle it — just
   nudge them: "hey, looks like X is dead"
5. In extremis, you can shut down everything with `team_shutdown(force: true)`
   on each member and start fresh

### Full protocol details
See `~/.config/opencode/flerwin/TEAM_BRIEF.md` — Watchdog protocol section.

## Opting out

If the user says "solo", "no team", or "quick one", skip all this and just do the
task yourself.
