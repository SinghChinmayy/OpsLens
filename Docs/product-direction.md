# OpsLens product direction

Decision: rename Aegis to OpsLens and prioritize AI-assisted infrastructure investigation. This document records the agreed direction; all capabilities below are planned.

## Audience and promise

Developers and operators of small service environments need an understandable answer to what changed, what is affected, what could explain it, and what to check next. OpsLens is an AI infrastructure investigator, implemented as a tool-using agent harness for DevOps diagnosis.

## Investigation flow

A user question or signal selects an environment, service, and time window. The system retrieves bounded evidence, forms hypotheses, performs permitted diagnostics, revises its findings, and produces a cited report. A visible timeline records tool calls and their evidence; it need not expose private model chain-of-thought. Findings distinguish observations, hypotheses, confirmed findings, and unknowns. Confirmation requires explicit supporting checks, not model confidence alone.

## Architecture boundaries

- API and web UI: service overview, investigation creation, timeline, report, and evidence inspection.
- Go worker: durable investigation lifecycle, cancellation, tool/time/token budgets, and bounded retries.
- Evidence adapters: metrics, logs, traces, deployment events, and dependency snapshots. Choose concrete telemetry products during the first integration; do not build all adapters at once.
- PostgreSQL: services, investigations, evidence metadata/snapshots, tool calls, findings, and audit events. Raw telemetry stays in its source systems with explicit snapshot retention.
- Model adapter: one provider initially, structured outputs with evidence IDs and allowed diagnostic requests.
- Policy/tool gateway: validates scope, identity, tool name, arguments, result size, query duration, and budget before any call. No arbitrary shell, URL fetch, or database query capability.
- Evaluation adapter: versioned inputs and independent outcomes exchanged with ChaosLab; no dependency on its private Python files.

Use Go, Chi, pgx, SQL migrations, structured logs, Prometheus metrics, and OpenTelemetry as the intended backend stack. UI framework and exact model are undecided. Keep provider and telemetry interfaces narrow; defer a plugin platform.

## Domain language

- Environment: an explicitly authorized sandbox or service deployment.
- Service: an observed application component.
- Signal: timestamped observation suggesting a problem.
- Investigation: bounded attempt to answer an operational question.
- Evidence: sourced, timestamped observation with an ID, freshness, scope, and provenance.
- Hypothesis: candidate explanation with supporting and conflicting evidence.
- Diagnostic: authorized read-only tool invocation used to distinguish explanations.
- Finding: report statement with evidence references and an explicit certainty category.
- Incident: tracked operational disruption that may have multiple investigations.
- Runbook: curated diagnostic or recovery procedure; suggested use is not authorization to execute.

## First vertical slice

1. Health endpoint and persisted service registration.
2. Create an investigation for a service, question, and bounded time window; return 202 and a stable ID.
3. Worker loads a recorded evidence bundle and records lifecycle states: queued, running, completed, failed, cancelled, or budget_exhausted.
4. A deterministic baseline produces a cited report with unknowns.
5. Add one real read-only telemetry adapter and then a bounded model/tool loop.
6. Read report, timeline, and evidence through versioned APIs; add a minimal web UI.

Acceptance: investigation survives worker restart without losing its audit history; cancellation/budget exhaustion stops new calls; invalid tool requests never execute; unauthorized environments are inaccessible; a report cannot cite nonexistent evidence; stale or missing data is visible. Include persistence tests, contract tests, logs, metrics, and reproducible requests.

## Initial scope and demo

Use one local Compose environment and a small representative set of failures: service unavailable, injected latency, and partial successful responses. Questions should include “what is slow?”, “what changed?”, and “what should I check next?”. Add resource-pressure scenarios only after the initial investigation works.

Demonstrate a reversible Visits failure that produces HTTP 200 with missing visits, followed by a crash where availability returns but data remains missing. OpsLens must distinguish service availability, response completeness, data preservation, and full resolution. Diagnosis is the first deliverable; these scenarios do not imply a recovery executor exists.

## Trust and execution boundaries

Treat logs, traces, labels, retrieved documents, and model output as untrusted. Redact secrets; bound inputs; maintain provenance; validate structured tool calls; authenticate requests and enforce environment authorization outside the model. Retrieved text cannot grant permissions. Read-only queries still need cost and load limits. Store provider usage/cost where available and enforce explicit per-investigation budgets.

MVP has no infrastructure mutations. Later remediation requires allowlisted actions, exact target identity, immutable plan-bound approval, expiry, precondition revalidation, idempotency, durable execution states, bounded attempts, and independent postconditions. Model credentials never confer executor or approval authority. Restarting is not inherently reversible and cannot reconstruct lost data.

## ChaosLab and evaluation

Keep repositories separate: OpsLens investigates; ChaosLab creates failures and evaluates outcomes. Exchange versioned JSON with environment/service identity, timestamps, evidence IDs, scenario version, code commits, and image digests. Keep fault labels and expected answers in evaluator-only records.

First strengthen ChaosLab admission locking, experiment-owned cleanup, safety-path tests, and separate availability/correctness verdicts. These are research recommendations; suspected race/cleanup issues need reproduction.

Compare a deterministic runbook selector with the AI on identical evidence. Begin with healthy controls, reversible failures, partial data, stale telemetry, misleading logs, and ambiguous causes. Split replay cases by scenario variant to reduce leakage. Proposed initial evaluation: about 30 labelled cases and repeated live scenarios; adjust to available time and cost.

Measure diagnosis/runbook selection, evidence accuracy, unsupported claims, appropriate abstention, tool-policy violations, false full-recovery declarations, latency, and model cost. Log cleanup ownership and never attribute watchdog restoration to OpsLens. Publish actual results with conditions; zero observed unsafe actions is not a universal guarantee.

## Milestones

1. Persisted investigation and deterministic replay report.
2. One live telemetry integration and constrained read-only diagnostics.
3. Model-driven investigation with budgets, citations, and abstention.
4. Timeline UI, ChaosLab evaluation, repeatable demo, documented results.
5. Human-approved remediation with independent semantic verification.
6. Narrow policy-authorized automatic recovery only after reliable evaluation.

Defer Kubernetes, multi-cloud, multi-tenancy, arbitrary command execution, autonomous code changes, model training, and general deployment management.

## Open decisions

Choose the first telemetry adapter, model/provider budget, evidence retention/redaction policy, UI framework, authentication mechanism, and sandbox boundary before their implementation slices. The aim is a useful Go backend product with a measured AI component, not a claim of production autonomous healing.
