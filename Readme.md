# OpsLens

An AI infrastructure investigator: understand why a service is failing or slow through evidence from metrics, logs, traces, deployments, and dependencies.

**Status: design stage.** This repository contains the product direction and implementation plan. No application, integrations, AI investigator, or recovery executor has been implemented yet. Previously named Aegis.

## The problem

Developers operating small service environments must piece together information across monitoring tools to explain failures and performance regressions. OpsLens will collect relevant evidence, investigate competing explanations, and make its findings inspectable.

The first question it should answer is: **“Why is this service failing or slow?”**

## Intended experience

1. Select a service and time window, or ask a question about an alert.
2. Inspect recent changes and symptoms.
3. Follow an investigation timeline as the AI requests bounded, read-only diagnostics.
4. Read a report separating observations, hypotheses, conflicting evidence, and unknowns.
5. Open cited evidence and choose the next diagnostic or suggested runbook.
6. Save and replay investigations to evaluate their quality.

Example, not an implemented result: checkout slowed after a deployment; traces suggest connection-pool waiting rather than high database CPU. OpsLens should propose a discriminating check, not treat timing correlation as proven causation.

## What makes this an AI harness

The model proposes hypotheses and diagnostic steps. The harness supplies scoped context, validates tool requests, enforces permissions and budgets, executes approved read-only queries, and records evidence and outcomes. Reports must cite evidence IDs. Missing telemetry is unknown, not healthy.

## First release

One local Docker Compose environment, one model provider, service overview, investigation timeline, bounded monitoring queries, evidence-backed reports, and saved replay cases. The backend direction remains Go, REST, PostgreSQL, and a worker for investigations. Monitoring systems retain raw telemetry; OpsLens stores normalized references and bounded evidence snapshots.

Infrastructure mutation is outside the first release. Human-approved remediation follows useful diagnosis; narrowly preauthorized healing follows demonstrated execution safety and recovery verification.

## Built-in evaluator

OpsLens will include its own evaluation module, fixtures, scenario catalogue, and replay runner in this repository. It will compare investigations with labelled expected outcomes and measure evidence quality, diagnosis, abstention, latency, and cost. Expected answers stay outside the investigator's evidence context. ChaosLab remains a separate project with no planned integration or dependency.

## Documentation

- [Product direction, architecture, and milestones](Docs/product-direction.md)
- [Project instructions](AGENTS.md)
- [Historical API study guide](Docs/learnings/api-guide.md)
- [Historical week-one exercise](Docs/logs/week1.md)

The older Aegis documents are retained as historical learning material. The product direction supersedes their remediation-first implementation order.
