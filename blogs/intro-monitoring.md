---
title: "Always On: Real-Time Workflow Monitoring with Pegasus AI"
date: 2026-07-28
author: Pegasus AI Team
category: monitoring
tag: Monitoring
description: Scientific workflows run for hours or days across hundreds of nodes. Pegasus AI's online monitoring layer watches every job in real time — detecting anomalies, surfacing bottlenecks, and giving researchers the introspection they need to act before a run goes wrong.
---

A workflow that takes three days to finish gives you a lot of time to wonder whether it is going well. The standard answer is to check back when it is done and read the logs. The problem with that answer is that by the time something shows up in the logs, the damage is already done.

A job that stalled six hours ago has been holding a slot for six hours. A memory spike that crossed a threshold four hours in triggered a cascade of retries you did not know about. A site that started throttling connections two hours ago has been silently slowing everything downstream.

None of these show up until you look, and most people look at the end.

## What online monitoring means

Online monitoring means watching a workflow while it runs, not after. For Pegasus AI, this means tracking performance metrics at the job level in real time: CPU utilization, memory consumption, I/O rates, network activity, and job state transitions — all streaming as the workflow executes.

The goal is not just to collect data. It is to make the data actionable while there is still time to act.

That requires two things: the ability to detect when something is wrong, and the ability to explain what it is. A dashboard that shows a spike in memory usage is useful. A system that tells you which job caused it, what input triggered it, and whether the same pattern has appeared in previous runs is much more useful.

## Anomaly detection at workflow scale

Production scientific workflows execute thousands of jobs across heterogeneous resources. No human can watch all of them. Pegasus AI's monitoring layer applies anomaly detection automatically — flagging jobs whose behavior deviates from what has been observed for that task type under similar conditions.

Deviation can take several forms: a job that is running much longer than expected for its input size, a job consuming significantly more memory than similar past runs, a job whose I/O pattern suggests it is reading data it should not need. Each of these is a signal, and each requires a different response.

The monitoring system surfaces these signals in context. Not just "job X is slow" but "job X is slow given its input size, execution site, and the runtime distribution for this task type in the training history."

## Bottleneck identification

Beyond individual jobs, the monitoring layer tracks workflow-level performance: which stages are consistently the slowest, where data movement is introducing delays, which sites are underperforming relative to their allocation. This is the information a researcher needs to improve a workflow for the next run — not just to understand what happened in this one.

Bottleneck identification at this level requires correlating metrics across jobs, stages, and sites simultaneously. A single slow job is noise. A pattern of slow jobs at the same stage across multiple runs is a design problem worth fixing.

## Designed to run alongside execution

Pegasus AI's monitoring is designed as a lightweight layer on top of the existing Pegasus-WMS execution infrastructure. It does not modify workflows or require changes to how jobs are submitted. It streams data from HTCondor and the execution environment and processes it in parallel with the workflow run.

The result is full observability from job submission to final output — with anomaly alerts and bottleneck reports available while the workflow is still running, not just when it is done.

---

*Part of the Pegasus AI research program, funded by the National Science Foundation under award [2513101](https://www.nsf.gov/awardsearch/showAward?AWD_ID=2513101).*
