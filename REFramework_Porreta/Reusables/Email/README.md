# Send Email module

## Purpose
This module handles the parts every email has in common — body, subject, recipients and attachments — separately from **how** the email is sent.
Switching the sending engine (Integration Service, SMTP, Office 365…) only changes the engine workflow. Callers are not affected.

## Workflows
| Workflow | Role (arguments and errors are in each annotation) |
|---|---|
| [SendEmail.xaml](SendEmail.xaml) | Entry point: builds the body, collects the attachments, calls the engine |
| [Metadata/Metadata_BuildBody.xaml](Metadata/Metadata_BuildBody.xaml) | Reads the HTML template, inlines the CSS, applies text / image / table replacements, extracts the subject |
| [Metadata/Metadata_Attachments.xaml](Metadata/Metadata_Attachments.xaml) | Builds the attachment list from folders (non-recursive) and individual files |
| [SendEmail_EngineOptions/SendEmail_IntegrationsService.xaml](SendEmail_EngineOptions/SendEmail_IntegrationsService.xaml) | Engine in use: Integration Service Send Email (HTML body) |
| [SendEmail_EngineOptions/SendEmail_SMTP.xaml](SendEmail_EngineOptions/SendEmail_SMTP.xaml) | SMTP engine — stub, not connected |

### Sub-module
| Folder | Role |
|---|---|
| [Notifications/](Notifications/Notifications.md) | Exception notification emails used by Main (`SendEmail_Exception.xaml`). **REF-specific**: reads the config JSON `Email.Exception` block and a `QueueItem`. Never throws — failures are logged at Error |

The rest of the module has no REF dependency. To reuse it in another project, copy `Reusables/Email/` without `Notifications/`, plus the templates you need from `Data/HtmlTemplates/`.

## How to use
1. Create an HTML template (see the conventions below). Existing templates are in `Data/HtmlTemplates/`.
2. Invoke `SendEmail.xaml` with the template path, the recipients / CC / BCC arrays, the attachment folders and files, and the replacement dictionaries (any of them may be `Nothing`).

### Template conventions
- **Text:** `{{key}}` in the HTML is replaced by `in_TextReplacements(key)`.
- **Image:** `{{key}}` is replaced by an `<img src="data:image/<ext>;base64,…">` built from the file path in `in_ImageReplacements(key)`.
- **Table:** `{{key}}` is replaced by an HTML table built from the DataTable in `in_TableReplacements(key)`.
- **Subject:** the content of the **first `<h1>`**, taken after the text replacements, so it can contain placeholders. Match is exact: `<h1>…</h1>` on a single line, no attributes.
- **CSS:** the `<style>` tag contains only a comment holding the path of the stylesheet, e.g. `<style>/*Data/HtmlTemplates/styles.css*/</style>` (see below). The path is relative to the project folder.
- **Attachments:** only the top level of each folder is read, plus the individual files. Paths that don't exist are logged as Warn and skipped.

## Design decisions

### Subject defined in the template
The subject is the template's `<h1>`, so the subject and the body are maintained in one file (Diary 2025.05.01). The caller doesn't pass a subject.

### CSS inlining (workaround for external CSS)
Most email clients (Gmail, Outlook…) ignore `<link rel="stylesheet">`. `Metadata_BuildBody` inlines the stylesheet at runtime, so every template can share one `.css` file:
1. Read the template.
2. Take the first `<style>…</style>` block — regex `<style[^>]*>([\s\S]*?)<\/style>`.
3. Take the first `/* … */` comment inside it — regex `\/\*([\s\S]*?)\*\/` — and strip `/*` `*/` to get the path.
4. Read that CSS file and replace the comment with its content.

Benefits: one stylesheet for every template, and the output works in email clients. A `<link>` tag left in a template is harmless but has no effect.

### Split between metadata and engine
The `Metadata_*` workflows prepare content that doesn't depend on the engine. The engine workflows only send it.

### Errors propagate from SendEmail
`SendEmail.xaml` and the Metadata / engine workflows don't catch; the caller decides. The Notifications sub-module is the exception: it catches everything, because it runs in Main's transition actions (see [Notifications.md](Notifications/Notifications.md)).

## Extending — adding an engine
1. Create `SendEmail_EngineOptions/SendEmail_<Engine>.xaml` with the same arguments as the Integration Service one: `in_Recipients`, `in_CC`, `in_BCC` (`String[]`), `in_Subject`, `in_Body` (HTML), `in_Attachments` (`String[]`).
2. Invoke it in the `Send email` sequence of `SendEmail.xaml`. There is no config switch between engines yet.

## Limitations / roadmap
- The engine is fixed to Integration Service. SMTP is a stub: `in_Recipients` is a `String`, with no CC / BCC / attachments and no server settings.
- The Integration Service connection is fixed at design time (ConnectionId). Rebind it per environment through the package bindings.
- Missing files (template, stylesheet, image) throw. A template without `<style>` or without `<h1>` fails. "Verify if the image exists" is still a TODO.
- Base64 `data:` images are blocked by many clients (e.g. Gmail, Outlook desktop). Check the target clients, or use CID attachments.
- The MIME type comes from the file extension (`.jpg` → `image/jpg`; the standard type is `image/jpeg`).
