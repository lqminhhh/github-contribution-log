# Contribution [1]: enhance observability with structured logging and metrics

**Contribution Number:** 1  
**Student:** Minh Le  
**Issue:** https://github.com/microsoft/qlib/issues/2098    
**Status:** Phase III (In Progress)

---

## Why I Chose This Issue

I chose this issue because it connects directly to my interests in Python, data infrastructure, and AI/ML tooling. Qlib is a quantitative research and machine learning platform, so contributing to its logging or observability system would help me better understand how large open-source data/AI projects are maintained in practice. The issue is also meaningful because better observability can help users identify bottlenecks, debug workflows, and compare performance across data processing, model training, and backtesting tasks.

I also chose this issue because the proposed first phase, structured logging in qlib/log.py, seems like a possible entry point for a first open-source contribution if the scope is narrowed carefully. I do not plan to take on the full issue at once because the complete proposal includes structured logging, metrics, tracing, configuration changes, and updates across several modules. My initial goal is to ask for mentor and maintainer feedback on whether a smaller contribution focused only on opt-in structured logging would be acceptable.

---

## Understanding the Issue

### Problem Description

Qlib currently has only basic observability support. It uses standard Python logging through `get_module_logger`, some manual timing through `TimeInspector`, and experiment metrics through `R.log_metrics()`, but these pieces are scattered and limited. There is no unified way to produce structured logs, collect system or workflow performance metrics, or trace execution across the major stages of a quant research pipeline. This makes it harder for users and contributors to debug slow workflows, understand bottlenecks, monitor model training behavior, inspect backtest execution costs, or observe online serving and rolling update processes.

### Expected Behavior

Qlib should provide an opt-in observability layer that helps users monitor and debug workflows without changing existing default behavior. At a minimum, users should be able to enable structured logging and get more useful machine-readable logs. Over time, Qlib should also support optional performance metrics collection and workflow tracing for important operations such as dataset loading, model training, backtesting, and online updates. The feature should be backward compatible, configurable through Qlib’s existing initialization/config flow, and impose little to no overhead when disabled.

### Current Behavior

Right now, Qlib mostly logs plain text messages and occasionally records elapsed time with `TimeInspector`. Some training-related metrics are logged to the experiment recorder through `R.log_metrics()`, but this is focused on experiment results rather than system observability. There is no built-in support for structured log output, no centralized metrics collector, no exposed cache hit/miss monitoring, and no trace or span model for following execution across components. In practice, observability exists only in isolated pieces, so users have to inspect logs manually or add custom instrumentation themselves.

### Affected Components

- `qlib/qlib/log.py`: Core logging utilities, including `get_module_logger` and `TimeInspector`.
- `qlib/qlib/config.py`: Global configuration handling, including `logging_config`and `qlib.init()` setup.
- `qlib/qlib/model/trainer.py`: Training workflow entry points where model execution and task lifecycle metrics could be added.
- `qlib/qlib/workflow/recorder.py`: Existing experiment metric logging that may be reused or extended for observability-related reporting.
- `qlib/qlib/data/data.py` and `qlib/qlib/data/cache.py`: Data loading and cache behavior, which are important instrumentation points for performance and cache visibility.
- `qlib/qlib/backtest/exchange.py` and `qlib/qlib/backtest/executor.py`: Backtest execution flow, where timing and decision/execution observability would be useful.
- `qlib/qlib/workflow/online/` and `qlib/qlib/contrib/rolling/`: Online serving and rolling workflow code, where monitoring and tracing would help operational visibility.

---

## Reproduction Process

### Environment Setup

I used the local `qlib/` repository checked out inside this workspace and reviewed the current code on the `main` branch. Since this issue is a feature gap rather than a crash or failing behavior, my "reproduction" process focused on confirming that the requested observability features do not currently exist in the codebase.

My setup work in Week 2 was mainly code reading and repo inspection:

