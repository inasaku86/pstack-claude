# Devin tool mapping for pstack

pstack skills are written in Claude Code tool language (the `Skill` tool, the `Agent` tool, `AskUserQuestion`, Claude model names). On Devin the skills are the same files; only the tool names resolve differently. Read this when a pstack skill names a Claude tool, a driver or bundled skill, or a Claude model. This file is Devin-specific. Other runtimes must use their own concrete tools, model names, and configuration paths.

## Tool actions

| pstack / Claude action | Devin equivalent |
|------------------------|------------------|
| Read a file | `read` |
| Create / edit / delete a file | `write`, `edit`, `MultiEdit` |
| Run a shell command | `exec` |
| Search file contents / find files | `grep`, `find_file_by_name` |
| Fetch a URL | `web_get_contents` |
| Search the web | `web_search` |
| Invoke a skill (the `Skill` tool, `/command`) | `skill_invoke` for loaded plugin skills; otherwise open the SKILL.md and follow its instructions. |
| Dispatch a subagent (the `Agent`/`Task` tool) | `devin_session_create` — each child session runs on its own machine with its own checkout |
| Dispatch N parallel subagents in one turn | N session specs in one `devin_session_create` call; `run_workflow` for structured fan-out |
| Wait for a subagent result | pass `notify_on_response: true` on create or message; read the settled session with `devin_session_interact(action='get')`. Never poll. |
| Free a finished subagent slot | Child sessions settle on their own; `devin_session_interact(action='sleep')` if one lingers |
| Track tasks (the todolist; `TaskCreate` / `TaskUpdate`, or `TodoWrite` on Claude Code) | `task_create`, `task_update`, `task_list` |
| Ask the human a fixed-choice question (`AskUserQuestion`) | `message_user` with `content_type="user_question"` |
| GitHub/PR operations (`gh`, `gh api`) | The builtin git tools: `git_view_pr`, `git_create_pr`, `git_update_pr`, `git_pr_checks`, `git_ci_job_logs`, `git_comment_on_pr`. Use `gh` only where no builtin exists. |
| Read the session transcript (`~/.claude/projects/*.jsonl`) | `devin_session_events` on the session id |

## Subagent policy

poteto-mode's Subagents section sets Claude-specific defaults (`subagent_type: "pstack:poteto-agent"`, `run_in_background: true`). On Devin:

