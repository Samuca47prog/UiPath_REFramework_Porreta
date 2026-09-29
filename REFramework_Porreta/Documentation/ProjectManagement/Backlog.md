# Backlog — fixes from the project review (2026.09.24)

Items found during the full review of XAMLs, annotations and markdowns. Tick an item when it's fixed, and also remove it from the *Limitations / roadmap* section of the linked note.
Priority: **P1** = wrong behavior / can crash the job · **P2** = misleading or fragile · **P3** = cleanup.

## 1. Code — Main / Framework

- [x] **P1 · Notifications without try/catch** — `Main.xaml`, the 3 `SendEmail_Exception` invokes in transition actions. If the email fails (Integration Service, template, CSS), the transition throws, End Process never runs and applications are not closed. Wrap each invoke in a try/catch that logs the failure. → [Main.md](../Main.md)
> Solved this by adding a try catch in the whole SendEmail_Exceptions.xaml

- [ ] **P1 · Config load failure crashes Main** — `Main.xaml` Initialization. If `InitAllSettingsJson` throws (e.g. asset not found), `Configjson` stays Nothing.
  - ~~The Exception transition then calls `SendEmail_Exception` with a null config → NullReferenceException in the transition.~~ Covered since 2026.09.29: the NullReferenceException is caught inside `SendEmail_Exception` and logged at Error.
  - Still open: the Exception transition condition `Cint(Configjson("Max")(...))` throws on an SE while loading, and End Process's Faulted rule `CBool(Configjson("FlowControl")(...))` throws after a BRE while loading. Guard both with `Configjson IsNot Nothing`. → [Main.md](../Main.md)
> Solved this by
> 1. Setting Config Asset as dependency or Use Local Config.
> 2. Adding FlowControl flag in config to throw or not when asset is not found in InitAllSettingsJson

- [-] **P1 · Asset key resolution** — `InitAllSettingsJson.xaml`, For Each Assets. `currentAsset.First.ToString` (the asset **name**) is used as the root key. It should be `CType(currentAsset, JProperty).Name` as the key and `.Value` as the asset name. It works today only because key = value (`ProcessToKill`). → [Config.md](../Config.md)
> This is working fine


- [ ] **P2 · Faulted reason loses the exception** — `Main.xaml` End Process → Terminate Workflow. After a Process SE, Initialization clears `SystemException`, so `Exception=[SystemException]` is Nothing and the reason says "System Exception was cleaned". Keep a `LastException` variable, set in the Process catches and the Initialization catches, and use it in Terminate (then update `Main_SEinProcess_TestCase`).
- [ ] **P2 · Wrong counts in the Faulted reason** — the reason prints `TransactionNumber` and `SuccessfulTransactionCount`, which both have a +1 offset, and the text says "processed". Print `TransactionNumber - 1` and `SuccessfulTransactionCount - 1`.
- [ ] **P2 · Max consecutive system exceptions gets retried** — `Main.xaml` Initialization. The "Throw Consecutive Exceptions exceeded" exception is caught as an init SE, so the init retry runs and `Exception_inInit` is sent. Throw it as a separate, identifiable exception (or set a flag) and skip the retry in that case.
- [ ] **P2 · `SystemExceptionsAtInitCount` never reset** — it counts across the whole job. After a Process SE, any later init failure has fewer retries left. Decide: reset it on a Successful init transition, or document it as a per-job budget.
- [ ] **P2 · Init transitions depend on their order** — the Retry condition `SystemException isNot Nothing` overlaps with the Exception transition. Make it exclusive: `SystemException IsNot Nothing AndAlso BusinessException Is Nothing AndAlso SystemExceptionsAtInitCount < CInt(Configjson("Max")("SystemExceptionsAtInitialization"))`.
- [ ] **P2 · KillAllProcesses not run before an init retry** — the docs promised it. It only runs in the first-run branch. Add it to the Retry transition action, or drop the promise (already removed from Main.md).
- [ ] **P3 · BRE in Initialization logged at Trace** — `Main.xaml`, catch "Business exception at initialization". Change to Error.
- [ ] **P3 · Typo in log text** — "Retrying initialization. Attemp #". If you fix it, update `Main_SEinInitialization_TestCase`, which asserts this text.
- [ ] **P3 · Empty placeholder** — `Main.xaml` End Process "Send Email - EndProcess" (summary email). Implement it or remove it.
- [ ] **P3 · Stale DisplayNames after the move** — the 3 notification invokes in `Main.xaml` and `Mocks/Main_mock.xaml`, and the invoke in `SendEmail_Exception_TestCase`, still say `(Framework\Porreta\Notifications\SendEmail_Exception.xaml)`. Inside `SendEmail_Exception.xaml`, the SendEmail invoke says `(Reusables\Email\SendEmail\SendEmail.xaml)`. The `WorkflowFileName`s are correct; only the labels are wrong.
- [ ] **P3 · Screenshot name uses 12-hour time** — `TakeScreenshot.xaml` `"yyMMdd.hhmmss"` → `HHmmss`.
- [ ] **Decision · Missing asset** — `InitAllSettingsJson.xaml` throws a BusinessRuleException on "Could not find the asset" (no retry) and only logs a Warn for every other failure (REF throws). The message matching is also a string compare that depends on language. Decide the policy.
- [ ] **Decision · Asset folder** — all assets are read from `in_OrchestratorQueueFolder`. REF reads a folder per asset (`OrchestratorAssetFolder`). Keep and document it (done in Config.md), or support `{ "Name": "...", "Folder": "..." }` entries.

