---
trigger: model_decision
description: Route EGM requests to shared skills. Require fresh-context isolation for Goldfish reviews.
---

# EGM Routing and Isolation Contract (Experimental)

1. Resolve all EGM workflows from the shared `skills/<skill-name>/SKILL.md` files at the repository root. Never rely on copied skill instructions.
2. Resolve reviewer personas from `agents/*.md` and document rubrics from `resources/document_standards/*.md` relative to the same root.
3. Preserve the human approval pauses and file-output boundaries from each `SKILL.md`.
4. Run Goldfish workflows only with isolated, read-only subagents that receive no parent history. If Windsurf Cascade cannot create these subagents, stop. Report that subagent isolation is unavailable. Do not run multiple reviewers in one context.
