# Qlib Issue #2098 Full PR Completion Plan

Goal: finish a review-ready implementation for Qlib issue #2098 before opening the upstream pull request.

Working principle: keep observability opt-in, lightweight, dependency-free by default, and backwards compatible. The PR should improve Qlib observability without changing existing logging output unless users explicitly enable the new behavior.

## Current Status

Completed so far:

- [x] Added `JSONFormatter` in `qlib/qlib/log.py`.
- [x] JSON logs include `timestamp`, `level`, `logger`, and `message`.
- [x] JSON logs preserve custom `extra={...}` fields.
- [x] Existing default plain-text logging behavior remains unchanged.
- [x] Added tests for plain-text logging regression.
- [x] Added tests for JSON formatter output.
- [x] Added tests for `extra` fields in JSON logs.
- [x] Verified `tests/test_log.py` passes in the Python 3.12 dev environment.

Still missing for the full issue:

- [x] Config-level structured logging enablement.
- [x] `qlib.init(...)` integration for structured logging.
- [x] Metrics collection abstraction.
- [ ] At least a few high-value metrics instrumentation points.
- [ ] Optional metrics export/reporting path.
- [ ] Workflow or operation tracing, if kept in scope.
- [ ] Documentation/examples showing how users enable the observability features.
- [ ] Broader tests beyond the standalone formatter tests.

## 1. Reconfirm Scope Before More Coding

- [ ] Re-read issue #2098 and write the final PR scope in one paragraph.
- [ ] Decide the minimum full PR scope:
      structured logging + lightweight metrics + basic tracing hooks.
- [ ] Keep the following as non-goals unless maintainers explicitly request them:
      Prometheus dependency, OpenTelemetry dependency, large refactors, default logging changes.
- [ ] Confirm no new upstream PR already solves the same issue.
- [ ] Keep `.DS_Store` / unrelated `.gitignore` changes out of the final Qlib PR.

## 2. Structured Logging Finish-Line

Already started. Finish this first because it is the safest foundation.

- [x] Add reusable `JSONFormatter`.
- [x] Preserve default logging behavior.
- [x] Preserve `extra` fields.
- [x] Add a config-level test showing JSON logging can be enabled through logging config.
- [x] Decide final public API for enabling structured logging.

Preferred implementation path:

- [x] Support a config shape like:

```python
qlib.init(logging_config={
    "structured": True,
    "format": "json",
})
```

or, if this fits Qlib better:

```python
qlib.init(logging_config={
    "version": 1,
    "formatters": {
        "json": {
            "()": "qlib.log.JSONFormatter",
        }
    },
    ...
})
```

- [x] Keep `get_module_logger(...)` backwards compatible.
- [x] Only add `structured=True` to `get_module_logger(...)` if it can be done cleanly without surprising existing call sites.
- [ ] Add docs/example showing structured logs.

Acceptance criteria:

- [x] Default logs still look the same.
- [x] Users can opt into JSON logs.
- [x] Tests prove both default and structured modes work.

## 3. Metrics Foundation

Build a small internal metrics layer before instrumenting many files.

Implementation target:

- [x] Add a lightweight metrics module, likely `qlib/qlib/metrics.py` or `qlib/qlib/utils/metrics.py`.
- [x] Implement no-op behavior when metrics are disabled.
- [x] Support basic metric types:
      counters, gauges, and timers.
- [x] Avoid required third-party dependencies.
- [x] Add a small in-memory registry for tests and simple console/export use.
- [x] Add config enablement through `qlib.init(...)` or existing config patterns.

Possible API:

```python
from qlib.metrics import get_metrics_recorder

metrics = get_metrics_recorder()
metrics.increment("qlib.cache.hit")
metrics.timing("qlib.data.load_seconds", elapsed)
metrics.gauge("qlib.memory.usage_mb", value)
```

Acceptance criteria:

- [x] Metrics are disabled by default.
- [x] Calling metrics methods while disabled is safe and cheap.
- [x] Enabled metrics are recorded in a testable way.
- [x] No external service is required.

## 4. First Metrics Instrumentation Points

Start narrow. A huge instrumentation sweep is risky.

Recommended first targets:

- [ ] Data/cache layer:
      cache hit count, cache miss count, load duration.
- [ ] Dataset or handler loading:
      dataset prepare/load duration.
- [ ] Model training:
      fit/train duration.

Optional if clean:

