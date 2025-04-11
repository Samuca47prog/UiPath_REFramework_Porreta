
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