- Reviewed `qlib/README.md` and `qlib/docs/index.rst` to understand the project structure and major subsystems.
- Inspected `qlib/qlib/log.py`, `qlib/qlib/config.py`, `qlib/qlib/cli/run.py`, `qlib/qlib/model/trainer.py`, and `qlib/qlib/workflow/recorder.py`.
- Searched the repository for `get_module_logger`, `TimeInspector`, `R.log_metrics`, `structured`, `observability`, `tracing`, and related terms to verify the current implementation status.

I did not need to fully run a Qlib workflow yet to confirm the issue, because the missing functionality is visible directly in the source code and configuration layer.

### Steps to Reproduce

1. Open `qlib/qlib/log.py` and inspect the logging utilities used throughout Qlib.
2. Confirm that `get_module_logger` returns a wrapper around standard Python logging and that `TimeInspector` only provides manual timing logs.
3. Open `qlib/qlib/config.py` and inspect the default `logging_config` and `qlib.init()` configuration handling.
4. Search the repository for observability-related terms such as `structured`, `observability`, `trace`, `MetricsCollector`, `prometheus`, and `OpenTelemetry`.
5. Inspect workflow- and training-related modules such as `qlib/qlib/model/trainer.py` and `qlib/qlib/workflow/recorder.py` to see how metrics are currently recorded.
6. Observe that:
   - __No built-in structured logging API__
   - __No centralized metrics collection abstraction__
   - __No trace/span workflow instrumentation__
   - __Existing metrics logging is limited to experiment tracking via `R.log_metrics()`__

### Reproduction Evidence

- **Commit showing reproduction:** Not applicable yet. This week focused on verifying the current repository state and narrowing scope before implementation.
- **Screenshots/logs:** Not applicable. The evidence for this issue is primarily in the current source code and configuration surface.
- **My findings:** The issue is still relevant in the current `qlib/main` branch. Qlib has plain-text logging, some manual timing through `TimeInspector`, and experiment metrics through `R.log_metrics()`, but it does not yet have an opt-in structured logging mode, a unified observability layer, or workflow tracing support.

---

## Solution Approach

### Analysis

The root cause is not a single bug in one file. The larger issue is that Qlib's observability support grew in a piecemeal way. Logging, timing, and metrics exist in different places, but they are not designed as one coherent system.

From my review of the current code:

- `qlib/qlib/log.py` provides a logger wrapper and `TimeInspector`, but there is no structured formatter or structured logger interface.
- `qlib/qlib/config.py` supports normal Python `logging_config`, but there is no first-class observability configuration model for structured logging, metrics, or tracing.
- `qlib/qlib/model/trainer.py` and other workflow modules do useful work, but they do not emit standardized performance or lifecycle metrics in a reusable way.
- `qlib/qlib/workflow/recorder.py` supports experiment metrics, which is useful, but that is different from system-level observability.

Because the issue touches __multiple subsystems__, trying to solve everything at once would create too much scope for a first contribution. The practical root cause for the first phase is that Qlib lacks an extensible, opt-in logging foundation that later metrics and tracing work could build on.

### Proposed Solution

My proposed approach is to break the issue into smaller, reviewable phases and start with the lowest-risk, highest-value slice: opt-in structured logging.

First of all, I plan to:

- extend the logging utilities in `qlib/qlib/log.py` so Qlib can emit structured log output when explicitly enabled,
- add configuration support through `qlib.init()` / `logging_config` in `qlib/qlib/config.py`,
- preserve the current default logging behavior for all existing users,
- add tests and usage examples so the new behavior is documented and safe to review.

### Implementation Plan

UMPIRE framework:

- **Understand:** Qlib currently has basic logging and some timing/experiment metrics, but it does not have an opt-in structured logging mode or a unified observability foundation.

- **Match:** Qlib already centralizes logging through `get_module_logger`, applies global logging configuration through `qlib.init()`, and records some experiment metrics through `R.log_metrics()`. These existing hooks mean a structured logging feature can likely be added without rewriting the whole logging surface.

