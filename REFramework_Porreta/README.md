# REFramework Porreta

This project implements:

* **Config as JSON** — [Documentation/Config.md](Documentation/Config.md)
   * `Data/Config.json` (or an Orchestrator text asset) replaces `Config.xlsx`: nested groups, inline comments, Git-friendly diffs. Assets listed in `Assets` are loaded into the config at runtime.
* **Retry for Initialization** — [Documentation/Main.md](Documentation/Main.md#initialization)
   * A system exception in Initialization re-runs the state up to `Max.SystemExceptionsAtInitialization` times instead of ending the job.
* **Business exception in Initialization** — [Documentation/Main.md](Documentation/Main.md#transitions)
   * A `BusinessRuleException` in Initialization (e.g. invalid credentials) goes straight to End Process, with no retry.
* **Faulted job when something failed** — [Documentation/Main.md](Documentation/Main.md#end-process)
   * With `FlowControl.ShouldMarkJobAsFaulted`, the job ends as Faulted in Orchestrator when Initialization failed or any transaction failed (BRE included).
* **Exception email notifications** — [Reusables/Email/Notifications/Notifications.md](Reusables/Email/Notifications/Notifications.md)
   * An HTML email on BRE / SE in Process and on a failed Initialization, each toggled and configured in `Email.Exception`. A failed email is only logged and never stops the job.
* **Send Email module** — [Reusables/Email/README.md](Reusables/Email/README.md)
   * Engine-independent email workflow: HTML templates with `{{placeholders}}`, shared CSS inlined at runtime, subject from the `<h1>`, images, tables and attachments.
* **Kill processes from config** — [Framework/KillAllProcesses.xaml](Framework/KillAllProcesses.xaml)
   * Kills the `;`-separated processes of the `ProcessToKill` asset / key, logging a Warn for each one that can't be killed.
* **Orchestrator API helpers** (in development) — [Reusables/OrchApi/README.md](Reusables/OrchApi/README.md)
   * GetToken, GetJobs, GetThisJob, RestartJob, built on the HTTP request snippet; groundwork for `ShouldReleaseThisMachine.xaml`.
* **Mock-based tests** — [Tests/README.md](Tests/README.md)
   * Main is tested through `Mocks/Main_mock.xaml`, which forces BRE / SE per scenario; per-test config through the `ConfigJson_Testing` asset.

Everything else behaves like the standard REFramework — see [REFramework Documentation-EN.pdf](Documentation/REFramework%20Documentation-EN.pdf).

## Documentation rules (short version — full rules in [AGENTS.md](AGENTS.md))
- **Annotation = contract.** The root annotation of each `.xaml` documents purpose, arguments, how the workflow is built and what it throws. Arguments are documented **only** there.
- **Markdown = how to use it and why.** Design decisions, conventions, how to extend, limitations.
- **Location.** A module's note sits in its folder (`README.md`). Project-level notes are in `Documentation/`.
- **Config keys.** Each key is described by its inline comment in [Data/Config.json](Data/Config.json).

## Index

### Project level
| Note | Covers |
|---|---|
| [Documentation/Main.md](Documentation/Main.md) | Main.xaml state machine: Initialization retry, notifications, counters, Faulted rule |
| [Documentation/Config.md](Documentation/Config.md) | Config JSON structure, local vs asset source, how it is loaded |

### Modules
| Note | Covers |
|---|---|
| [Reusables/Email/README.md](Reusables/Email/README.md) | Send Email module: HTML templates, replacements, CSS inlining, attachments, engines |
| [Reusables/Email/Notifications/Notifications.md](Reusables/Email/Notifications/Notifications.md) | Exception notification emails (BRE / SE in Process, exception in Initialization); sub-module of Email, never throws |
| [Reusables/OrchApi/README.md](Reusables/OrchApi/README.md) | Orchestrator API helpers (in development) |
| [Code_Snippets/README.md](Code_Snippets/README.md) | HTTP request snippet, per-test config helper |
| [Tests/README.md](Tests/README.md) | Testing approach: Main_mock, scenarios, testing config asset |

### Project management
| Note | Covers |
|---|---|
| [Documentation/ProjectManagement/Updates.md](Documentation/ProjectManagement/Updates.md) | Changelog by cycle |
| [Documentation/ProjectManagement/Diary.md](Documentation/ProjectManagement/Diary.md) | Dated working log |
| [Documentation/ProjectManagement/Backlog.md](Documentation/ProjectManagement/Backlog.md) | Fixes found in the project review, as a checklist |

### Tools and references
| File | Covers |
|---|---|
| [Documentation/Tools/DocstringPrompt.md](Documentation/Tools/DocstringPrompt.md) | Prompt that drafts a workflow docstring from its XAML |
| [Documentation/REFramework Documentation-EN.pdf](Documentation/REFramework%20Documentation-EN.pdf) | Documentation of the original UiPath REFramework |