## 2. Code — Email / Notifications

- [ ] **P1 · Recipient lists split on spaces** — `SendEmail_Exception.xaml`, 5 arguments (Recipients, CC, BCC, AttachmentFolders, AttachmentFiles) use `Split(x)`, whose default delimiter is a space. Use `x.Split(";"c, StringSplitOptions.RemoveEmptyEntries).Select(Function(s) s.Trim).ToArray` (this also removes the `{""}` from empty values). → [Notifications.md](../../Reusables/Email/Notifications/Notifications.md)
- [ ] **P2 · Config `Subject` unused** — the subject comes from the template `<h1>`. Remove the key from `Config.json`, or make it an override when it isn't empty.
- [ ] **P2 · Image placeholder not replaced** — `{{Wally_robot_image}}` in the BRE / SE / Init templates stays as literal text because `SendEmail_Exception` doesn't pass `in_ImageReplacements`. Pass it or remove it from the templates.
- [ ] **P2 · Metadata_BuildBody crashes on template variations** — no `<style>` tag, no `/* */` comment, or no `<h1>` → null FirstMatch → NullReferenceException. `<h1 class=…>` or a multi-line h1 doesn't match. Handle a null match (keep the body without CSS, use an empty or default subject) and relax the h1 regex (`<h1[^>]*>([\s\S]*?)</h1>`). → [Email README](../../Reusables/Email/README.md)
- [ ] **P2 · Missing image file throws** — TODO "Verify if the image exists" in `Metadata_BuildBody` Images loop. Check that the file exists and log a Warn.
- [ ] **P2 · Base64 images blocked by clients** — Gmail and Outlook desktop block `data:` images. Test the target clients, and consider CID inline attachments.
- [ ] **P3 · MIME type** — `.jpg` → `image/jpg`; the standard type is `image/jpeg`. Map the extension (and update `Metadata_BuildBody_TestCase`, which asserts `data:image/jpg`).
- [ ] **P3 · SMTP engine stub** — `SendEmail_SMTP.xaml`: `in_Recipients` is a `String`, with no CC / BCC / attachments and no server config. Align its arguments with the Integration Service engine. Add an engine switch in config.
- [ ] **P3 · Leftover `<link rel="stylesheet">`** in `BRE_Body.html` and `ExceptionInit_Body.html`. It has no effect; remove it.

## 3. Code — OrchApi / Snippets (in development)

