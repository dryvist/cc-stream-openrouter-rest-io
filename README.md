# cc-stream-openrouter-rest-io

Cribl Stream pack that polls the OpenRouter API with scheduled REST Collectors
and stamps each event for Splunk.

## Collectors

| Job | Endpoint | Schedule (UTC) | Time range | Datatype / sourcetype |
| --- | --- | --- | --- | --- |
| `OpenRouter_Activity` | `GET /activity?date=<day>` | `30 1 * * *` | `-1d@d` to `@d`, missed runs resumed | `openrouter:activity` |
| `OpenRouter_Credits` | `GET /credits` | `5 * * * *` | snapshot | `openrouter:credits` |
| `OpenRouter_Keys` | `GET /keys?include_disabled=true` | `10 * * * *` | snapshot | `openrouter:keys` |

All three require an OpenRouter management key, read from a Cribl text secret.
Every collector ships with its schedule disabled.

`/activity` returns one row per UTC day, model and endpoint for the last 30
completed days. The job's relative time range is its checkpoint: each run asks
for the day that ended before it, and `resumeMissed` reruns skipped days.

`/keys` uses offset pagination to fetch all available keys in pages of up to
100. Key creation, disabling and deletion show as differences between
consecutive snapshots.

## Configuration

| Item | Kind | Default |
| --- | --- | --- |
| `openrouter_management_key` | Cribl text secret | none; create it on the worker group |
| `openrouter_secret_name` | pack variable | `openrouter_management_key` |
| `openrouter_index` | pack variable | `openrouter` |
| `openrouter_api_base_url` | pack variable | `https://openrouter.ai/api/v1` |

## Output

The `openrouter_usage` pipeline routes events to index `openrouter` with
sourcetypes `openrouter:activity`, `openrouter:credits`, and `openrouter:keys`.
It also sets `host`, `source`, and `_time` (activity rows use the start of
their UTC day). The route sends to `__group`, so the worker group's routes
decide the destinations.

## Development

```sh
make install
make docker-up
make test
```
