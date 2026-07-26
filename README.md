# opencode-config

Personal [opencode](https://opencode.ai) configuration — agents, skills, and preferences.

## Usage

Clone or copy the contents into `~/.config/opencode/`:

```sh
cp -r . ~/.config/opencode/
```

This includes:

- `opencode.jsonc` — main config (model, plugins, permissions)
- `ensemble.json` — model routing for the flerwin agent team
- `AGENTS.md` — coding preferences and communication style
- `agents/` — subagent definitions (flerwin-lead, flerwin-dev, etc.)
- `skills/` — custom skill definitions
- `flerwin/` — team working agreement and task data
