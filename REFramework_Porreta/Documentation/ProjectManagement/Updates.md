## Cycle 2

### Send Email
THIS MODULE IS NOT COMPLETE

This module intends to map commom email caracteristics, not the way the email is sent.
By doing that, the way email is sent, by Integration Service, SMTP, Office 365, etc, will affect its only workflow.

Body:
- Body is defined by html file
- Variables can be added to be replaced in the body using tag {{variable}} inside html and a dictionary with the mapped value
- These variables can be simple texts, Images or tables.

Attachments:
- Attachments can be caught from a folder or list of files (always combined, not exclusive)

Recipients:
TBD

To have an implementation to test, module is using Integration Service connection to send emails.

### Doc String template
I've included a prompt for documenting Workflow files based in their xaml code.
It is not fancy else than the prompt. For now, you have to copy the xaml code, insert it to the prompt, and paste it in your favorite GenAi chat.

### HTTP Requests code snippet
API_HTTPS.xaml is a code snippet used to speed up implementations of http requests.

This is a code snipet because:
* http requests have many different parameters, which would make it harder to wrap in a workflow
* A project may have more than one http request needed, so one workflow could not be easily used for different http requests like GETs, POSTs, PUTs, etc.

#### How to use?
After adding the Code_Snippets folder, you create a new workflow for the intended http request.

An implementation of the snippet can be found at [Reusables/ORCH_API/ORCH_API_GetToken.xaml](https://github.com/Samuca47prog/UiPath_REFramework_Porreta/blob/main/REFramework_Porreta/Reusables/ORCH_API/ORCH_API_GetToken.xaml)

## Cycle 1

### KillAllProcess.xaml

#### Implementation
KillAllProcess is used to kill all process defined in key "ProcessToKill" of Config, which is defined via Asset.

KillProcess.xaml is a sub workflow implemented to kill a list of given process names.
It uses Kill Process activity surrounded by a try catch to retrieve exceptions in case of failute to kill process.

#### How to use?
- If you want to **kill any given processes**, use KillProcess.xaml passing the list of process names as ```{"Notepad", "mspaint"}```
- If you want to **kill all the processes**, make sure your processes names are ```;``` separated at ProcessToKill asset and invoke KillAllProcesses.xaml  
Asset ```ProcessToKill``` name is defined in Config Assets sheet.