# Tests

## Approach
- **Unit tests** for reusables (`Tests/Reusables/...`, including `Tests/Reusables/Email/Notifications/`) and framework pieces (`Tests/Framework/...`, `Tests/Porreta/...`). The test folders mirror the source folders.
- **Main tests** (`Tests/Main/...`) run [Mocks/Main_mock.xaml](../Mocks/Main_mock.xaml). It is a copy of Main where two invokes are mocked to force exceptions:
  - `InitAllApplications` → throws depending on `Configjson("Test")("Scenario")`: `BREinInitialization` / `SEinInitialization`.
  - `Process` → throws depending on the queue item's `SpecificContent("Exception")`: `BRE` / `SE`.
- Keep `Main_mock.xaml` in sync with `Main.xaml`. Apart from the two mocks they should be identical.

## Per-test config
Tests that need different config values use the [ModifyConfigJsonForThisTestCase](../Code_Snippets/README.md) pattern: they load the local config, change keys (e.g. `Test.Scenario`, `Max.SystemExceptionsAtInitialization`, `FlowControl.ShouldMarkJobAsFaulted`), upload it to the **`ConfigJson_Testing`** asset, and run Main with `in_ConfigJsonAssetName = "ConfigJson_Testing"`.
This asset is shared, so tests are meant to run **one at a time** (on purpose).

## Main scenarios
| Test | How it is triggered | Checks |
|---|---|---|
| `Main_BREinInitialization_TestCase` | `Test.Scenario = BREinInitialization` | Exception message ends with `Forced BRE in initialization` |
| `Main_SEinInitialization_TestCase` | `Test.Scenario = SEinInitialization` | SE raised **and** the log contains `Retrying initialization. Attemp #<Max>` |
| `Main_BREInProcess_TestCase` | Queue item with `Exception = BRE` | Exception message ends with `Forced BRE in process` (job Faulted) |
| `Main_SEinProcess_TestCase` | Queue item with `Exception = SE` | Terminate reason contains `System Exception was cleaned` — this checks current behavior, see [Main.md](../Documentation/Main.md) |
| `Main_SEinProcess_SuccessJobPerConfigFlag_TestCase` | Queue item with `Exception = SE` + changed `ShouldMarkJobAsFaulted` | No exception (job not Faulted) |

The Main tests also send the real notification emails when `Enabled` is true in the testing config. A failed email is logged at Error and doesn't change the result of these tests.

## Notes
- The `SendEmail_*` tests really send the email through the Integration Service connection, then check the newest email received.
- `SendEmail_Exception_TestCase` (`Tests/Reusables/Email/Notifications/`) forces `Enabled = True`, builds a BRE and a queue item, and invokes the workflow, origin from `in_ExceptionOrigin` (default `BRE_inProcess`). Its *Then* is empty, and the workflow never throws, so the test passes even when no email is sent. Check the logs / inbox manually until an assertion is added (Backlog §4).
- `KillAllProcesses_TestCase` / `KillProcesses_TestCase` compare `Process.ProcessName` (no extension) with the configured names. If the config has `notepad.exe`, the assertion passes even when nothing was killed.
- Legacy REF tests are planned for removal: `InitAllSettingsTestCase`, `GetTransactionDataTestCase`, `ProcessTestCase`, `InitAllApplicationsTestCase`, `MainTestCase`, `WorkflowTestCaseTemplate`. They still use the Excel `Config` dictionary.
