---
name: setup-pstack
description: Configure which models pstack uses per role. Detects available models and writes the current runtime's override sheet. Use for /setup-pstack, "configure pstack models", changing pstack's model choices, or turning the SessionStart hook on or off.
---

# Setup pstack

On Devin, read the [platform mapping](../poteto-mode/references/devin-tools.md), including its per-skill notes, before following this skill.


On another runtime, read [Other runtimes](#other-runtimes) below for where the sheet lives and how it loads; the steps are the same.

Write the current runtime's per-role model override sheet, using the path in [Other runtimes](#other-runtimes). Each pstack skill names a default model inline; the override sheet adapts those defaults to the models you actually have access to.

On Devin the sheet lives at `~/.devin/pstack-models.md`. Devin has no `@` include: the pstack plugin rule loads the routing mandate at session start, and skills read the sheet directly when they consult role defaults. Write the file at that path and it applies to every Devin session on this machine.

## Steps

### 1. Detect available models

Enumerate the `devin_mode` values `devin_session_create` accepts in this session. That is the dependable source; on this fork they are the SWE-2 modes listed in [Models](#models) below, each running that tier of SWE-2. The default panel is listed there too. Ask the user to confirm or paste any additional slugs they want available. Never write a real slug you have not confirmed is available. The aliases `inherit-parent` and `auto` are always valid even though they are not detected slugs. Both mean the role runs on the parent session's mode, which the create call expresses by omitting `devin_mode`.

### 2. Load current state

The default role-to-model mapping is the rule shape shown in the Write the override sheet step below. If the current runtime's sheet already exists, read it and treat its values as the current choices. Otherwise start from those defaults. A line whose role is not in that shape, such as `how critics`, is from a retired role. Drop it. An older sheet may name models no longer available; replace each with the closest current mode.

### 3. Map and confirm

Show every role with its current model, marking any real slug not in the detected set as needing a choice. Also list each line step 2 dropped or rewrote. Ask whether to accept as-is or change specific roles, offering the detected modes plus `inherit-parent` and `auto` as the options. Prefer `message_user` with fixed choices over free text. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena selects one value from it that differs from the parent's when possible. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

Then ask for the default reasoning effort, the `default effort` line. It is `session`, which keeps the parent session's effort, or one of the levels in [Models](#models). Start from the default listed there. Every role value without a suffix runs at it. Then ask whether any role should run at another level. A role value may carry one after its slug, as in `<slug> @xhigh`; panel entries take their own, as in `<slug> @xhigh, <slug> @max`. On Devin there is no separate effort flag: fold the level into the mode choice per devin-tools' Reasoning effort section. Leave the suffix off for the default effort.

### 4. Choose whether the session hook routes tasks

On Claude Code and Codex, the plugin's `SessionStart` hook injects the poteto-mode mandate on startup, resume, clear, and compact. On Devin the same mandate comes from the plugin rule `poteto-mode-routing`, which loads at session start. Ask whether to keep routing on. The default is on. The answer is the `session hook` line in the current runtime's sheet: `on` or `off`. With no sheet or no line, routing stays on. On Devin, honor `off` by ignoring the routing rule's mandate; the line is inert where a hook is what reads it.

### 5. Validate

Every real slug written must be in the detected set. `inherit-parent` and `auto` always pass. Validate the slug without any `@<level>` suffix, and the level against the effort levels in [Models](#models). The `default effort` value is one of those levels or `session`. If a chosen real slug or level is not available, stop and ask again.

### 6. Write the override sheet

Write the current runtime's sheet with the shape below. Overwrite the whole file so re-runs stay idempotent.

```markdown
# pstack model configuration

Per-role model overrides for pstack skills. Each pstack SKILL.md names its defaults in a Models section; the values here override those defaults. Delete a line to fall back to the skill default. A value of `inherit-parent` or `auto` runs that role on the parent session's model (the `Agent` call omits `model`); an alias entry in a panel list still counts toward that panel's fan-out. A model may carry a reasoning effort, as in `swe-2-medium @xhigh` (levels: low, medium, high, xhigh, max); the role then runs through the pstack effort agent of that level, each entry of a panel list on its own. `default effort` sets the level for a value without one; `session` keeps the parent session's effort. `session hook: off` stops the SessionStart hook from injecting the poteto-mode mandate; any other value, or no line, leaves it on.

feature, refactoring: swe-2-medium
bug-fix: swe-2-max
perf-issue: swe-2-max
hillclimb: swe-2-max
judgment and prose: swe-2-medium
strongest judgment: swe-2-max
how explorer: swe-2-medium
how explainer: swe-2-medium
why investigators: swe-2-medium
why synthesizer: swe-2-medium
reflect tooling: swe-2-medium
reflect judgment, divergent, synthesizer: swe-2-medium
arena runners: swe-2-max, swe-2-high, swe-2-medium
arena cross-judge pool: swe-2-max, swe-2-high, swe-2-medium
swarm workers: swe-2-medium
architect runners: swe-2-max, swe-2-high, swe-2-medium
interrogate reviewers: swe-2-max, swe-2-high, swe-2-medium

default effort: session
session hook: on
```

### 7. Wire it in

On Devin, write the sheet to `~/.devin/pstack-models.md`; nothing else is needed — pstack skills read that path when they apply role defaults. Do not write model rows into `AGENTS.md`: the sheet is the record.

### 8. Confirm

Tell the user where the override was written, how its model rows load, and whether the plugin hook is on. Re-running this skill updates the override sheet.

## Other runtimes

The role lines are the same everywhere. What differs is the sheet path, how the runtime loads it, and how you list models. Detect models with the runtime's own tool and never write a slug you have not seen listed. A runtime whose subagent call has no model parameter still gets the sheet, as the record of the user's choice, and applies it where it can. The `session hook` line applies to the Claude Code and Codex plugins.

| Runtime | Sheet | Load | List models | Status |
| --- | --- | --- | --- | --- |
| Claude Code | `<config>/pstack-models.md` | `@<config>/pstack-models.md` in `<config>/CLAUDE.md` | the `Agent` tool's model parameter | verified live |
| Devin | `~/.devin/pstack-models.md` | read directly by pstack skills; routing comes from the plugin rule | the `devin_mode` values `devin_session_create` accepts, see [devin-tools.md](../poteto-mode/references/devin-tools.md#model-names) | this fork's target |
| opencode | `~/.config/opencode/pstack-models.md` | add the path to the `instructions` array in `opencode.json` | the `models` slash command in the session | from published docs, no live session |
| Gemini CLI | `~/.gemini/pstack-models.md` | `@~/.gemini/pstack-models.md` in `~/.gemini/GEMINI.md` | the `model` slash command in the session | from published docs, no live session |
| Prime Agent | no documented sheet path; Prime's configuration chooses models | | | no live session |

## Models

Stamped from `plugins/pstack/models.json` (edit there, rerun `tools/generate.mjs`).

- Available models: `swe-2-medium`, `swe-2-high`, `swe-2-max`
- Default panel: `swe-2-max`, `swe-2-high`, `swe-2-medium`
- Reasoning effort levels: `low`, `medium`, `high`, `xhigh`, `max`
- Default reasoning effort: `session`
- Single-role default: `swe-2-medium`
