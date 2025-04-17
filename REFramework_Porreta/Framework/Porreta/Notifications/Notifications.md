# Workaround for External CSS in Email Templates

## Purpose

Most email clients (like Gmail, Outlook, etc.) do not support external CSS via `<link rel="stylesheet" href="...">` for security reasons. To overcome this limitation, we implement a dynamic solution that **inlines the contents of an external `.css` file** into the `<style>` tag of the HTML template **at runtime**.

## How It Works

Each HTML email template includes a comment inside its `<style>` tag that indicates the path to the associated external CSS file. During execution, the following steps occur:

1. **Load HTML Template:**  
   The HTML template is loaded as a string.

2. **Extract CSS File Path:**  
   A regular expression is used to extract the path to the external CSS file from the comment within the `<style>` tag.  
   **Example Regex:**  
   ```regex
   /\*\s*path:\s*(.*?)\s*\*\//
    This regex captures any text following /* path: and preceding */.

3. **Read the CSS File:**
    The corresponding CSS file is read from disk (or storage).

4. **Inline CSS Content:**
    The contents of the CSS file are injected directly into the <style> tag of the HTML template.

5. **Final HTML Output:**
    The final HTML contains all CSS definitions inlined inside the <style> tag, ensuring compatibility with email clients.

## Benefits
* Maintainability: External CSS remains in a single .css file, making it easy to update styles across templates.

* Reusability: The logic works for any HTML template as long as the comment is included properly.

* Email-safe output: Results in a fully inlined <style> section that works across email clients like Gmail, Outlook, etc.


# SendEmail_BusinessExceptionForTransaction.xaml

## Purpose
This workflow is responsible for preparing and sending a notification email when a Business Rule Exception occurs during the processing of a transaction item.

## Input Arguments
- in_Config (InArgument<Dictionary<String, Object>>)
Dictionary structure to store configuration data of the process, including settings, constants, and asset values used during the email notification setup.

- in_BusinessException (InArgument<BusinessRuleException>)
The Business Rule Exception that triggered the error handling process.

- in_TransactionItem (InArgument<QueueItem>)
The transaction item being processed at the time the Business Rule Exception was thrown.

## Output Arguments
(None)

## Workflow Structure
The workflow begins by checking a configuration toggle that enables or disables the sending of business exception notifications. If enabled, it builds a dictionary of text replacements using the exception and transaction data, then invokes a reusable email-sending workflow with the prepared arguments.

## Error Handling
This workflow does not explicitly throw new exceptions. It assumes the exception has already occurred and is being handled. The process includes a `Return` activity that safely exits early if the notification toggle is disabled.