- [ ] Backtest execution duration.
- [ ] Recorder/workflow metric logging through `R.log_metrics()` if it fits naturally.

Acceptance criteria:

- [ ] At least 2-3 meaningful metrics are emitted when metrics are enabled.
- [ ] No measurable behavior change when metrics are disabled.
- [ ] Tests verify metric names and values without relying on flaky timing.

## 5. Metrics Export / Reporting

Keep export simple for the first full PR.

Minimum useful path:

- [ ] Provide a way to inspect/export collected metrics from the in-memory recorder.
- [ ] Add optional console summary or logger-based summary.
- [ ] Document metric names and example output.

Defer unless maintainers request it:

- [ ] Prometheus endpoint/exporter.
- [ ] OpenTelemetry metrics export.
- [ ] Any required cloud/external monitoring integration.

Acceptance criteria:

- [ ] Users can see collected metrics without extra infrastructure.
- [ ] Export path is optional.
- [ ] Tests cover export/report output.

## 6. Basic Workflow Tracing Hooks

Do this only after structured logging and metrics are stable.

Recommended lightweight scope:

- [ ] Add trace/span IDs to structured logs when tracing is enabled.
- [ ] Provide a context manager for spans, for example:

```python
with trace_span("dataset.prepare"):
    ...
```

- [ ] Use spans around a small number of stable workflow boundaries:
      dataset prepare, model fit, backtest run, recorder operations.
- [ ] Keep tracing disabled by default.
- [ ] Avoid mandatory OpenTelemetry dependency.

Acceptance criteria:

- [ ] Trace IDs can connect related structured logs.
- [ ] Disabled tracing has no effect on existing workflows.
- [ ] Tests cover span creation and disabled-mode behavior.

## 7. Tests to Add Before PR

Structured logging:

- [x] JSON formatter output.
- [x] Extra fields.
- [x] Default plain-text regression.
- [x] Config-enabled JSON logging.
- [x] Exception formatting in JSON logs.

Metrics:

- [x] No-op recorder behavior when disabled.
- [x] Counter increment behavior.
- [x] Gauge behavior.
- [x] Timer context behavior.
- [ ] Timer behavior with stable mocked timing.
- [ ] Export/report behavior.
- [ ] One integration test for an instrumented Qlib component.

Tracing:

- [ ] Trace/span context creation.
- [ ] Trace IDs appear in structured logs when enabled.
- [ ] No trace fields appear when disabled unless explicitly requested.

Regression:

- [x] Run `python -m pytest tests/test_log.py -q`.
- [ ] Run targeted tests for any touched data/model/workflow files.
- [ ] Run `git diff --check`.

## 8. Documentation and PR Evidence

- [ ] Add a short documentation section or example showing:
      how to enable JSON logging, how to enable metrics, and how to read the output.
- [ ] Include sample JSON log output in the PR description.
- [ ] Include sample metrics output in the PR description.
- [ ] Document what is intentionally deferred.
- [ ] Update `github-contribution-log/README.md` with:
      implementation progress, tests run, final PR link, and review status.

## 9. Final Pre-PR Checklist

- [ ] `git status` is clean except intended files.
- [ ] No `.DS_Store`, local environment, cache, or unrelated formatting changes.
- [ ] Branch is up to date with upstream `main`.
- [ ] Tests pass in Python 3.12.
- [ ] Diff has been reviewed line by line.
- [ ] PR description explains:
      what changed, why it was needed, how to enable it, tests run, and follow-up work.

## 10. Suggested Build Order From Here

1. Finish structured logging config integration.
2. Add tests for config-enabled JSON logging and exception formatting.
3. Add lightweight metrics recorder module.
4. Add tests for metrics recorder.
5. Instrument 2-3 stable components.
6. Add tests for at least one instrumentation point.
7. Add simple metrics export/reporting.
8. Add lightweight trace/span context only if time remains.
9. Update docs and contribution README.
10. Run final checks and open the upstream PR.

## Definition of Done for Full PR

- [ ] Structured logging is opt-in, tested, and documented.
- [ ] Metrics are opt-in, tested, and documented.
- [ ] At least 2-3 meaningful Qlib operations emit metrics.
- [ ] Basic trace/span support exists, or tracing is explicitly documented as deferred.
- [ ] Existing default behavior remains unchanged.
- [ ] No mandatory new external observability dependency is introduced.
- [ ] The PR is review-ready and honestly describes completed scope plus deferred follow-ups.
