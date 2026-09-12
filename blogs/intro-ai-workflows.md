---
title: "From Idea to HPC Pipeline: AI-Assisted Workflow Creation"
date: 2026-08-05
author: Pegasus AI Team
category: ai-workflows
tag: AI Workflows
description: Most scientists who need HPC workflows are not workflow engineers. Pegasus AI uses LLM-powered tools to help researchers compose, validate, and run scientific pipelines — from a description of the problem, not a knowledge of the tooling.
---

Writing a Pegasus workflow is not conceptually difficult. You describe what each step reads, what it writes, what resources it needs, and which steps depend on which. Do that clearly enough and Pegasus will plan and execute the workflow across any supported resource.

The difficulty is that doing it clearly enough takes time and specific knowledge. You need to know the Pegasus API, the container format, the site catalog structure. You need to know that a step which searches a directory for its inputs will behave unpredictably at scale, and that a container that has not been rebuilt since its definition changed will silently run stale code.

Most scientists who need a workflow do not have that knowledge and should not need to acquire it to do their work.

## What AI-assisted workflow creation changes

Pegasus AI adds LLM-powered tools to the Claude Code environment that close the gap between scientific intent and workflow implementation.

The starting point can be almost anything. A description of five scripts and what they do. A photograph of a whiteboard with boxes and arrows. A published paper with a methods section. An existing Snakemake or Nextflow pipeline. In each case, the assistant reads what you provide, asks about anything it cannot determine, and produces a working Pegasus workflow.

The workflow comes with:

- A generator script that builds and submits the workflow
- A container definition that packages the tools the workflow needs
- A local test script that validates each step before cluster submission
- A README that describes what the workflow does and how to run it

None of these require workflow engineering knowledge to produce. They require a clear description of the science.

## The skills

The AI-assisted workflow tools are packaged as a Claude Code plugin with specific skills for specific tasks:

| Skill | When to use it |
|---|---|
| `/pegasus-scaffold` | Starting from scratch — new workflow, empty directory |
| `/pegasus-wrapper` | Adding one step to a workflow that already exists |
| `/pegasus-convert` | You have a Snakemake or Nextflow pipeline |
| `/pegasus-review` | Workflow is written but not yet run |
| `/pegasus-debug` | A job failed and the log is not obvious |

Each skill reads the Pegasus reference documentation and the closest existing workflow before writing anything. What comes out follows established patterns rather than invented ones.

## What validation and debugging add

Writing a workflow is one thing. Getting it right is another.

`/pegasus-review` reads a workflow before it runs and reports mistakes that fail silently: a step that expects files in a location Pegasus will not put them, a container definition that has drifted from the code it is meant to package, a resource request that does not match the task's actual requirements.

`/pegasus-debug` takes a failed job's output and works backwards to the cause. On a cluster you cannot log into, this is the difference between a ten-minute turnaround and a three-hour one.

## What this looks like in practice

Across the workflows built with these tools, the starting material has ranged from a single sentence to a published paper. Four workflows started from nothing more than a sentence describing the scientific question. Several more came from Nextflow or Snakemake pipelines. One came from a photograph of a whiteboard sketch.

In every case, the output was a working workflow — tested on real infrastructure, with containers that build, with error handling that matches the failure modes of the data sources it queries.

The goal is not to replace workflow engineers. It is to make HPC accessible to the researchers who need it but do not have the time or background to become workflow engineers first.

---

*See it in practice: [From Scripts to HPC — a worked example](blog.html?post=build-a-pegasus-workflow-with-claude-skills)*

*Part of the Pegasus AI research program, funded by the National Science Foundation under award [2513101](https://www.nsf.gov/awardsearch/showAward?AWD_ID=2513101).*
