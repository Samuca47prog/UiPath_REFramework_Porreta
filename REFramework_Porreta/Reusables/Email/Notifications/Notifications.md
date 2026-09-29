# Exception notifications

## Purpose
Sends an email when an exception happens, using settings from the config. [SendEmail_Exception.xaml](SendEmail_Exception.xaml) builds the text replacements and calls the [Send Email module](../README.md).
Arguments and error handling: see the workflow annotation.

This is a sub-module of the Email module, but it is **REF-specific**: it reads the `Email.Exception.<origin>` block of the config JSON and a `QueueItem`. When copying `Reusables/Email/` to a project without the REF config, leave this folder out.

## Origins
`in_ExceptionOrigin` selects the config block `Email.Exception.<origin>`.

| Origin | Raised in Main (transition action) | Template | Transaction data |
|---|---|---|---|
| `BRE_inProcess` | Process Transaction → Business Exception | `Data/HtmlTemplates/BRE_Body.html` | yes |
| `SE_inProcess` | Process Transaction → System Exception | `Data/HtmlTemplates/SE_Body.html` | yes |
| `Exception_inInit` | Initialization → Exception (failed initialization); used for both BRE and SE | `Data/HtmlTemplates/ExceptionInit_Body.html` | no (`in_TransactionItem` is Nothing) |

Where these transitions sit in the state machine: [Main.md](../../../Documentation/Main.md).

## Config block
Each origin has the same keys (described in [Data/Config.json](../../../Data/Config.json)):
`Enabled`, `BodyHtmlFilePath`, `Subject`, `Recipients`, `CC`, `BCC`, `AttachmentFolders`, `AttachmentFiles`.
If `Enabled` is not true, the workflow returns without sending anything.

## Placeholders provided
| Placeholder | Value |
|---|---|
| `{{Recipient_Name}}` | Local part (before `@`) of the first recipient |
| `{{Exception_Message}}`, `{{Exception_Source}}`, `{{Exception_StackTrace}}` | From `in_Exception` |
| `{{Transaction_Reference}}`, `{{Start_Datetime}}`, `{{Specific_Content}}` | From `in_TransactionItem`, only when it is not Nothing. `Specific_Content` = `key: value` lines |

## Design decisions

### A notification never stops the job
The whole workflow is wrapped in one Try Catch (`System.Exception`). Any failure — config block missing or `in_ConfigJson` Nothing, template / CSS not found, Integration Service error — is logged at **Error** level (`Send Email Exception Failed`, with message and stack trace) and **not rethrown**.
Reason: Main invokes this workflow in state-machine transition actions. An exception there is not caught by any state, so End Process would never run and applications would stay open (Backlog P1, 2026.09.29).
The catch is inside the workflow, not around each invoke in Main, so every caller — the three Main transitions and any future origin — gets the same protection.

Consequence: a failed notification is visible only in the logs. The job status and the transaction status are not affected.

### Why it lives in the Email module
It only builds replacements and calls `SendEmail.xaml`, so it sits next to the module it uses. It was moved from `Framework/Porreta/Notifications/` to `Reusables/Email/Notifications/` on 2026.09.29.

## Extending — adding an origin
1. Add a block `Email.Exception.<NewOrigin>` to the config, with the same keys.
2. Create the template in `Data/HtmlTemplates/`. Put the subject in its `<h1>`.
3. Invoke `SendEmail_Exception.xaml` with `in_ExceptionOrigin = "<NewOrigin>"` at the point where the event happens. No try/catch is needed around the invoke.

## Limitations / roadmap
- Recipients / CC / BCC / attachment lists are split with VB `Split(x)`, whose default delimiter is a **space**, not `;`. A list like `a@x.com;b@y.com` is not split into two addresses, and empty values become `{""}`.
- The config `Subject` key is not used. The subject comes from the template's `<h1>`.
- Image and table replacements are not passed yet, so `{{Wally_robot_image}}` in the templates stays as literal text.
- Failures are only logged. There is no fallback channel (e.g. an Orchestrator alert) when the email itself can't be sent.
- The workflow has no root docstring yet; only the Try Catch carries a DEV NOTE (see [Backlog](../../../Documentation/ProjectManagement/Backlog.md) §5).
