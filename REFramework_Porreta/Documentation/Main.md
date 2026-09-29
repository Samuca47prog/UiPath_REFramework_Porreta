# Main.xaml — state machine

Main keeps REF's four states. This note describes what Porreta changes or adds. The workflow and variable annotations inside `Main.xaml` describe each piece.

| State | Change compared to REF |
|---|---|
| Initialization | Loads the JSON config, **retries on system exception**, sends a notification on failure |
| Get Transaction Data | Unchanged (Check Stop Signal → GetTransactionData) |
| Process Transaction | Sends notifications on BRE / SE, counts successful transactions |
| End Process | Can **mark the job as Faulted** |

## Initialization

On every entry: `SystemException` is reset. On the **first run** only (`Configjson Is Nothing`):
1. Log the screen resolution.
2. `InitAllSettingsJson.xaml` → `Configjson` (see [Config.md](Config.md)).
3. Main arguments override the queue name and folder.
4. `SuccessfulTransactionCount = 1`.
5. `KillAllProcesses.xaml`.
6. Add log field `logF_BusinessProcessName`.

On every entry after that: the max consecutive system exceptions check (throws if reached), then `InitAllApplications.xaml`.

### Transitions
| Transition | Condition | Action |
|---|---|---|
| Successful → Get Transaction Data | no BRE and no SE | — |
| Exception (failed initialization) → End Process | BRE, **or** SE with `SystemExceptionsAtInitCount >= Max.SystemExceptionsAtInitialization` | Notification `Exception_inInit` |
| System exception (Retry) → Initialization | SE | `SystemExceptionsAtInitCount += 1`, log `Retrying initialization. Attemp #n` |

- `Max.SystemExceptionsAtInitialization = N` means **N retries** (N+1 attempts). `0` means no retry.
- The Retry condition overlaps with the Exception condition, so the **order of the transitions matters**: Exception must be evaluated first.
- A business exception in Initialization is never retried.

## Process Transaction

| Transition | Action |
|---|---|
| System Exception → Initialization | Notification `SE_inProcess` |
| Business Exception → Get Transaction Data | Notification `BRE_inProcess` |
| Success → Get Transaction Data | `SuccessfulTransactionCount += 1` |

## Notifications

The three notifications call [SendEmail_Exception.xaml](../Reusables/Email/Notifications/SendEmail_Exception.xaml) directly in the transition action, with no try/catch in Main. This is safe because the workflow catches every exception itself and only logs it at Error: a failed email never interrupts the transition, so the state machine continues (End Process still runs). Details: [Notifications.md](../Reusables/Email/Notifications/Notifications.md).

## End Process

1. `CloseAllApplications.xaml`. If it fails → `KillAllProcesses.xaml`.
2. `Send Email - EndProcess`: an empty placeholder (planned: summary email on success / failure).
3. **Finally — Faulted rule:** if `FlowControl.ShouldMarkJobAsFaulted` is true **and** (`SystemException` or `BusinessException` is set, **or** `TransactionNumber > SuccessfulTransactionCount`) → `Terminate Workflow`, so the job ends as **Faulted**.
   In practice the job is Faulted if **any transaction failed** (BRE included) or Initialization failed.

## Counters
| Variable | Starts | Meaning |
|---|---|---|
| `TransactionNumber` | 1 | Number of the **next** transaction (REF) |
| `SuccessfulTransactionCount` | 1 (first run) | Successful transactions + 1 (same offset as `TransactionNumber`, so the two can be compared) |
| `SystemExceptionsAtInitCount` | 0 | Initialization retries so far. **Not reset** during the job |
| `ConsecutiveSystemExceptions` | 0 | REF counter, reset on success / BRE |

## Limitations / roadmap
- **Config load failure** (`Configjson` stays Nothing) still breaks the job, even though the init notification no longer throws:
  - SE while loading (e.g. file not found): the Exception transition condition calls `Cint(Configjson("Max")(...))` → NullReferenceException in the condition, before End Process.
  - BRE while loading (asset not found): the condition is true without touching `Configjson`, the notification fails silently (logged at Error), End Process runs, then the Faulted rule's `CBool(Configjson("FlowControl")(...))` throws → the job ends Faulted with a NullReferenceException instead of the BRE message.
- By the time End Process runs, a Process system exception has already been cleared by Initialization. The Terminate reason then says `System Exception was cleaned`, and the counts it prints are one too high because of the +1 offset.
- The max consecutive system exceptions throw happens inside Initialization, so the init retry catches it and retries it.
- KillAllProcesses runs only on the first run, not before each Initialization retry.
- The DisplayNames of the three notification invokes still say `Framework\Porreta\Notifications\SendEmail_Exception.xaml`; the `WorkflowFileName` is correct (`Reusables\Email\Notifications\`). Same in `Mocks/Main_mock.xaml`.
