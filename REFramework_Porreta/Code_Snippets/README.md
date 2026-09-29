# Code snippets

Snippets are starting points you copy into a new workflow. They are not invoked as-is.

## API/API_HTTPS.xaml — HTTP request
### Why a snippet and not a workflow
- HTTP requests have too many parameters (method, headers, body, auth…) to wrap well in one workflow.
- A project often needs several different requests (GET, POST, PUT…), and a single generic workflow would get in the way.

### How to use
1. Create a new workflow for the request (e.g. `Reusables/<Area>/<Area>Https_<Action>.xaml`).
2. Paste the snippet, set the URL, method, headers and body.
3. Update the `Switch` status-code cases for your scenario (the annotation reminds you).
4. Map the output: `Deserialize JSON` → out arguments.

Example implementation: [Reusables/OrchApi/OrchApiHttps_GetToken.xaml](../Reusables/OrchApi/OrchApiHttps_GetToken.xaml).

### Known issues
- The 4xx/5xx check uses `Mod 100` where it needs `\ 100` (see [OrchApi README](../Reusables/OrchApi/README.md)). Every copy has the same bug.
- For an unmapped status code, the snippet throws a `BusinessRuleException`, while the OrchApi copies throw a `SystemException`. Decide which one is the default.

## ModifyConfigJsonForThisTestCase.xaml — config for a single test
Loads the local config through `InitAllSettingsJson.xaml`, changes the keys this test needs, and uploads the result to the testing asset (`ConfigJson_Testing`) with `Set Asset`. Main then reads that asset through `in_ConfigJsonAssetName`, so neither the local JSON nor the `ConfigJson` asset is changed.

Side effects to keep in mind:
- The uploaded JSON has its assets already resolved, has no `Assets` key and has no comments. This is why `InitAllSettingsJson` skips assets when the key is missing.
- The asset is shared, so tests run one at a time (on purpose, see [Tests/README.md](../Tests/README.md)).
