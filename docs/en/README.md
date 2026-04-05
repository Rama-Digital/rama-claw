# Documentation Map

This repository stores public documentation focused on workflows, implementation checklists, design decisions, and operational notes.

## Main Structure

### `docs/en/workflows/`
Documentation is organized **by workflow**, not by document type.

Why:
- each workflow keeps all context in one place,
- readers do not need to jump across folders,
- the structure scales better as more workflows are added.

## Recommended Workflow Folder Pattern

```text
docs/en/workflows/<workflow-name>/
  README.md
  CHECKLIST.md
  COMMAND-DESIGN.md
  NOTES.md
```

### File Roles
- `README.md` → main workflow overview
- `CHECKLIST.md` → implementation and validation checklist
- `COMMAND-DESIGN.md` → command and interaction design
- `NOTES.md` → rationale, context, and extra notes

## Existing Workflow
### `docs/en/workflows/telegram-openai-connect/`
- `README.md`
- `CHECKLIST.md`
- `COMMAND-DESIGN.md`
- `NOTES.md`

## How to Use
1. open the relevant workflow folder,
2. read `README.md` to understand the flow,
3. use `CHECKLIST.md` during implementation,
4. use `COMMAND-DESIGN.md` for behavior decisions,
5. read `NOTES.md` for additional context.

## Compatibility Note
Legacy path `docs/workflows/` is still kept to avoid breaking existing links.
