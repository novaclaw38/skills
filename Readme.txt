SKILLS FOR REAL ENGINEERS (fork)
================================

This repository is a fork of mattpocock/skills: a set of small, composable agent skills
for doing real engineering with coding agents such as Claude Code and Codex.

SKILL GROUPS
------------
Engineering: grill-with-docs, triage, improve-codebase-architecture, to-spec, to-tickets,
implement, wayfinder, prototype, diagnosing-bugs, research, tdd, domain-modeling,
codebase-design, code-review, resolving-merge-conflicts, wizard.
Productivity: grill-me, handoff, teach, to-questionnaire, wait-what, grilling,
writing-for-agents.

PROJECT LAYOUT
--------------
  skills/         The skills (skills/engineering, skills/productivity, ...)
  docs/           Documentation
  scripts/        Maintenance scripts
  .claude-plugin/ Claude Code plugin manifest
  CONTEXT.md, AGENTS.md, CLAUDE.md, CHANGELOG.md   Project context and history

INSTALLING
----------
  Claude Code:  claude plugins install mattpocock-skills
  Other agents: npx skills@latest add mattpocock/skills
Then run /setup-matt-pocock-skills once per repo.

See README.md for the full description and the reasoning behind each skill. See LICENSE for
the license terms.