- **Plan:**  
    1. Review `qlib/qlib/log.py` and decide the least invasive way to support structured logging while preserving the current logger wrapper design.  
    2. Update `qlib/qlib/config.py` so structured logging can be enabled through configuration without changing defaults.  
    3. Confirm that CLI-driven workflows in `qlib/qlib/cli/run.py` still initialize logging correctly under both default and structured modes.  
    4. Add focused tests for backward compatibility and structured output behavior.  
    5. Document example usage for enabling structured logging.  

- **Implement:** I have not started coding yet. My next implementation step is to narrow the first PR to structured logging only and then create a working branch for that slice.

- **Review:**  
    - Keep the feature opt-in.  
    - Do not change existing logging behavior by default.  
    - Keep the PR focused on a small number of files.  
    - Follow existing Qlib patterns instead of introducing a separate logging framework too early.  
    - Add tests for any public-facing config or behavior change.  

- **Evaluate:** I will verify the first PR by checking that:
    - existing logger behavior remains unchanged when structured logging is disabled,
    - structured logging can be enabled through config,
    - structured fields are emitted in the expected format,
    - and representative workflow entry points still initialize and use logging correctly.

---

## Testing Strategy

### Unit Tests

- [x] Default logger behavior remains unchanged when the JSON formatter is not selected.
- [x] The opt-in JSON formatter produces machine-readable output with timestamp, level, logger name, and message fields.
- [x] Logger calls that include additional fields through `extra` preserve those fields in JSON output.

### Integration Tests

- [ ] Initialize Qlib through `qlib.init()` with default logging config and confirm existing workflows still start normally.
- [ ] Initialize Qlib with structured logging enabled and confirm the CLI/workflow entry path still uses the configured logger behavior.

### Manual Testing

I ran `python -m pytest tests/test_log.py -q` in a Python 3.12 development environment. All three focused tests passed. Configuration-level and workflow integration tests remain as the next implementation increment.

---

## Implementation Notes

### Week 2 Progress

This week I focused on understanding the project, validating that the issue is still relevant, and narrowing the scope of a realistic first contribution.

Completed this week:

- [x] reviewed the Qlib repository structure and identified the major subsystems relevant to the issue,
- [x] read the current logging, config, workflow, and trainer entry points,
- [x] confirmed that the observability issue is still relevant on the current `main` branch,
- [x] wrote a scoped contribution plan,
- [x] and narrowed the likely first PR to opt-in structured logging instead of the full observability roadmap.

Main challenge:

- The issue as written is very broad and spans logging, metrics, tracing, data workflows, training, backtesting, and online serving. The main decision this week was to avoid treating it as one large implementation task and instead define a small first slice that could realistically be reviewed and merged.

### Week 3 Progress

This week I implemented the first focused slice of issue #2098:

- added an opt-in `JSONFormatter` to `qlib/log.py`,
- included standard timestamp, severity, logger name, and message fields in each JSON record,
- preserved fields supplied through Python logging's `extra` argument,
- kept Qlib's existing plain-text behavior unchanged by default,
- added three focused unit tests in `tests/test_log.py`,
- and verified that all three tests pass under Python 3.12.

The main environment challenge was that the original Conda base environment used unsupported Python 3.13 and was missing `setuptools_scm`. I resolved this by using a Python 3.12 Qlib development environment before running the tests.

### Code Changes

