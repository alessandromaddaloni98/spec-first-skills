# Add retry with backoff to the weather API client

**Status**: implemented (2026-03-12)
**Prerequisites**: none
**Related specs**: `spec/cache-forecasts.md` (downstream: the cache assumes failed calls return `None`)

> Implement this spec with the `spec-do` skill, which updates it at the end
> of the work (Outcome, Status, new entries in Decisions).

## 1. Context

`WeatherClient.fetch_forecast()` fails the whole dashboard refresh on the first
timeout. The upstream API returns sporadic 503s at peak hours; one retry would
recover almost all of them.

Small, isolated change: one function, one config key, two tests. The difficulty
is not the retry itself but keeping the current contract: on final failure the
method must still return `None`, not raise, because the cache relies on it.

Out of scope: retries for the geocoding client, any change to the cache.

## 2. File map

| File | Role in this change |
|---|---|
| `app/clients/weather.py` | `fetch_forecast()` gets the retry loop |
| `config/settings.yaml` | new key `weather.max_retries` |
| `tests/clients/test_weather.py` | two new tests |
| `docs/configuration.md` | to update: document `weather.max_retries` |

## 3. Premises to verify (Step 0)

- `fetch_forecast()` is the only caller of `requests.get` in `app/clients/weather.py`:
  `grep -n "requests.get" app/clients/weather.py` → exactly one match.
- The cache treats `None` as "no data": `grep -n "is None" app/cache/forecasts.py`.

If either no longer holds, stop and report.

## 4. Decisions

| ID | Date | Decision | Why |
|---|---|---|---|
| D1 | 2026-03-10 | Retry only on timeouts and HTTP 429/502/503/504 | Other errors (401, 404) won't fix themselves |
| D2 | 2026-03-10 | Exponential backoff: 1s, 2s, 4s; default `max_retries: 2` | Keeps the worst-case refresh under 10s |
| D3 | 2026-03-11 | Log each retry at WARNING with attempt number and status code | Added during implementation: needed to see retries in production logs |

Discarded: using the `tenacity` library (adds a dependency for 15 lines of code).

## 5. Execution

1. In `fetch_forecast()`, wrap the request in a loop of `max_retries + 1` attempts,
   applying D1 and D2. Read `max_retries` from settings, default 2.
2. On final failure keep the current behaviour: log the error and return `None`.
3. Add `weather.max_retries: 2` to `config/settings.yaml`.
4. Tests: one where the first call returns 503 and the second succeeds; one where
   every call returns 503 and the result is `None`. Mock `time.sleep`.

## 6. Pitfalls

- Tests that don't mock `time.sleep` will pass but take seconds: slow tests are
  a symptom, not a success.
- The geocoding client has the same flaw. Don't "fix it for symmetry": out of scope.

## 7. Acceptance criteria

- [ ] A single 503 followed by a 200 returns the forecast.
- [ ] Persistent 503s return `None` after `max_retries + 1` attempts, without raising.
- [ ] A 401 is not retried.
- [ ] Test suite green, no new failures.

## 8. Do NOT

- Don't make `fetch_forecast()` raise on final failure (breaks the cache contract).
- Don't add new dependencies.

## 10. Final report

Outcome per acceptance criterion, test results, and any change to the method's
signature or return values.

## 11. Outcome

2026-03-12 — All steps done, all four criteria met. D3 added during implementation.
Suite: 148 passed, 0 new failures. `docs/configuration.md` updated with the new key.
No change to the method signature.

## Working rules
- If a choice is ambiguous and changes the system's behaviour: stop and ask.
- No improvements, simplifications or style fixes outside the scope.
- Don't delete data files without explicit confirmation.
- Don't read `.env` files, don't make calls with real credentials, never print secrets in console or logs.
- Distinguish new test failures from pre-existing ones; don't fix unrelated ones.
- Code, names, docstrings, comments and logs follow the repo's conventions.
