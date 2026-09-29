# REFramework Porreta — Documentation index

REFramework Porreta is UiPath's REFramework with the following changes: a JSON config (local file or Orchestrator asset), a retry for Initialization, exception email notifications, an option to mark the job as Faulted, Orchestrator API helpers, and mock-based tests.

## Documentation rules (short version — full rules in [AGENTS.md](../AGENTS.md))
- **Annotation = contract.** The root annotation of each `.xaml` documents purpose, arguments, how the workflow is built and what it throws. Arguments are documented **only** there.
- **Markdown = how to use it and why.** Design decisions, conventions, how to extend, limitations.
- **Location.** A module's note sits in its folder (`README.md`). Project-level notes are here in `Documentation/`.
- **Config keys.** Each key is described by its inline comment in [Data/Config.json](../Data/Config.json).

## Index

### Project level
| Note | Covers |
|---|---|
| [Main.md](Main.md) | Main.xaml state machine: Initialization retry, notifications, counters, Faulted rule |
| [Config.md](Config.md) | Config JSON structure, local vs asset source, how it is loaded |

### Modules
| Note | Covers |
|---|---|
| [Reusables/Email/README.md](../Reusables/Email/README.md) | Send Email module: HTML templates, replacements, CSS inlining, attachments, engines |
| [Reusables/Email/Notifications/Notifications.md](../Reusables/Email/Notifications/Notifications.md) | Exception notification emails (BRE / SE in Process, exception in Initialization); sub-module of Email, never throws |
| [Reusables/OrchApi/README.md](../Reusables/OrchApi/README.md) | Orchestrator API helpers (in development) |
| [Code_Snippets/README.md](../Code_Snippets/README.md) | HTTP request snippet, per-test config helper |
| [Tests/README.md](../Tests/README.md) | Testing approach: Main_mock, scenarios, testing config asset |

### Project management
| Note | Covers |
|---|---|
| [ProjectManagement/Updates.md](ProjectManagement/Updates.md) | Changelog by cycle |
| [ProjectManagement/Diary.md](ProjectManagement/Diary.md) | Dated working log |
| [ProjectManagement/Backlog.md](ProjectManagement/Backlog.md) | Fixes found in the project review, as a checklist |

### Tools and references
| File | Covers |
|---|---|
| [Tools/DocstringPrompt.md](Tools/DocstringPrompt.md) | Prompt that drafts a workflow docstring from its XAML |
| [REFramework Documentation-EN.pdf](REFramework%20Documentation-EN.pdf) | Documentation of the original UiPath REFramework |
