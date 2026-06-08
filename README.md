# Contribution [1]: enhance observability with structured logging and metrics

**Contribution Number:** 1  
**Student:** Minh Le  
**Issue:** https://github.com/microsoft/qlib/issues/2098    
**Status:** Phase I (In Progress)

---

## Why I Chose This Issue

I chose this issue because it connects directly to my interests in Python, data infrastructure, and AI/ML tooling. Qlib is a quantitative research and machine learning platform, so contributing to its logging or observability system would help me better understand how large open-source data/AI projects are maintained in practice. The issue is also meaningful because better observability can help users identify bottlenecks, debug workflows, and compare performance across data processing, model training, and backtesting tasks.

I also chose this issue because the proposed first phase, structured logging in qlib/log.py, seems like a possible entry point for a first open-source contribution if the scope is narrowed carefully. I do not plan to take on the full issue at once because the complete proposal includes structured logging, metrics, tracing, configuration changes, and updates across several modules. My initial goal is to ask for mentor and maintainer feedback on whether a smaller contribution focused only on opt-in structured logging would be acceptable.

---

## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

Qlib currently has only basic observability support. It uses standard Python logging through `get_module_logger`, some manual timing through `TimeInspector`, and experiment metrics through `R.log_metrics()`, but these pieces are scattered and limited. There is no unified way to produce structured logs, collect system or workflow performance metrics, or trace execution across the major stages of a quant research pipeline. This makes it harder for users and contributors to debug slow workflows, understand bottlenecks, monitor model training behavior, inspect backtest execution costs, or observe online serving and rolling update processes.

### Expected Behavior

[What should happen?]

Qlib should provide an opt-in observability layer that helps users monitor and debug workflows without changing existing default behavior. At a minimum, users should be able to enable structured logging and get more useful machine-readable logs. Over time, Qlib should also support optional performance metrics collection and workflow tracing for important operations such as dataset loading, model training, backtesting, and online updates. The feature should be backward compatible, configurable through Qlib’s existing initialization/config flow, and impose little to no overhead when disabled.

### Current Behavior

[What actually happens?]

Right now, Qlib mostly logs plain text messages and occasionally records elapsed time with `TimeInspector`. Some training-related metrics are logged to the experiment recorder through `R.log_metrics()`, but this is focused on experiment results rather than system observability. There is no built-in support for structured log output, no centralized metrics collector, no exposed cache hit/miss monitoring, and no trace or span model for following execution across components. In practice, observability exists only in isolated pieces, so users have to inspect logs manually or add custom instrumentation themselves.

### Affected Components

[Which parts of the codebase are involved?]

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

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

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
