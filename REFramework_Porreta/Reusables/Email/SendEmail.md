# SendEmail

## Purpose
This workflow is responsible for composing and sending email with dynamic content and optional attachments. It prepares the email body using an HTML template with variable replacements and builds the attachments list from given folders and files before sending the final message.

## Input Arguments
- in_BodyHtmlFilePath (InArgument<String>)
  Path to the baseline HTML file used as the template for the email body.

- in_Recipients (InArgument<String[]>)
  List of emails that will receive the email.

- in_CC (InArgument<String[]>)
  List of emails that will receive the email as CC.

- in_BCC (InArgument<String[]>)
  List of emails that will receive the email as BCC.

- in_AttachmentsFolders (InArgument<String[]>)
  Array of folder paths to be searched for attachments. Only files at the top level are considered (non-recursive).

- in_AttachmentsFiles (InArgument<String[]>)
  Array of individual file paths that should be attached to the email if they exist.

- in_TextReplacements (InArgument<Dictionary<String, String>>)
  Dictionary containing text placeholders and their replacement values. Placeholders should be in the format `{{key}}` in the HTML file.

- in_ImageReplacements (InArgument<Dictionary<String, String>>)
  Dictionary mapping image placeholders to their file paths. The workflow will embed them as Base64-encoded images.

- in_TableReplacements (InArgument<Dictionary<String, DataTable>>)
  Dictionary of table placeholders and corresponding `DataTable` objects to be rendered as HTML tables in the email.

## Output Arguments
- None

## Workflow Structure
The workflow first defines the recipients list and invokes the `Metadata_BuildBody` workflow to generate the final HTML content using provided replacements. Then, it calls the `Metadata_Attachments` workflow to build the attachments list from folders and files. Finally, it invokes `SendEmail_IntegrationsService` to send the email with all components (subject, body, recipients, attachments, CC, BCC).

## Error Handling
Each invoked workflow handles its own internal validations. Missing folders or files for attachments trigger warning logs in `Metadata_Attachments`, and invalid replacements in `Metadata_BuildBody` are handled without throwing exceptions. The main workflow does not explicitly throw errors but relies on child workflows to manage exceptions internally.


# Metadata_BuildBody

## Purpose
This workflow is responsible for preparing an HTML email body. It reads an HTML template file and performs dynamic replacements (text, images, tables) based on the provided input arguments.

Variables can be added to be replaced in the body using tag {{variable}} inside html and a dictionary with the mapped value

## Input Arguments
- in_BodyHtmlFilePath (InArgument<String>)
Path to the baseline HTML file that serves as a template.

- in_TextReplacements (InArgument<Dictionary<String, String>>)
 Dictionary of keys and values to replace placeholders in the HTML.

- in_ImageReplacements (InArgument<Dictionary<String, String>>)
Dictionary mapping keys to image file paths.

- in_TableReplacements (InArgument<Dictionary<String, DataTable>>)
Dictionary mapping keys to DataTable objects.

## Output Arguments
- out_Body (OutArgument<String>)
Final HTML content after performing all replacements.

- out_Subject (InArgument<String>)
  Subject line of the email to be sent.
  defined by the first h1 content in the body html

## Workflow Structure
The Build Body sequence constructs the final HTML email body by reading a template file and performing dynamic replacements for text, images, and tables, if provided. It iterates through the provided dictionaries to replace placeholders in the template with the corresponding values (e.g., text replacements, Base64-encoded image tags, or HTML tables). Key activities include conditional checks for replacements, iterative updates to the body content, and the dynamic generation of HTML elements.


## Error Handling


# Metadata_Attachments

## Purpose
This workflow is responsible for building a consolidated list of file paths (attachments) by validating and collecting files from specified folders and directly provided file paths.

## Input Arguments
- in_Folders (InArgument<String[]>)
  Array of folder paths to be checked. If a folder exists, its immediate file contents will be added to the attachments list. This is a non-recursive search.

- in_Files (InArgument<String[]>)
  Array of individual file paths. If the file exists, it will be appended to the attachments list.

## Output Arguments
- out_AttachmentsPaths (OutArgument<List<String>>)
  List containing the full paths of all files that exist and were collected from both folder and file inputs.


## Workflow Structure
The workflow begins by initializing an empty list of attachments. It iterates through each folder in `in_Folders`, checks for existence, retrieves the file paths, and merges them into the output list. It then processes each file in `in_Files`, validating their existence and adding valid ones to the same list. All invalid paths trigger a log message.

## Error Handling
The workflow handles missing folders and files gracefully using `FolderExistsX` and `FileExistsX` activities. If a folder or file does not exist, a warning is logged using `Log Message`, and the execution continues without interruption or exception throwing.


