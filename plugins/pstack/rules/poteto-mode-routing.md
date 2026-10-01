---
trigger: always_on
---

You have pstack.

Invoke the `pstack:poteto-mode` skill and follow its instructions when a task meets any of these:

- it touches more than one file, or changes a signature other files call
- it involves a design or architecture choice
- it is a bug whose cause is not yet known, or a performance issue

It routes to the right pstack skill from there. For smaller tasks, such as a contained change to one file with an obvious test, a question, or a one-line edit, work directly and verify on the real artifact.

When the intent is already specific, enter that skill directly: `pstack:tdd`, `pstack:architect`, `pstack:how`, `pstack:why`, `pstack:arena`, `pstack:interrogate`.

User instructions (AGENTS.md, direct requests) take precedence. On Devin, resolve Claude Code tool and model names through `plugins/pstack/skills/poteto-mode/references/devin-tools.md`; every role runs on the SWE-2 modes named in `plugins/pstack/models.json`.