- [ ] **P1 · HTTP error check** — `responseStatus Mod 100` → `responseStatus \ 100` in `API_HTTPS.xaml`, `OrchApiHttps_GetJobs`, `GetThisJob`, `GetToken`, `RestartJob`. Today 400/401/403/500 are not caught, while 204/304 are. → [OrchApi README](../../Reusables/OrchApi/README.md)
- [ ] **P1 · GetThisJob `out_JobJson` never assigned** — ShouldReleaseThisMachine would crash on it. Also, `/odata/jobs(<Key>)` passes the GUID where the numeric Id is expected. Use `$filter=Key eq <guid>` or the Id.
- [ ] **P1 · RestartJob** — `StatusCode` and `Result` are not bound, so it always throws "not mapped".
- [ ] **P2 · GetToken** — a stray `Parameters = POST` form parameter; the default `in_Url` points to staging; the annotation links to the DU API guide instead of the Orchestrator external apps docs.
- [ ] **P2 · Unmapped status code throws a different exception** — the snippet throws `BusinessRuleException`, the OrchApi copies throw `SystemException`. Pick one.
- [ ] **P2 · ShouldReleaseThisMachine** — "Create this job again" is empty; pending jobs usually have no `HostMachineName` (the "no specific machine" case isn't handled); confirm what the `SpecificPriorityValue` values mean; decide where Main calls it.

## 4. Tests

- [ ] **P2 · SendEmail_Exception_TestCase can't fail** — `Tests/Reusables/Email/Notifications/`. The *Then* sequence is empty, and since the Try Catch was added, a failed send is only logged. Add a check, e.g. read the newest email received (like the `SendEmail_*` tests) or assert that no `Send Email Exception Failed` Error was logged. A second case with a broken config block would check that the workflow doesn't throw.
- [ ] **P2 · Kill tests can't fail** — `KillAllProcesses_TestCase` / `KillProcesses_TestCase` compare `Process.ProcessName` (`notepad`) with `notepad.exe`. Strip `.exe` in the assertion. Also check whether the Kill Process activity accepts `.exe`, or change the config / asset to `notepad`.
- [ ] **P3 · Remove legacy Excel tests** — `InitAllSettingsTestCase`, `GetTransactionDataTestCase`, `ProcessTestCase`, `InitAllApplicationsTestCase`, `MainTestCase`, `WorkflowTestCaseTemplate` (or rewrite the template for `ConfigJson`).
- [ ] **P3 · Tests to add** — config load failure (asset not found) in Initialization; notification failure does not stop the job; multiple recipients split by `;`.
- [ ] **P3 · `Mocks/mock_config.json`** has an empty `mockedWorkflows`. Check that Studio links `Main_mock.xaml` to `Main.xaml`.

## 5. Annotations (the contract — see [AGENTS.md](../../AGENTS.md))

- [ ] `SendEmail.xaml` — remove `in_Subject`; add `in_Recipients`, `in_CC`, `in_BCC`; make Error Handling accurate (missing HTML / CSS / image files throw).
- [ ] `Metadata_BuildBody.xaml` — `out_Subject` is an OutArgument; list the throws; replace the long CSS annotation on "replace styles section" with a pointer to the Email README.
- [ ] `SendEmail_Exception.xaml` — add a root docstring (it has none): arguments, the `Exception_inInit` origin, `in_TransactionItem` can be Nothing, and Error Handling = catches everything, logs at Error, never rethrows. Fix the `in_TransactionItem` argument annotation (it says "Business Rule Exception" only). The DEV NOTE 2026.09.29 on the Try Catch can then become a one-line pointer to Notifications.md.
- [ ] `InitAllSettingsJson.xaml` — add a root docstring: local vs asset, folder source, UpdateLocalConfig, assets resolved and removed, BRE on a missing asset.
- [ ] `KillAllProcesses.xaml` — `in_Config` → `in_ConfigJson`; invoke display name `Framework\KillProcesses.xaml` → `Reusables\KillProcesses.xaml`; add the "separated by `;`" usage.
- [ ] `RetryCurrentTransaction.xaml` — "Config.xlsx" → the `Max.RetryNumber` key in Config.json.
- [ ] `KillProcesses.xaml` — headings `#` → `##` (standard format).
- [ ] `SendEmail_IntegrationsService.xaml`, `SendEmail_SMTP.xaml`, OrchApi workflows, `ShouldReleaseThisMachine.xaml` — add docstrings (OrchApi can wait until it's stable).
- [ ] `Main.xaml` — short annotations on the Initialization Retry / Exception transitions and the 3 notification transitions, pointing to `Documentation/Main.md`; typo "Enhenced".
- [ ] Resolve or move the `DEV NOTE: 2026.09.23` annotations (InitAllSettingsJson, ModifyConfigJsonForThisTestCase, Main_SEinInitialization_TestCase).

## 6. Config.json comments

- [ ] `ShouldMarkJobAsFaulted` — replace the REF text with the real rule: Faulted if any transaction failed (BRE included) or Initialization failed.
- [ ] Root `ProcessToKill` — remove "if in_ProcessToKill is null" (no such argument); say the `ProcessToKill` asset overwrites it.
- [ ] `Enabled` (3×) — "will trigger business/system exception" → "sends the notification email".
- [ ] `Exception_inInit` — comments were copy-pasted from the system exception block; rewrite them for Initialization.
- [ ] `Subject` — mark it `/* UNUSED */` or remove it (see §2).
- [ ] Trailing comma after `"ConsecutiveSystemExceptions": 0,` — Newtonsoft accepts it; remove it anyway so the file is closer to standard JSON.
- [ ] Consider real booleans for `Enabled` (`true` instead of `"True"`). `CBool` handles both.

## 7. Project cleanup

- [ ] `entry-points.json` points to `Tests\Framework\Main.xaml`, which doesn't exist. Fix it or delete it.
- [ ] Remove `Config.xlsx` and `Framework/InitAllSettings.xaml` (legacy Excel config).
- [ ] Delete the 40 screenshots in `Exceptions_Screenshots/` and git-ignore that folder, keeping `placeholder.txt`.
- [ ] Delete the empty folders `Documentation/Data` and `Documentation/Framework`.
- [ ] Check whether `UiPath.MicrosoftOffice365.Activities 2.8.11-preview` can move to a stable version.
