# AGENTS.md — Documentation conventions

These rules apply to anyone, human or AI agent, who adds or changes workflows in this project (REFramework Porreta).
The project uses two documentation layers. Each has its own job, and they must not repeat each other.

| Layer | Where it lives | What it owns |
|---|---|---|
| **Annotation** (docstring) | Inside the `.xaml`, on the root activity | The **contract**: purpose, arguments, how the workflow is built, what it throws |
| **Markdown** | A `.md` next to the thing it describes | The **how-to and the why**: design decisions, conventions, how to extend it, limitations |

Rule of thumb: *if Studio's argument panel would show it, it goes in the annotation. If it explains a decision or how to use a module, it goes in markdown.*

---

## 1. Annotations — the contract

Every workflow has a docstring in the **annotation of its root activity**, in this format. Use `##` headings in every workflow; don't use `#`.

```
## Purpose
This workflow is responsible for ... (1–2 sentences)

## Input Arguments
- in_Name (InArgument<Type>)
  What it is, format, whether it may be null/empty, default behavior.

## Output Arguments
- out_Name (OutArgument<Type>)
  What it contains and when it is null.
(If none: "None")

## Workflow Structure
2–3 sentences on how it is built and which workflows it invokes.

## Error Handling
- What is thrown (BusinessRuleException / SystemException / rethrow) and when.
- What is swallowed and only logged (and at which level).
(Never leave this empty. If nothing is thrown, say so.)
```

Rules:
- **Arguments are documented only here.** A markdown file must never copy the argument list; link to the workflow instead.
- In/Out/InOut types must match the XAML exactly (e.g. `out_Subject` is `OutArgument`, not `InArgument`).
- When an argument is added, renamed or removed, update the docstring in the same change.
- Activity-level annotations stay short: one line on *why* this activity exists, if it isn't obvious from the DisplayName.
- Temporary notes use the dated form, and are removed or moved into markdown once resolved:
  ```
  DEV NOTE: YYYY.MM.DD
  <what and why>
  ```
- A long explanation inside an annotation (more than ~10 lines on design) belongs in markdown. Leave a one-line pointer instead, for example: `See Reusables/Email/README.md — CSS inlining`.
- `Documentation/Tools/DocstringPrompt.md` generates a first draft of a docstring from XAML. Always review the draft against the actual XAML.

## 2. Markdown — how to use it and why

### Where the file goes
- **Module / folder level** → next to the workflows, as `README.md` in that folder (or `<Workflow>.md` when the folder has a single main workflow).
  If the folder is copied to another project, its documentation goes with it.
  Examples: `Reusables/Email/README.md`, `Reusables/OrchApi/README.md`, `Reusables/Email/Notifications/Notifications.md`, `Code_Snippets/README.md`, `Tests/README.md`.
- **Cross-cutting / project level** → `Documentation/`.
  - `Documentation/README.md` — these rules in short + an **index of every note in the project**.
  - `Documentation/Config.md` — config contract and loading behavior.
  - `Documentation/Main.md` — Main.xaml state machine: transitions, retries, counters, Faulted rule, notifications.
  - `Documentation/ProjectManagement/Updates.md` — changelog.
  - `Documentation/ProjectManagement/Diary.md` — dated working log.
  - `Documentation/Tools/` — prompts and helpers for documenting.
- Each topic is documented in **one place**, under the module that implements it. (Example: CSS inlining is implemented in `Metadata_BuildBody`, so it is documented in `Reusables/Email/README.md`, not in Notifications.)

### Module README structure
```
# <Module name>

## Purpose
What problem the module solves (2–3 sentences).

## Workflows
| Workflow | Role |
|---|---|
| [Name.xaml](Name.xaml) | one line — details in its annotation |

## How to use
Steps / minimal example of invoking it. Conventions the caller must follow
(e.g. template placeholders `{{key}}`, file naming, config keys it reads).

## Design decisions
Why it is built this way; alternatives rejected.

## Extending
How to add a new variant (e.g. a new email engine, a new exception origin).

## Limitations / roadmap
Known gaps, TODOs, what is not implemented yet.
```
Leave out a section that doesn't apply; don't add filler.

### What markdown must NOT contain
- Argument lists (these live in the annotation).
- Descriptions of each config key (they live as inline comments in `Data/Config.json`; `Config.md` documents only the structure and loading behavior).
- A description of behavior that the code doesn't have yet. Mark planned work explicitly under *Limitations / roadmap*.
- File paths or names that don't exist. Links are relative (`../Reusables/Email/README.md`), not GitHub URLs.

## 3. Config.json comments — reference for each key
- Every key has an inline `/* comment */` describing its meaning, allowed values and effect.
- When a key's behavior changes, update its comment in the same change.
- Unused keys are removed or marked `/* UNUSED: reason */`.

## 4. Changelog and diary
- `Updates.md`: one section per cycle; each entry = what changed + link to the module note. No how-to content here.
- `Diary.md`: dated entries (`## YYYY.MM.DD`), newest first. Short working notes and next steps.

## 5. Checklist when changing a workflow
- [ ] The root annotation docstring matches the arguments and the error behavior.
- [ ] The module markdown is updated if usage, a convention or a design decision changed.
- [ ] `Config.json` comments are updated if a config key was added or changed.
- [ ] `Documentation/README.md` index is updated if a note was added, moved or renamed.
- [ ] `Updates.md` has an entry; DEV NOTEs are resolved or dated.
- [ ] Names in the docs match the files (e.g. `KillAllProcesses.xaml`, not `KillAllProcess`).

## 6. Notes for AI agents
- Read this file, `Documentation/README.md` and the module's markdown before changing a module.
- Treat the XAML as the source of truth. If a note and the code disagree, report the mismatch; don't silently "fix" either one.
- Editing XAML: only the `sap2010:Annotation.AnnotationText` attribute may be changed without an explicit request. Keep XML escaping intact (`&#xA;` for newlines, `&quot;`, `&lt;`, `&gt;`, `&amp;`) and don't reformat or reorder the rest of the file. Any other XAML change requires the user's approval.
- Write in English, and keep the dense, technical tone of the existing notes.
