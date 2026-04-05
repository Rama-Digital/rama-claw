# Rama Claw

Public documentation repo for practical operational workflows, implementation checklists, command design notes, and field context.

Repositori dokumentasi publik untuk workflow operasional, checklist implementasi, catatan desain command, dan konteks lapangan.

## Language / Bahasa

- 🇮🇩 Indonesia: `docs/id/`
- 🇬🇧 English: `docs/en/`

## Repository Structure

- `docs/id/` → dokumentasi Bahasa Indonesia
- `docs/en/` → English documentation
- `docs/workflows/` → legacy workflow path (kept for compatibility)

## Recommended Workflow Pattern

```text
docs/<lang>/workflows/<workflow-name>/
  README.md
  CHECKLIST.md
  COMMAND-DESIGN.md
  NOTES.md
```

## Current Workflow Content

- Telegram OpenAI OAuth connect workflow
- implementation safety checklist
- `/openai` command design and behavior
- operational notes and troubleshooting context

## Documentation Principles

- practical and reusable steps first,
- avoid sensitive internal details,
- keep examples safe for public sharing,
- workflow-centric structure (one workflow, one folder, complete context).
