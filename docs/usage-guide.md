# TrainCheck Usage Guide

TrainCheck monitors deep‑learning training runs for “silent” correctness issues. This document explains common ways to apply TrainCheck and clarifies its current limitations.

## Typical Scenarios

- **Sanity checking new pipelines.** When bringing up a new model or porting code between frameworks, use TrainCheck to confirm that basic API interactions follow expected patterns.
- **Regression testing.** After modifying training code or upgrading dependencies, run TrainCheck on a short reference run to ensure key behaviors (gradient updates, optimizer steps, etc.) remain intact.
- **Monitoring long experiments.** Use the semi‑online checker to spot issues early in multi‑hour or multi‑day jobs. TrainCheck can alert you soon after a silent failure occurs, instead of waiting for final metrics to degrade.
- **Debugging unusual behavior.** If training diverges or performs suspiciously, collect a trace and compare it against invariants inferred from a known-good run to narrow down where behavior deviates.

## What TrainCheck Does Well

- Detects **missing or reordered API calls** that break common training assumptions.
- Verifies **parameter updates** and other state changes that should occur inside optimizer steps.
- Works with standard PyTorch scripts without requiring code modifications.

## Current Limitations

- **Framework support.** TrainCheck primarily targets PyTorch. Other frameworks or highly customized backends may require additional instrumentation work.
- **Online monitoring.** Real-time checking is in development. Today you need to periodically run the checker on the growing trace output.
- **Semantic bugs.** TrainCheck focuses on low-level behavioral invariants. Issues that only manifest in final accuracy or require understanding of model semantics may not be caught.
- **Large-scale traces.** Very long or distributed runs can generate huge traces, which may slow inference or checking. Short representative runs usually suffice for inference.

## Potential Pitfalls

- **Overfitting invariants.** Invariants inferred from a single narrow run may fire on benign differences. Consider collecting traces from several healthy runs to generalize better.
- **Instrumentation overhead.** Trace collection adds runtime overhead. Start with short exploratory runs to verify that the instrumentation does not alter training dynamics.
- **Unsupported APIs.** Rarely used or third-party APIs may not be instrumented by default, leading to missed events.

## Choosing Reference Runs

Good invariants come from good reference runs. Start with short, known‑good scripts—official examples or your own healthy training jobs. Collect traces from a few such runs to capture normal variation and avoid overfitting. If your workflow changes significantly (e.g., new optimizer or distributed backend), regenerate invariants from a fresh reference run.

## Diagnosing Violations

When TrainCheck reports a violation, inspect the accompanying log snippet and trace location. Compare the failing run against your reference trace to understand what deviated. Often the problematic API call or missing state update will pinpoint the root cause. The [technical documentation](./technical-doc.md) explains how to examine traces in more detail.

## Summary

TrainCheck shines when used as an additional safety net during development and testing of PyTorch training pipelines. It can reveal silent issues early, but it does not replace traditional testing or model evaluation. Keep its limitations in mind, and feel free to suggest improvements or contribute support for new frameworks!
