# pstack

Lauren Tan's [pstack](https://github.com/cursor/plugins/tree/main/pstack) is an opinionated Cursor skill stack that improves agent outcomes. This fork retargets the port at [Devin](https://devin.ai): every model role routes to SWE-2 (`swe-2-medium`, `swe-2-high`, `swe-2-max`) and tool names resolve through a Devin mapping instead of Codex's. Based on [michael-denyer/pstack-claude](https://github.com/michael-denyer/pstack-claude), which ports pstack for Claude Code and Codex.

Tell `poteto-mode` your goal and it will invoke the correct workflow for the task. It keeps your code concise, simple and verified.

## Install

### Claude Code

Run in Claude Code:

```text
/plugin marketplace add michael-denyer/pstack-claude
/plugin install pstack@pstack-claude
```

### Devin

Install `plugins/pstack` from this repository as a Devin plugin (Devin loads `skills/`, `agents/`, and plugin rules). Once installed, pstack skills are available in every session and the `poteto-mode-routing` rule auto-routes qualifying tasks.

Every pstack role dispatches subagents through `devin_session_create` with a `devin_mode` of `swe-2-medium`, `swe-2-high`, or `swe-2-max`; `references/devin-tools.md` inside `poteto-mode` is the Claude-to-Devin tool and model map. No other model names appear anywhere in the plugin, so nothing can route to a non-SWE-2 model by accident.

Run `setup-pstack` to change role defaults, set a reasoning effort per role (for example `arena runners: swe-2-medium @xhigh, swe-2-high @max` — on Devin the level folds into the mode choice, so `xhigh` means `swe-2-high` and `max` means `swe-2-max`), or turn automatic routing off by disabling the rule or writing `session hook: off` in `~/.devin/pstack-models.md`.

For Claude Code, Codex, Prime Agent, OpenCode, or Gemini CLI installs, see the upstream README and [shared installation](docs/reference.md#shared-skills-installation).

## Getting started

```text
Use poteto-mode to fix the search filter resetting when I change pages.
```

For a bug, it reproduces the failure, uses `how` and `why` to investigate, delegates the fix, then reruns the failing case. If the fix crosses a function boundary, it brings in `architect` before implementation. You receive the fix and the failing and passing evidence.

[Other playbooks](plugins/pstack/skills/poteto-mode/SKILL.md#playbooks) cover planning, features, refactoring, performance issues, investigations, prototypes, PR maintenance, shipping, and longer projects.

![A request enters poteto-mode. Playbook options include Plan, Bugs, Features, and Refactor. Planning can use architect, arena, or swarm; review and verification can use interrogate, tests, and measurements. Supporting skills include how, why, and unslop. The output is Finished work validated.](assets/pstack-overview.png)

## Details

- [Skills and slash commands](docs/reference.md#slash-commands)
- [Runtime setup](docs/reference.md#runtime-support)
- [Models and dependencies](docs/reference.md#configuration-and-dependencies)
- [Maintenance and port scope](docs/reference.md#maintenance)

## Data handling

pstack has no server or telemetry. Anything its skills ask your agent to read, including session transcripts, goes to your model provider. Scripts run locally, and PR tools use your GitHub CLI login.

## Contributing

Thanks for helping make this port better. Bug reports, documentation fixes, and runtime improvements are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for the checks and where your change belongs. Report vulnerabilities privately as described in [SECURITY.md](SECURITY.md).

## License

This port, including its modifications and additions, is also [MIT-licensed](LICENSE), © 2026 Michael Denyer. Original pstack © 2026 Lauren Tan; imported cursor-team-kit skills © 2026 Cursor. See [LICENSE-cursor-team-kit](LICENSE-cursor-team-kit) and [NOTICE.md](NOTICE.md).
