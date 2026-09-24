# 📁 Config JSON in REFramework

This project uses a **JSON file** to manage configuration settings in place of the traditional `Config.xlsx`. The JSON structure organizes keys into hierarchical groups, improving readability and maintainability.

## 🧾 JSON Example

Below is a simplified version of the structure:

```json
{
  "Orchestrator": {
    "QueueName": "TestQueue", /* Orchestrator queue Name */
    "QueueFolder": "Porreta" /* Folder where the queue is defined */
  },
  "LogF": {
    "BusinessProcessName": "Framework" /* Log grouping name */
  },
  "FlowControl": {
    "UpdateLocalConfig": false,
    "ShouldMarkJobAsFaulted": false
  },
  ...
  "Assets": {
    "ProcessToKill": "ProcessToKill"
  }
}
```

---

## 🔁 How Assets Are Handled

- The `Assets` key in the JSON works **exactly like the "Assets" sheet in Excel**.
- All keys under `Assets` are:
  1. Retrieved from Orchestrator using the `Get Asset` activity.
  2. Placed in the **root level** of the configuration dictionary (e.g., `config("ProcessToKill")`).
  3. After loading, the `"Assets"` section is **removed** from the in-memory JSON config.

This makes asset access seamless and backward-compatible with the REF logic.

---

## 📊 Excel vs JSON — Comparison

| Feature                      | Excel (`Config.xlsx`)                                 | JSON (New Approach)                                      |
|-----------------------------|--------------------------------------------------------|----------------------------------------------------------|
| **Readability**             | Flat, large sheets                                    | Nested, organized structure                             |
| **Type Safety**             | All values are strings                                | Supports booleans, integers, and nulls natively         |
| **Version Control Friendly**| ❌ Hard to diff in Git                                 | ✅ Easy to version & review changes                      |
| **UI Editing**              | ✅ Easy for non-technical users                        | ❌ Requires text editing                                |
| **Programmatic Access**     | Requires lookups across sheets                        | Direct dictionary access (e.g., `config("Log")`)        |
| **Comments Support**        | Cell notes only                                       | ✅ Inline with `/* comment */` in value line            |
| **Validation**              | Manual, prone to typo                                | Possible to validate schema with JSON tools             |
| **Flexibility**             | Limited to 3-column pattern                           | Fully customizable nesting & naming                     |

---

## ✅ Benefits of Using JSON

- Clean hierarchical structure
- Inline documentation (`/* comment */`)
- Supports multiple data types
- Easier Git diffs and merge resolution
- Better integration with CI/CD pipelines

---

## 🛠 Loading the JSON in UiPath

ConfigJson variable is loaded with `InitAllSettingsJson.xaml`.

1. Uses `Read Text File` to load the JSON as a string.
2. Uses `Deserialize JSON` activity to convert it into a `JObject`.
3. Uses a custom routine or invoke code to flatten `"Assets"` into the root config.

---

## ☁️ Use Local or Asset JSON

Both Local Config.Json or Asset can be used.
- To use local Config from `Data\Config.Json`, let in_ConfigJsonAssetName empty in InitAllSettingsJson
- To use Asset Config from Orchestrator folder in_OrchestratorQueueFolder, use `in_ConfigJsonAssetName`
