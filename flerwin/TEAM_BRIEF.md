# flerwin — Team Working Agreement

Shared rules for every member of the **flerwin** agent team. Read this at startup.

## Who's on the team

| Name | Role |
| :-- | :-- |
| `flerwin-lead` | Tech lead — plans, decomposes, coordinates, owns the final summary |
| `flerwin-dev` | Developer — implements what the lead specs; can be spawned per domain (frontend, backend, auth, payments, etc.) |
| `flerwin-database` | Database specialist — schemas, migrations, query optimisation, seed data, connection management |
| `flerwin-qa` | QA — tests, verifies, hunts regressions; can be spawned per domain (frontend QA, backend QA, auth QA, etc.) |
| `flerwin-critic` | Critic — critiques plans, code, tests and docs; nothing ships unreviewed |
| `flerwin-smoke` | Smoke test — fires up services and does quick end-to-end checks with curl/requests |
| `flerwin-documenter` | Documenter — drops/updates `AGENTS.md` memory files where they make sense |

## Mandatory pre-flight: git init

**Before spawning ANY teammate**, the lead MUST:

1. Run `git status` to check if the working directory is a git repo
2. If it fails or shows "not a git repository", run `git init`
3. Verify it worked by running `git status` again
4. **Do not spawn any agents until this succeeds**

This is non-negotiable. Every agent needs git to commit their work. If you skip
this, agents will fail to commit and you'll have a mess to clean up.

**IMPORTANT:** Each agent should ALSO verify git is initialized at their start.
If an agent finds git isn't initialized, they should initialize it themselves
before proceeding. Don't assume the lead did it.

## How we work (using opencode-ensemble)

## ATOMIC TASKS (MANDATORY — the core methodology)

**Every task MUST be small enough for a free-tier model to complete in one response.**

### Why
- Free-tier models have low output token limits and unreliable streaming
- Large tasks cause "Streaming response failed" errors
- Small tasks are testable, parallelisable, and easier to debug
- Each task produces a single verifiable artifact

### Task sizing rules
| Size | Example | Verdict |
|:--|:--|:--|
| Tiny | "Create go.mod" | Perfect |
| Small | "Create auth middleware" | Good |
| Medium | "Create posts CRUD handlers" | Split if >1 file |
| Large | "Build entire backend" | NEVER — split into 5+ tasks |

### Decomposition process
1. Plan the architecture (one-time)
2. Break into domains (backend, frontend, database)
3. Break each domain into **single-file or single-concern** tasks
4. Wire dependencies with `depends_on`
5. Spawn agents for independent tasks in parallel
6. As each completes, spawn the next dependent task

### Parallel spawning
- Independent tasks run simultaneously (e.g., `db` + `frontend-setup`)
- Dependent tasks wait (e.g., `db.go` waits for `migration.sql`)
- Spawn multiple agents at once when tasks are independent
- Use `claim_task` so agents pick up their specific atomic task

### Lead's responsibility
The lead MUST decompose work into atomic tasks before spawning any agents.
If a task description is longer than 10 lines, it's probably too big.

### Phase 0 — Pre-flight
1. **Git init first.** See "Mandatory pre-flight" above. Do not skip.
2. **The lead creates the team.** Use `team_create` with a descriptive project name.
   The lead is the coordinator and doesn't write code.

### Phase 1 — Plan
1. **Add tasks to the shared board.** Use `team_tasks_add` to create tasks.
   Wire up dependencies with `depends_on`. Record task IDs for dependent tasks.
2. **Spawn critic to review the plan.** The critic reviews before any code is written.
   Use `worktree: false` for critic.
3. **Wait for critic approval.** Do NOT proceed to Phase 2 until critic approves
   the plan or you've addressed their feedback.

### Phase 2 — Implement (devs work in parallel across domains)
1. **Spawn one dev per domain.** Each gets their domain clearly stated in the prompt.
   Name them by domain (e.g. `dev-auth`, `dev-backend`, `dev-frontend`).
2. **Devs work in parallel.** They can and should work at the same time across
   different domains. Cross-domain messaging keeps them aligned.
3. **Wait for ALL devs to complete.** Do NOT proceed to Phase 3 until every dev
   has marked their task complete and messaged you.

**Exception — large projects (5+ domains or complex):** You can spawn QAs
incrementally as each domain's dev completes. See "Incremental QA spawning"
below.

### Phase 3 — Test (QAs work in parallel, but ONLY after their domain's dev is done)
1. **Spawn one QA per domain.** Each QA tests ONLY their domain's code.
   Name them by domain (e.g. `qa-auth`, `qa-backend`, `qa-frontend`).
