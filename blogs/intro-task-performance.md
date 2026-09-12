---
title: "Smarter Scheduling: Why HPC Needs Better Runtime Estimates"
date: 2026-07-14
author: Pegasus AI Team
category: task-performance
tag: Task Performance
description: Why a single walltime estimate is not enough for production HPC workloads — and how Pegasus AI uses ML-based runtime envelopes to give schedulers the information they actually need.
---

Every job on an HPC cluster needs a walltime limit before it can run. Set it too tight and the job gets killed before it finishes. Set it too loose and you hold a slot that another job could have used. Most sites default to something like 3600 seconds and move on.

That default works until it doesn't. On a busy cluster, consistently over-estimated walltimes push real queue wait times up for everyone. Underestimated ones waste compute hours on jobs that never complete. And neither outcome shows up cleanly in any single metric — the damage is diffuse.

## The core problem

Runtime prediction in HPC is hard for a specific reason: at the moment a scheduling decision must be made, you have almost no information about what the job will actually do.

You know the task name. You know what resources were requested. You know the size of the input data. You know where the job will run. You do not know the execution path, the branching behavior, or how the application will interact with the memory hierarchy and I/O subsystem on that particular node on that particular day.

That uncertainty does not go away with a better model. It is structural. A job that ran in 80 seconds last Tuesday might run in 5000 seconds today — same code, same inputs, different system load. The right response is not to predict a single number more accurately. It is to represent the uncertainty explicitly.

## What Pegasus AI does differently

Instead of predicting one runtime, Pegasus AI predicts three: a lower bound, a central estimate, and an upper bound. The upper bound replaces the static walltime. The central estimate is used for scheduling cost accounting. The lower bound can flag jobs that finish suspiciously fast — a signal worth investigating.

All three come from the same model, trained on pre-execution information only. The model adapts to how much history is available for each task type: well-represented tasks use a learned predictor with task-specific historical references; sparse tasks use empirical quantiles from their limited history; tasks that have never been seen before use a hierarchical fallback that transfers runtime information from related task families and configuration buckets.

The result is a consistent representation — lower, central, upper — regardless of which estimation path was taken.

## Why this matters in practice

Against a static 3600 s walltime, the predicted upper bound reduces mean excess runtime estimation by 47.5% while keeping the underestimation rate below 1%. For tasks that have never appeared in training, under-prediction drops from 28.9% (the task-median baseline) to 4.3%.

For a site running hundreds of workflows a day, that difference translates directly into shorter queue times, fewer preempted jobs, and better utilization of allocated compute.

The framework is integrated into Pegasus-WMS as a lightweight post-planning step. It operates on the generated workflow, patches the submit files, and hands control back to HTCondor. No changes to the workflow specification, no changes to the execution path.

---

*Dive deeper: [Beyond Point Estimates — the full technical post](blog.html?post=runtime-prediction-interval-envelopes)*

*Part of the Pegasus AI research program, funded by the National Science Foundation under award [2513101](https://www.nsf.gov/awardsearch/showAward?AWD_ID=2513101).*
