# Changelog

Newest cycle first. Each entry says **what changed** and links to the note that explains it. How-to content lives in the module notes.

## Cycle 3 (current)

### Config JSON
The config moved from `Config.xlsx` to `Data/Config.json` (local file or Orchestrator asset), loaded by `InitAllSettingsJson.xaml`. Includes `UpdateLocalConfig` and assets loaded into the root of the config.
→ [Config.md](../Config.md)

### Initialization retry
Initialization is retried on system exceptions, up to `Max.SystemExceptionsAtInitialization`. A business exception goes straight to End Process.
→ [Main.md](../Main.md)

### Faulted job
`FlowControl.ShouldMarkJobAsFaulted` ends the job as Faulted when any transaction failed or Initialization failed.
→ [Main.md](../Main.md)

### Exception notifications
Emails for `BRE_inProcess`, `SE_inProcess` and `Exception_inInit`, all handled by one workflow, `SendEmail_Exception.xaml`.
→ [Notifications.md](../../Reusables/Email/Notifications/Notifications.md)

### Notifications moved into the Email module; failures no longer stop the job (2026.09.29)
- `SendEmail_Exception.xaml` and its note moved from `Framework/Porreta/Notifications/` to `Reusables/Email/Notifications/`; its test moved to `Tests/Reusables/Email/Notifications/`. Main and Main_mock invoke the new path.
- The whole workflow is wrapped in a Try Catch that logs failures at Error and doesn't rethrow, so a failed email no longer breaks Main's transitions (Backlog P1).

→ [Notifications.md](../../Reusables/Email/Notifications/Notifications.md), [Main.md](../Main.md)

### Orchestrator API helpers (in development)
GetJobs, GetThisJob, RestartJob, GetToken; `ShouldReleaseThisMachine.xaml`.
→ [Reusables/OrchApi/README.md](../../Reusables/OrchApi/README.md)

### Testing
Main tested through `Mocks/Main_mock.xaml`, driven by `Test.Scenario` and the queue item's `SpecificContent("Exception")`. Per-test config through the `ConfigJson_Testing` asset.
→ [Tests/README.md](../../Tests/README.md)

### Documentation
Documentation rules defined in [AGENTS.md](../../AGENTS.md): the annotation holds the contract, the markdown holds how-to and why. Notes moved next to their modules. Index added in [Documentation/README.md](../README.md).

## Cycle 2

### Send Email module
A module that maps what every email has in common (body from an HTML template with text / image / table replacements, attachments from folders and files, recipients) separately from the engine that sends it. Integration Service is the implemented engine.
→ [Reusables/Email/README.md](../../Reusables/Email/README.md)

### Docstring prompt
A prompt that drafts a workflow docstring from its XAML.
→ [Tools/DocstringPrompt.md](../Tools/DocstringPrompt.md)

### HTTP request snippet
`Code_Snippets/API/API_HTTPS.xaml`, a starting point for HTTP request workflows.
→ [Code_Snippets/README.md](../../Code_Snippets/README.md)

## Cycle 1

### KillAllProcesses / KillProcesses
- `Reusables/KillProcesses.xaml` kills a list of process names; each kill is wrapped in a try/catch that logs failures.
- `Framework/KillAllProcesses.xaml` kills the processes listed in config key `ProcessToKill` (separated by `;`), whose value comes from the `ProcessToKill` asset (JSON `Assets` section).