- There is no `poteto-agent` subagent type. Route an ad-hoc subagent through poteto-mode's style by dispatching a `devin_session_create` whose prompt tells it to read the `poteto-mode` skill in full first.
- Child sessions start working as soon as they are created, so `run_in_background: true` has no separate flag. Issue the dispatch and continue; set `notify_on_response: true` to be woken when each settles.
- There are no `pstack:effort-<level>` or `pstack:poteto-agent-<level>` types. When a role value carries `@<level>`, or the `default effort` line names a level, fold it into the mode choice per [Reasoning effort](#reasoning-effort) below and pass the result as `devin_mode`.
- There is no `comment-sicko` subagent type either. The **no-comments** skill spawns it on Claude Code; on Devin dispatch a child whose prompt tells it to read `poteto-mode/references/agents/comment-sicko.md` in full first.
- Claude Code runs every subagent on this machine, so writers need worktrees. Devin child sessions each get their own VM and their own clone, so writers are isolated by default; they share only the git remote, so each writer must push its own branch. A worker that needs state that exists only on your machine belongs in a `run_workflow` `vm_mode="shared"` agent instead.
- A child cannot see your uncommitted work: pass a branch or PR it can fetch, or describe the change in the prompt. Return typed results through `structured_output_schema` on the create call.
- Keep the rest of the policy unchanged. Pass file pointers not inlined context, review every subagent's diff yourself.

## Reasoning effort

The override sheet's `@<level>` suffix has no Devin per-call equivalent. Map it onto the mode itself: `low` and `medium` mean the default tier, `high` and `xhigh` mean the middle tier, `max` means the strongest tier — see [Model names](#model-names). When a role's model and its effort disagree, run the stronger of the two modes.

## Model names

Skills name single-role defaults and panel slugs from `plugins/pstack/models.json`; each model-consuming skill lists its own in a Models section. On Devin these slugs are `devin_mode` values, passed to `devin_session_create` for every dispatched subagent. This fork keeps every tier on SWE-2:

- Single-model roles: `swe-2-medium`.
- Roles that default to the strongest tier (`bug-fix`, `perf-issue`, `hillclimb`, `strongest judgment`): `swe-2-max`.
- Panel roles (`arena`, `architect`, `interrogate`, `how` critics, `reflect`): one child session per panel entry, so `swe-2-max`, `swe-2-high`, `swe-2-medium`. Diversity here comes from different effort tiers of the same model, which is weaker than true multi-model review; say so in the verdict when a panel matters.

`/setup-pstack` writes the configured model list. On Devin, set it to these `devin_mode` slugs.

## Session routing

Devin does not run plugin SessionStart hooks. This fork ships the same mandate as the plugin rule `poteto-mode-routing`, which Devin loads at the start of every session the plugin is installed in. Setting `session hook: off` in the sheet disables the hook on runtimes that have one; on Devin, disable the rule in the plugin to opt out.

## Driver and bundled skills pstack references

The [driver policy](../SKILL.md#non-negotiables) selects the app driver. For skills and drivers named by these workflows, use these Devin equivalents:

| Skill or driver named in pstack | On Devin |
|---------------------------------|----------|
| `run` (drive a CLI/TUI to see a change work) | Run the app yourself via `exec` and observe the real output. |
| Project UI driver | Drive the UI with the `computer` tool or a Playwright script on the CDP endpoint; for golden-path verification hand the brief to `testing_agent`. Do not claim done without observing the artifact. |
| `plugin-dev:skill-development` (Claude's SKILL.md authoring guidance) | Save the skill with `manage_plugin` (kind=skill). Keep `name` + `description` frontmatter and progressive disclosure. |
| `loop` (recurring/self-paced re-invocation, used by `babysit`) | A `devin_automation_manage` schedule or webhook trigger for real recurrence; inside a session, `wait` then re-check, or `notify_on_response` on the session you are watching. |
| `/loop` dynamic tick in `autopilot-full` | Same as `loop`. A live watch can also ride `git_pr_checks`, which returns as soon as checks settle. |

## Per-skill notes

Affected skill entry points point here. Most skills need only the tables above. These need one more mapping:

| Skill | On Devin |
|-------|----------|
| `interrogate` | The `subagent_type`/`model`/`readonly` dispatch fields map to `devin_session_create`; substitute the panel modes from Model names and keep the reviewers independent. `readonly` maps to an instruction not to write, plus omitting `repos` write intent. |
| `setup-pstack` | The skill's Other runtimes table names the Devin sheet path and how it loads; the slugs are the `devin_mode` values (see Model names above). The role rows are identical. |
| `no-comments` | There is no `comment-sicko` subagent type; see Subagent policy above. |
| `teach` | Running `how` and `why` in parallel maps to `devin_session_create` fan-out; image generation uses `generate_image`. |
| `create-verification-skill` | The generated skill lands under `.claude/skills/verify/` on Claude Code; write it to `.devin/skills/verify/` in the target repo instead. The app-driving harness is platform-neutral. |
| `maintain-verification-skill` | The parallel per-feature source readers map to `devin_session_create` fan-out; the project-local skill lives under `.devin/skills/`. |
| `babysit` | `loop` and `AskUserQuestion` resolve through the tables above; PR watches use `git_pr_checks` rather than a sleep loop. |
| `automate-me` | `plugin-dev:skill-development` resolves through the skills table above. |
| `architect` | The runner panel goes through the **arena** skill, so its `devin_session_create` fan-out and mode substitution apply here too. |
| `arena` | The parallel candidates and the cross-judge map to `devin_session_create`; substitute the panel modes for the runners and the cross-judge pool (see Model names above). Candidates are separate VMs, so each pushes its own branch for the graft step. |
| `how` | The parallel explorers and the explainer map to `devin_session_create` fan-out; substitute the configured modes. A repo-scoped question on a connected repo can also use `devin_ask_wiki_question` when a wiki exists. |
| `reflect` | The three reviewers and the synthesizer map to `devin_session_create`; substitute the configured modes. The transcript finder reads Claude Code's layout under `~/.claude/projects/`, so pass the session digest from `devin_session_events` instead. |
| `swarm` | Each worker is a `devin_session_create` call on the configured mode, and those calls already run concurrently; each writing worker pushes its own branch or output directory (see Subagent policy above). |
| `why` | The parallel investigators and the synthesizer map to `devin_session_create`; substitute the configured modes. List MCP servers from the session's own `mcp_list_servers`, not from `.mcp.json` or `claude mcp list`. |

## Vendored scripts

`skills/poteto-mode/scripts/` ships the `watch-pr` PR watcher, the `orch` store CLI, and `worktree-audit.mjs`. The `watch-pr/ship-pr` command owns pending-merge inspection and cancellation; `resume.mjs` owns the shared checkpoint locator described in [Resume storage](resume-storage.md). These scripts use bun and Node.js and run the same on Devin; invoke them through `exec`. They need `bun`, `gh`, and (for stack work) `gt`. `worktree-audit.mjs` reads Claude Code transcripts under `~/.claude/projects/`; on Devin, use `devin_session_events` for session history instead. It imports the transcript walker from `skills/reflect/scripts/find-transcript.mjs`, so keep the `reflect` skill installed beside `poteto-mode`.

## Instructions file

Where a pstack skill says "your instructions file", on Devin that is the repo's `AGENTS.md`, plus whatever rules your installed plugins carry. The pstack override sheet lives at `~/.devin/pstack-models.md`; read it when a pstack skill tells you to consult role defaults.
