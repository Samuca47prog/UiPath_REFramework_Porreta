# Config JSON

The project keeps its configuration in a **JSON object** (`Configjson`, type `JObject`) instead of REF's `Config.xlsx` dictionary. Keys are grouped by topic.
Each key is described by its inline comment in [Data/Config.json](../Data/Config.json). This note covers only the structure and how the config is loaded.

## Structure

| Group | Content |
|---|---|
| `Orchestrator` | Queue name and folder (the Main arguments can override both) |
| `LogF` | Values for custom log fields (`BusinessProcessName`) |
| `FlowControl` | Behavior flags (`UpdateLocalConfig`, `ShouldMarkJobAsFaulted`) |
| `Path` | Folders used by the framework (screenshots) |
| `ProcessToKill` (root) | Processes killed by `KillAllProcesses.xaml`, separated by `;`. The asset of the same name overwrites it |
| `Email.Exception.<origin>` | Settings for each notification email — see [Notifications.md](../Reusables/Email/Notifications/Notifications.md) |
| `Test` | `Scenario`, read by `Mocks/Main_mock.xaml` — see [Tests/README.md](../Tests/README.md) |
| `Max` | Retry and exception limits (transaction retry, Initialization retries, consecutive system exceptions) |
| `Log` | Fixed parts of log messages |
| `Retry` | Retry counts for Get Transaction Item / Set Transaction Status |
| `Assets` | `"<key>": "<Orchestrator asset name>"` — loaded at runtime, then removed (see below) |

Access pattern: `Configjson("Group")("Key")`. Values are `JToken`, so convert them explicitly (`.ToString`, `CBool(...)`, `CInt(...)`).

## Source: local file or asset

The source is chosen by the `in_ConfigJsonAssetName` argument of Main.xaml:
- **Empty** → local file `Data\Config.json`.
- **Filled** → a text asset with that name, read from the folder in `in_OrchestratorQueueFolder`. If that argument is empty, the job's own folder is used.

## Loading (`Framework/InitAllSettingsJson.xaml`)

1. Read the JSON string: `Get Asset` inside a Retry Scope, or `Read Text File` for the local file.
2. `Deserialize JSON` → `JObject`. Newtonsoft is used, so `/* comments */` and trailing commas are accepted. The file is **JSONC**, not strict JSON, and standard JSON validators will reject it.
3. **UpdateLocalConfig**: if `FlowControl.UpdateLocalConfig` is true **and** an asset was used, the asset string is written to `Data\Config.json`. Purpose: bring the latest asset value into the code during development.
4. **Assets**: for each entry in `Assets`, `Get Asset` is run (folder = `in_OrchestratorQueueFolder`) and the value is stored at the **root** of the config.
   - Asset not found (`"Could not find the asset"`) → `BusinessRuleException`, so Initialization ends without a retry.
   - Any other failure → logged as Warn and ignored.
   - If there is no `Assets` key, this step is skipped (the testing config has already had it removed).
5. The `Assets` key is removed from the in-memory config.
6. Back in Main: a non-empty `in_OrchestratorQueueName` / `in_OrchestratorQueueFolder` overwrites `Orchestrator.QueueName` / `Orchestrator.QueueFolder`.

### Asset folder — difference from REF
In REF, each asset row has its own `OrchestratorAssetFolder` column, and `in_OrchestratorQueueFolder` only affects the queue. Here **all** assets, and the config asset itself, are read from `in_OrchestratorQueueFolder`.

## Excel vs JSON

| | Excel (`Config.xlsx`) | JSON |
|---|---|---|
| Structure | Flat sheets (Settings / Constants / Assets) | Nested groups |
| Types | Values arrive as strings/objects | Numbers and booleans are native (some flags are still stored as strings, e.g. `"Enabled": "True"`) |
| Version control | Binary, hard to diff | Text, easy to diff and review |
| Editing | Easy for non-technical users | Requires editing text |
| Comments | Description column / cell notes | Inline `/* */` (JSONC) |
| Asset folder | Per asset | One folder for all (see above) |

## Limitations / roadmap
- **Asset key resolution:** the root key is currently taken from the asset's **name** (the entry's value), not from the JSON key. It works today only because the key and the value are the same (`"ProcessToKill": "ProcessToKill"`).
- If loading fails, `Configjson` stays Nothing and Main still dereferences it (see [Main.md](Main.md) — Limitations).
- `Config.xlsx` and `Framework/InitAllSettings.xaml` are leftovers from REF and are planned for removal.
- Handling of a missing asset (business exception vs system exception) is still under review.