- **Branch:** [`fix-issue-2098`](https://github.com/lqminhhh/qlib/tree/fix-issue-2098)
- **Files modified:** `qlib/log.py`, `qlib/config.py`, `qlib/metrics.py`, `tests/test_log.py`, `tests/test_metrics.py`
- **Key commit:** [`cb76ffc8` — feat: add structured JSON log formatter](https://github.com/lqminhhh/qlib/commit/cb76ffc8)
- **Follow-up commit:** `2fe1aa08` — add compact structured logging config support and internal config normalization
- **Approach decisions:** I used standard Python logging configuration for structured output and a small in-memory metrics recorder for the metrics foundation, keeping both features opt-in and dependency-free by default. The next increment is adding targeted instrumentation points in Qlib's data/cache or workflow code.

### Week 4 Progress

This week I continued expanding the implementation toward the full issue scope instead of submitting a structured-logging-only PR.

Completed this increment:

- added compact structured logging config support through `qlib.init(logging_config={"structured": True, "format": "json"})`,
- kept normal Python `dictConfig` logging support compatible,
- added JSON exception serialization coverage,
- added a config-level test proving structured logging can be enabled through Qlib logging config,
- updated the implementation plan to continue toward metrics and tracing before opening the upstream PR.

Current implementation status:

- Structured logging foundation is implemented.
- Config-level structured logging enablement is implemented.
- Metrics collection is the next active implementation area.
- Workflow tracing remains pending and may be deferred if the PR becomes too large.

Next planned increment:

- Add a lightweight, dependency-free metrics recorder that is disabled by default.
- Support counters, gauges, and timers.
- Add tests proving disabled-mode safety and enabled in-memory metric collection.

Metrics foundation progress:

- added `qlib.metrics` with a disabled-by-default no-op recorder,
- added an in-memory recorder for enabled metrics,
- supported counters, gauges, timings, timer context manager usage, and optional tags,
- added `metrics_config={"enabled": True}` support through Qlib config,
- added focused unit tests in `tests/test_metrics.py`,
- verified the structured logging and metrics tests together:

```bash
conda run -n qlib-dev python -m pytest tests/test_log.py tests/test_metrics.py -q
```

Result:

```text
10 passed
```

First instrumentation progress:

- added cache metrics to `SimpleDatasetCache`:
  - `qlib.cache.dataset.hit`
  - `qlib.cache.dataset.miss`
  - `qlib.cache.dataset.load_seconds`
- added cache metrics to `MemoryCalendarCache`:
  - `qlib.cache.calendar.hit`
  - `qlib.cache.calendar.miss`
  - `qlib.cache.calendar.load_seconds`
- added metric tags for cache type and frequency, for example `cache=simple,freq=day`,
- added unit tests with fake providers so cache instrumentation can be verified without downloading Qlib data,
- re-ran the focused logging + metrics test suite:

```bash
conda run -n qlib-dev python -m pytest tests/test_log.py tests/test_metrics.py -q
```

Result:

```text
12 passed
```

Metrics export/reporting progress:

- added `metrics.export()` as an explicit way to return collected metrics as a plain dictionary,
- added `metrics.summary()` for a human-readable summary of counters, gauges, and timing statistics,
- added `metrics.log_summary()` for optional logger-based reporting,
- kept reporting optional and dependency-free,
- added tests for export output, summary formatting, and logger-based reporting,
- re-ran the focused logging + metrics test suite:

```bash
conda run -n qlib-dev python -m pytest tests/test_log.py tests/test_metrics.py -q
```

Result:

```text
15 passed
```

Basic tracing progress:

- added lightweight `qlib.trace` support with no required external tracing dependency,
- added `trace_span(...)` context manager for creating spans when tracing is enabled,
- added nested span support with shared `trace_id` and `parent_span_id`,
- added `tracing_config={"enabled": True}` support through Qlib config,
- updated structured JSON logging so active spans add `trace_id`, `span_id`, and `span_name`,
- verified disabled tracing has no effect on structured logs,
- added focused tracing tests and JSON logging trace-context tests,
- re-ran the focused logging + metrics + tracing test suite:

```bash
conda run -n qlib-dev python -m pytest tests/test_log.py tests/test_metrics.py tests/test_trace.py -q
```

Result:

```text
21 passed
```

Real tracing instrumentation progress:

- wrapped `SimpleDatasetCache` dataset loading with a `cache.dataset` trace span,
- wrapped `MemoryCalendarCache` calendar loading with a `cache.calendar` trace span,
- updated fake-provider tests to confirm real provider calls execute inside active trace context,
- verified cache spans expose `trace_id`, `span_id`, and `span_name`,
- re-ran the focused logging + metrics + tracing test suite:

```bash
conda run -n qlib-dev python -m pytest tests/test_log.py tests/test_metrics.py tests/test_trace.py -q
```

Result:

```text
23 passed
```

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