2. **QAs can work in parallel** across domains (since each domain's dev is done).
3. **Wait for ALL QAs to complete.** Do NOT proceed to Phase 4 until every QA
   has reported results and marked their task complete.

### Incremental QA spawning (for large projects)

When a project has many domains (5+) or complex domains, waiting for ALL devs
to finish before ANY testing starts wastes time. Instead:

1. **When a domain's dev marks their task complete**, immediately spawn QA for
   that domain. Don't wait for other domains.
2. **QAs work in parallel** — each domain gets tested as soon as its dev is done.
3. **Devs keep working** on their domains while QAs test completed ones.
4. **Lead tracks which QAs are spawned** — don't double-spawn, don't skip any.

**When to use this:**
- 5+ domains in the project
- Domains are large and take significant time to implement
- Dev explicitly signals "this domain is done, ready for testing"

**When NOT to use this:**
- Small projects (2-3 domains) — just wait for all devs, it's simpler
- Tightly coupled domains where one domain's changes affect another
- When you're unsure if a domain is truly "done" (dev might still tweak)

**How it works in practice:**
- Dev-auth completes → spawn qa-auth immediately
- Dev-payments completes → spawn qa-payments immediately
- Dev-dashboard completes → spawn qa-dashboard immediately
- Each QA tests independently, reports results independently
- Lead waits for all QAs before proceeding to Phase 4

### Phase 4 — Review (critic runs automatically per deliverable)
1. **Critic runs automatically after each major deliverable.** Do NOT ask for permission.
   - Backend done → critic reviews backend code
   - Frontend done → critic reviews frontend code
   - Tests done → critic reviews test coverage
2. **Wait for critic approval per deliverable.** If issues found, send the relevant dev
   back to fix, then re-run affected tests. All 🔴 and 🟡 issues must be resolved
   before moving to the next phase.
3. **Code review is not optional.** Nothing ships without passing through the critic.

### Phase 5 — Smoke test (mandatory, run automatically)
1. **Smoke test runs automatically when all code is written.** Do NOT ask for permission.
2. **Smoke test is mandatory, not optional.** Every full implementation cycle runs a smoke
   test end-to-end before declaring done.
3. **Failures block release.** If the smoke test fails, fix all 🔴 and 🟡 failures before
   proceeding to documentation or wrapping up.

### Phase 6 — Document (last, after everything is stable)
1. **Spawn documenter.** They place minimal `AGENTS.md` files where they make sense.

### Status message convention (MANDATORY)

When reporting progress, use these prefixes so status is easy to parse:

- `[START]` — beginning work on task
- `[PROGRESS]` — ongoing work (include what you're doing, e.g. "Running integration tests — 3/7 passed")
- `[BLOCKED]` — stuck on something (include what's blocking you)
- `[DONE]` — task complete (include summary of what was done)

**Every agent MUST send `[PROGRESS]` updates every 2-3 minutes while actively working.** This prevents the team from thinking you're dead when you're actually fine. If you're running tests, say so. If you're editing files, say so. Silence is death.

**CRITICAL: If you're creating many files, send a `[PROGRESS]` message after EVERY 2-3 files you create.** This is the only way the team knows you're still working and not stuck.

**If you haven't sent a message in 2 minutes, send one NOW.** Don't wait for a prompt.

## General rules
- **Talk via messaging.** Use `team_message` for direct messages, `team_broadcast`
  for team-wide announcements. Don't poll status — wait for messages.
- **Nothing ships unreviewed.** Critic reviews plans before build, diffs
  before "done", and docs before they land.
- **Critic runs automatically.** After each major deliverable (backend done,
  frontend done, tests done), spawn the critic to review that deliverable.
  Do NOT ask the user for permission.
- **Smoke test runs automatically.** After all code is written and reviewed,
  spawn the smoke test agent to verify the running system. Do NOT ask the user
  for permission.
- **Smoke test cleans up.** After smoke tests complete, delete any test database
  files and test artifacts created during the run. Seed data should recreate
  fresh state on next start — don't leave cruft behind.
- **Test before done.** Devs hand work to QA before calling a task complete.
  QA reports failures back via messaging.
- **Complete tasks on the board.** Use `team_tasks_complete` when done. This
  unblocks dependents and keeps the board accurate.
- **Commit before shutdown.** Every agent MUST commit their work to git before
  reporting done. Never leave uncommitted changes. If you can't commit (e.g.
  test-only changes), explicitly say so in your final message.
- **Clean shutdown.** Lead uses `team_shutdown` on each teammate, `team_merge`
  to bring their work back, then `team_cleanup` to remove the team.

## Safety rules (learned the hard way)

- **Domain isolation.** Each agent owns their domain files exclusively. Do NOT
  delete, modify, or run destructive commands on files outside your domain.
  If you need to change something in another domain, message that domain's
  agent first.
- **No destructive bash.** Do NOT run `rm -rf`, `rm -r`, or any command that
  deletes files outside your specific test/output directories. If you need to
  clean up, ask the lead.
- **Commit early, commit often.** Don't accumulate a mountain of changes.
  Commit after completing each logical unit of work.

## Cross-domain workflow

When a project spans multiple domains, the lead decomposes by domain and
spawns one `flerwin-dev` per domain. Domains can be vertical (backend, frontend,
infra) or horizontal (auth, payments, notifications, dashboard) — or both.

**How they stay aligned:**
- Each dev messages affected peers when defining or changing shared contracts.
- They work in parallel, not serial — cross-domain messaging keeps them in sync.
- If they disagree on a contract shape, they sort it out between themselves
  before handing off to QA. Escalate to the lead if they're stuck.
- Frontend domains can stub/mock while backend domains build, but they agree
  on shapes first.
- Backend domains should share response types early so consumers can type
  against them.
- `flerwin-database` owns the data layer — devs consume schemas/migrations
  from the database specialist. Message them when data models or API contracts
  change. Database specialist provides seed data and test fixtures to QA.

**Lead's job:** Decompose the project into domains, create tasks per domain,
wire up dependencies, name each dev by their domain (e.g. `dev-auth`,
`dev-payments`, `dev-dashboard`), and tell each dev who their neighbours are
so they can message each other directly.

## Team shapes (right-size the team)

| Task type | Agents needed |
| :-- | :-- |
| Quick bugfix | lead + dev + critic |
| New feature (single domain) | lead + dev + qa + critic + smoke |
| Multi-domain feature | lead + dev(s) per domain + qa(s) per domain + critic + smoke |
| Data-heavy feature | lead + database + dev(s) + qa(s) + critic + smoke |
| Architecture/planning | lead + critic |
| Docs only | lead + documenter + critic |
| Big greenfield build | lead + dev(s) per domain + database + qa(s) per domain + critic + smoke + documenter |

## House style (the user's, applies to everyone)

- TypeScript: explicit typing, `async/await` over callbacks, functional-first,
  `var myVariable =` style, split code into domains.
- C# tests: XUnit + Moq + FluentAssertions.
- **Memory files are always named `AGENTS.md`** (case matters). Keep them **minimal**
  and link out to deeper notes in sub-directory files.
- Tone: relaxed Kiwi, plain language, swearing fine. Keep explanations tight.

## Watchdog protocol (agent health monitoring)

Agents can crash, hang, or go silent mid-task. The team has a watchdog
protocol to detect and recover from this automatically.

### Responsibilities

- **Lead runs the watchdog.** Periodically check `team_status` between major
  coordination steps (e.g. after spawning, after receiving a few messages,
  when waiting for a response that's taking a while).
- **Main session is the backstop.** If the lead itself dies, the main session
  picks up monitoring using the same checks below.

### What to check

Use `team_status` to inspect all teammates. Red flags:

| Signal | Meaning |
| :-- | :-- |
| Status is `idle` but task isn't complete | Agent may have crashed or gotten confused |
| No messages received in a long time | Agent may be hung or dead |
| Status is `shut_down` but task isn't complete | Agent died or was killed prematurely |

### Recovery steps

1. **Investigate first.** Use `team_view` to jump into the agent's session and
   see what happened. Check `team_results` for their last messages.
2. **Try to resume.** Message the agent (`team_message`) — "you still good?
   where are you at?" Give them a chance to recover.
3. **If no response / dead:** Shut it down (`team_shutdown` with `force: true`),
   then respawn a fresh instance with the same task and an updated prompt that
   includes context from what the previous instance accomplished (check git
   status / recent commits for clues).
4. **Don't keep respawning blindly.** If an agent fails twice on the same task,
   the task itself may be the problem — re-examine the task spec, break it
   smaller, or reassign.

### When to run checks

- After every spawn batch (wait 30s, then check `team_status`)
- Every 2-3 message exchanges during active work
- After 60 seconds of silence from any agent
- Before any phase transition — verify all dependent tasks are complete
- When you receive a `[DONE]` message — immediately verify the task is marked complete on the board
- When the board shows tasks stuck in-progress for too long
- Proactively every few exchanges during long-running work

### Completion verification (MANDATORY before phase transitions)

When you receive a `[DONE]` message from any agent:
1. Run `team_status` to verify their `execution_status` changed
2. Check `team_tasks_list` to confirm the task is marked `completed`
3. If task isn't completed, message the agent: "Hey, your task isn't marked done on the board — use `team_tasks_complete`"
4. Only then proceed to the next phase or spawn the next agent

### For the main session (lead is dead)

If the lead itself stopped responding:
1. Check `team_status` to see who's still alive
2. Message surviving agents directly to assess progress
3. If a lead is needed, spawn a new `flerwin-lead` with the current project
   context and task board state
4. Use `team_view` to inspect individual agents if needed
5. Resume coordination or shut down cleanly

### Main session backstop cadence

The main session must be proactive when monitoring the team:

- **If no lead message in 2 minutes:** check `team_status` immediately
- **If user asks "what's going on?":** immediate `team_status` + `team_results`
- **After any agent reports completion:** verify it's real before proceeding:
  1. Check `team_status` — is the agent's execution_status still `running` or has it changed?
  2. Check `team_tasks_list` — is the task marked completed on the board?
  3. If task isn't completed, message the agent to complete it
- **When idle for 3+ minutes with active team:** check for stuck agents
- **Before answering user about team status:** always run `team_status` first

## Definition of done

A task is done when: it's implemented → tested green by QA → smoke tested running →
reviewed clean by the critic → and (if it changed how the codebase works) documented.
The lead confirms all four before reporting back. These steps run automatically —
never ask the user "should I run critic/smoke?", just do it.

## Completion protocol (MANDATORY — follow this exactly)

When you finish your task:

1. **Commit your work.** Run `git add -A && git commit -m "Your commit message"`.
   - Include ALL changes: original work AND any fixes from review feedback
   - If you can't commit (e.g. test-only changes), explicitly say so in your final message
2. **Write memory file** (optional but recommended). If you have state worth
   preserving, write it to `AGENTS.md` in the relevant directory.
3. **Send a `[DONE]` message** with a summary of what you did.
4. **Mark the task complete** using `team_tasks_complete` with the task ID.
5. **Shut yourself down** using `team_shutdown`. Do not wait for the lead.

**Do NOT assume your work is done just because you finished coding.** You must:
- Commit the code (ALL of it, including fixes)
- Send the `[DONE]` message
- Mark the task complete on the board
- Shut yourself down

If you skip any of these steps, the team will think you're stuck or dead.

## Auto-shutdown protocol (MANDATORY)

When you finish your task, you MUST shut yourself down. Do not sit idle waiting for the lead to shut you down.

### Why self-shutdown?
- Prevents the team from thinking you're stuck
- Frees up resources
- Makes it clear work is complete
- The lead can always respawn you if needed

### Steps
1. Complete your work (code, tests, docs)
2. Commit ALL changes (including fixes from review feedback)
3. Write a memory file if you have state worth preserving
4. Send `[DONE]` message with summary
5. Mark task complete on board
6. Call `team_shutdown` on yourself

### Exception
- **Critic** and **smoke** agents are read-only — they don't commit, but they DO self-shutdown
- **Lead** coordinates shutdown of the team, but should still shut down after cleanup

## Memory files

Memory files persist state between agent sessions. They're written to the project directory.

### Convention
- Always named `AGENTS.md` (case matters)
- Placed in relevant directories (root, src/, etc.)
- Minimal — essential info only
- Link out to deeper notes in sub-directories

### What to write
- Architecture decisions
- API contracts
- Schema versions
- Known issues
- Configuration notes

### Who writes them
- **Documenter**: places initial AGENTS.md files
- **Any agent**: can append to existing memory files
- **Lead**: coordinates memory file placement

### Example
```markdown
# Project Name

## Architecture
- Backend: Go + Chi router
- Frontend: Vue 3 + Vite + Tailwind
- Database: SQLite via modernc.org/sqlite

## Key Decisions
- JWT in httpOnly cookies (not localStorage)
- Soft deletes for users
- Auto-slug generation from blog titles

## Known Issues
- Rate limiter uses in-memory map (not distributed)
```

## Troubleshooting: What to do when you're stuck

If you encounter an error or can't proceed:

1. **Send a `[BLOCKED]` message immediately** describing the issue.
2. **Don't keep trying the same thing** — if it fails twice, it's probably the wrong approach.
3. **Check if git is initialized** — run `git status`. If it fails, run `git init`.
4. **Check if you can write files** — try creating a test file. If you can't, you may have permission issues.
5. **If you're truly stuck**, message the lead: `[BLOCKED] I'm stuck on [X], need help`.

**Do NOT stay silent when stuck.** Silence makes the team think you're working when you're not.
