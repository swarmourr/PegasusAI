---
title: "Built for the Infrastructure: CI-Ready Design in Pegasus AI"
date: 2026-09-12
author: Pegasus AI Team
category: ci-design
tag: CI Design
description: Pegasus AI is not a standalone tool — it is designed from the ground up to work within the NSF cyberinfrastructure ecosystem. What CI-ready design means in practice, and why it matters for scientific reproducibility and interoperability.
---

Scientific software that only runs in one environment is not really scientific software — it is a local artifact. The whole point of computational science is that results should be reproducible, that methods should be transferable, and that infrastructure should not be the limiting factor in what questions can be asked.

Pegasus AI is built with that principle at its core. Every component is designed to integrate with the NSF cyberinfrastructure ecosystem — not as an add-on, but as a first principle.

## What CI-ready means

CI-ready design means the system works within the infrastructure that exists, not the infrastructure that would be ideal.

In practice, this means:

**Portability across resources.** A workflow described once should run on a campus cluster, an ACCESS allocation, a cloud provider, or a local HTCondor pool without modification. Pegasus handles site adaptation — choosing where each job runs, moving data to where it is needed, retrying on failure — so the workflow description stays independent of the execution environment.

**Container-based packaging.** Every workflow Pegasus AI generates includes a container definition that packages the scientific tools the workflow needs. The container travels with the workflow, so the execution environment on any site is the same. A result produced on an ACCESS cluster and a result produced on a campus cluster are produced by the same software stack.

**Provenance by default.** Every Pegasus workflow execution produces a complete provenance record: what ran, what inputs it used, what outputs it produced, when, where, and under what conditions. This record is not optional metadata — it is part of the execution infrastructure. Reproducing a result means replaying the provenance record, not reconstructing it from notes.

**Modular architecture.** Pegasus AI's components — runtime prediction, monitoring, AI-assisted creation — are built as modules that plug into the existing Pegasus-WMS execution pipeline. They do not require a separate deployment. They add capability to an infrastructure that science already uses.

## Why the NSF ecosystem specifically

The NSF cyberinfrastructure ecosystem — ACCESS, FABRIC, Chameleon, NAIRR, and the campus clusters connected to them — represents a large fraction of the compute available to U.S. researchers. Pegasus has been a part of that ecosystem for over a decade. Pegasus AI extends that presence with AI-assisted capabilities designed to work within the same operational constraints.

This matters for adoption. A tool that requires researchers to change their infrastructure, learn a new job submission system, or migrate their data somewhere new will be used by the people who are already enthusiastic adopters. A tool that slots into the infrastructure researchers already use will be used by everyone else — the scientists who need it most.

## Integration in practice

When a researcher submits a Pegasus AI workflow to an ACCESS allocation:

1. The Pegasus Planner generates the workflow structure and job descriptions.
2. The Runtime Predictor patches each job description with a predicted walltime bound before submission.
3. HTCondor schedules the jobs across the available resources.
4. The monitoring layer streams performance data as jobs execute.
5. The provenance database records every step of the execution.

None of these steps require the researcher to interact with the underlying infrastructure directly. The CI-ready design means the infrastructure handles itself.

## Reproducibility as a feature

The combination of container packaging, provenance recording, and workflow-level description means that a Pegasus AI workflow is reproducible by construction. Not reproducible in principle — reproducible in practice, by another researcher, on a different resource, months later.

That is what CI-ready design is ultimately for: making the infrastructure invisible, so the science is what survives.

---

*Part of the Pegasus AI research program, funded by the National Science Foundation under award [2513101](https://www.nsf.gov/awardsearch/showAward?AWD_ID=2513101).*
