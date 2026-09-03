# How to use this bundle

1. Copy everything except this file into your empty `finrisk/` repo root:
   `CLAUDE.md`, `plan.md`, `docs/`, `openapi/` (empty for now).
2. `cd finrisk && git init && claude`
3. First message to Claude Code: *"Read CLAUDE.md and plan.md, then execute Phase 0 from plan.md. Stop after Phase 0 for my review — don't start Phase 1."*
4. Review what it produces (architecture docs, ADRs, `openapi/finrisk.yaml` v0). Adjust `docs/specs/*.md` if its design decisions change any of your requirements — these files are meant to be edited as you go, not frozen.
5. Each new session: *"Check plan.md for current phase and continue."* Claude Code will re-read `CLAUDE.md` automatically every session since it's the project's memory file; `plan.md` you should explicitly point it to (or ask it to check first).
6. When you finish a phase, check its boxes in `plan.md` yourself (or ask Claude Code to, then verify) before moving to the next one — that checkbox is what keeps a long session from sprinting ahead.
7. Delete this file once you're set up.
