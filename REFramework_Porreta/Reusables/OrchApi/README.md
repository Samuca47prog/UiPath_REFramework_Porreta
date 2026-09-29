# Orchestrator API helpers

> Status: **in development.** `OrchApiHttps_GetThisJob.xaml` and `OrchApiHttps_RestartJob.xaml` are in `ignoredFiles` (excluded from publish).

## Purpose
Wrappers for Orchestrator API calls needed by framework features that Orchestrator activities don't cover (e.g. `Framework/Porreta/ShouldReleaseThisMachine.xaml`, which checks whether a higher-priority job is waiting for this machine).

## Workflows
| Workflow | Role | Status |
|---|---|---|
| [OrchApiHttps_GetJobs.xaml](OrchApiHttps_GetJobs.xaml) | `GET /odata/Jobs` with optional `$filter` / `$select` → `value` array | In use by ShouldReleaseThisMachine |
| [OrchApiHttps_GetThisJob.xaml](OrchApiHttps_GetThisJob.xaml) | Current job info + `GET /odata/jobs(...)` | In development |
| [OrchApiHttps_RestartJob.xaml](OrchApiHttps_RestartJob.xaml) | `POST /odata/Jobs/UiPath.Server.Configuration.OData.RestartJob` | In development |
| [OrchApiHttps_GetToken.xaml](OrchApiHttps_GetToken.xaml) | Client-credentials token for an External Application | Working |

## Request pattern
Every helper follows the [HTTP snippet](../../Code_Snippets/README.md):
build URL → request inside a Retry Scope → `Switch` on the status code (log mapped codes, throw on unmapped ones) → throw on the 4xx/5xx series → deserialize the response.

## Authentication
- **Orchestrator HTTP Request** (GetJobs, GetThisJob, RestartJob) authenticates with the robot's own context, so no token is needed. The folder is passed as an argument.
- **GetToken** is for calls outside that context. It reads an External Application's client id / secret from a credential asset and posts to the identity endpoint. The default `in_Url` points to **staging** — override it for other environments.

## Limitations / roadmap
- The 4xx/5xx check uses `responseStatus Mod 100`, but it needs integer division (`\ 100`). As written, 400/401/403/500 are not caught, while 204/304 are.
- GetThisJob: `out_JobJson` is never assigned. It also calls `/odata/jobs(<Key>)` with the job GUID, where the URL expects the numeric Id (TODO in the annotation).
- RestartJob: `StatusCode` and `Result` are not bound, so it always hits the "not mapped" throw.
- ShouldReleaseThisMachine: the "create this job again" step is empty, pending jobs usually have no `HostMachineName`, and the meaning of the priority values still needs to be confirmed.
