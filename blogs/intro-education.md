---
title: "Teaching Scientific Computing at Scale: PegasusAI in the Classroom"
date: 2026-09-03
author: Pegasus AI Team
category: education
tag: Education
description: HPC workflows are not just a research tool — they are a skill students need. Pegasus AI integrates into academic courses to give students direct experience with real scientific computing infrastructure, without requiring weeks of setup.
---

High-performance computing is no longer a niche skill. Across the sciences — biology, climate, physics, data science — the questions researchers want to ask require more compute than a laptop can provide. Graduate students and upper-level undergraduates increasingly need to work with HPC systems, yet most curricula have not caught up.

The gap is not motivation. Students want to engage with real infrastructure. The gap is access and time. Setting up an HPC environment, learning the workflow tooling, and getting a job running on a cluster can take weeks. In a semester course, that is most of the course.

## What Pegasus AI brings to the classroom

Pegasus AI is designed to shrink that setup time dramatically while giving students genuine experience with production-grade workflow infrastructure.

The AI-assisted workflow tools handle the boilerplate. A student describes a scientific pipeline in plain language — the data sources, the analysis steps, the expected outputs — and gets back a working workflow generator, a container definition, and a test script. The scaffolding is real: the same tools and patterns used in production research workflows, not a simplified educational version.

What the student learns is the science of the workflow: how to decompose a computational problem into stages, how to handle failures gracefully, how to think about data movement at scale. The tooling knowledge comes along naturally, without becoming the entire focus of the course.

## Designed for real data and real infrastructure

The workflows students build with Pegasus AI run on the same resources production researchers use: ACCESS HPC allocations, campus clusters, cloud providers. Students see real queue times, real node failures, real data movement costs.

This is not incidental. The goal is to give students the intuition that comes from working with actual infrastructure — the understanding of why certain design decisions matter, why a workflow that works on a laptop can behave differently at scale, why reproducibility requires more than running the same code twice.

A workflow built in a course is also a starting point for a research project. Several workflows developed in classroom settings have become the basis for published research, because the infrastructure was real enough to extend.

## Course integration

Pegasus AI fits into courses as a lab component, a project platform, or a research methods module. Instructors can:

- Assign workflow design as a project with automated validation via `/pegasus-review`
- Use existing workflow repositories as starting points for student extension projects
- Structure debugging exercises around intentionally broken workflows, using `/pegasus-debug` as a teaching tool

The plugin-based interface means students work in the same environment they would use in a research lab. There is no educational mode, no sandbox that insulates students from real decisions. The constraints they encounter are the same constraints a researcher encounters.

## What students take away

A student who has built and run a Pegasus workflow has:

- Written a workflow generator that scales from 2 to 200 input datasets with a single argument
- Built and published a container that packages their scientific tools
- Debugged a job failure on a cluster they could not log into
- Reproduced someone else's workflow run from its provenance record

These are not theoretical skills. They are the practical competencies a researcher needs to participate in modern computational science — and they transfer directly from the classroom to the lab.

---

*Part of the Pegasus AI research program, funded by the National Science Foundation under award [2513101](https://www.nsf.gov/awardsearch/showAward?AWD_ID=2513101).*
