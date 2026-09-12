---
title: From Scripts to HPC: A Worked Example with Pegasus and Claude Code
date: 2026-09-08
author: Pegasus AI Team
category: ai-workflows
tag: Workflows
description: A complete walkthrough of turning five Python scripts into a parallel HPC workflow using Pegasus AI — from a single prompt to a 25-job run on a live HTCondor pool.
---

You have a set of scripts that work. They pull some data, compute something, write
results, and they run fine on your laptop. The next step is more data than a laptop can
hold, or more runs than you want to sit and watch. That means a campus cluster, a cloud
allocation, a testbed like FABRIC or Chameleon, or an award of NAIRR or ACCESS compute.

Pegasus is built for that step. You describe your pipeline once, and the same
description runs on your laptop, a local HTCondor pool, a campus cluster, or a cloud
allocation. Pegasus decides where each job goes, moves the data to it, retries what
fails, and keeps a record complete enough to reproduce the run later. Your scripts stay
ordinary scripts.

Writing the description is not complicated. You say what each step reads and writes,
which results you want to keep, and roughly what each step needs to run. It is
straightforward work, but there is a fair amount of it, and a small slip tends to
surface an hour later as a job sitting idle on a cluster rather than as an error in
front of you. That is the part the `pegasus-ai` skills take over.

The rest of this post is one workflow from start to finish: the prompt, what came back,
and what happened when we ran it. Scripts are only one place to start from, and there is
a table near the end covering the others.

## The prompt

A description of what you already have is enough to start. No Pegasus vocabulary
required:

```
I have five Python scripts in bin/ that each work on their own:

  fetch_sage_data.py         pulls air quality readings for one sensor node
  extract_aqi_timeseries.py  builds an hourly time series from those readings
  analyze_pollutants.py      writes summary statistics and a plot
  detect_anomalies.py        flags unusual readings
  merge.py                   combines results from several nodes

They all take --input and --output. I want to run them over several sensor
nodes at the same time, on a cluster, and get back one combined report plus
a plot for each node. Please set this up as a Pegasus workflow and put the
tools in a container so it can run somewhere other than my laptop.
```

No Pegasus terms appear anywhere in that. What the assistant needs is the shape of the
work: five steps, one that runs per node, one that combines across nodes, and somewhere
bigger to run it.

## What came back

```
airquality-workflow/
├── workflow_generator.py     builds the workflow and submits it
├── bin/                      your scripts, wrapped for Pegasus
├── Apptainer/
│   └── AirQuality_Forecast_Container.def
├── README.md
└── requirements.txt
```

The generator is the only new thing to understand. It reads command line arguments,
works out how many copies of each step are needed, and writes the workflow out for
Pegasus to plan. Your scripts are untouched; wrappers handle the arguments and the file
names.

## The shape of it

![How the air quality workflow is put together](assets/img/blog/airquality-workflow-shape.svg)

Everything inside the dashed box happens once per sensor node, in parallel. The merge
step waits for all of them and produces a single report.

Asking for more nodes is a command line argument rather than a rewrite. The same file
describes two nodes or two hundred.

## Does it work?

Yes. We ran it on a small HTCondor pool against two live SAGE sensor nodes, asking for
two days of readings:

```
./workflow_generator.py --data-source sage --sage-vsn W045 W06C \
    --start-date 2026-09-06 --end-date 2026-09-08 -o workflow_sage.yml
```

Twenty-five jobs, no failures, about five minutes start to finish. The tools travel to
the cluster inside a container that the generated project also builds for you, so
nothing needs installing on the machines that run the work.

![PM2.5 measured at SAGE node W045](assets/img/blog/sage-W045-analysis.png)

Forty-eight hourly PM2.5 readings from node W045. The anomaly step flagged the spike at
19:00 on 6 September by itself, and the merge step combined both nodes into one report.

The second node is the more interesting result:

![PM2.5 measured at SAGE node W06C](assets/img/blog/sage-W06C-analysis.png)

W06C publishes sparsely. Over the same two days that gave W045 forty-eight readings, it
produced two. It did not hold up the workflow or take the run down with it. It produced
a thin but valid result and everything downstream carried on.

That behaviour is deliberate, and if you take one idea from this post, take this one.

A step that cannot get its data still writes the file it promised, then reports the
failure. If it exits without writing anything, Pegasus reports a missing file instead of
your actual error, holds the job, and the workflow stops with nothing useful in the log.

Sources that might legitimately be empty are treated as best effort, so the run only
fails if all of them come back empty.

## The skills

Everything above came from one plugin. Install it from inside Claude Code:

```
/plugin marketplace add pegasus-isi/claude-plugin-marketplace
/plugin install pegasus-ai@scitech
```

There is nothing else to configure. It adds these:

| Skill | Use it when |
|---|---|
| `/pegasus-help` | You are not sure where to start |
| `/pegasus-scaffold` | New workflow, empty directory |
| `/pegasus-wrapper` | Adding one step to a workflow that exists |
| `/pegasus-apptainer` | Building the container |
| `/pegasus-convert` | You already have a Snakemake or Nextflow pipeline |
| `/pegasus-review` | Written but not yet run |
| `/pegasus-debug` | A job failed and the log is not obvious |

`/pegasus-scaffold` produced the project above. Two others become useful quickly.

`/pegasus-review` reads what you have and reports the mistakes that fail silently: a step
that searches a directory for its inputs instead of being handed them, or a container
that has not been rebuilt since its definition changed.

`/pegasus-debug` takes a failed job's output and works backwards to the cause. That saves
real time the first few times something breaks on a machine you cannot log into.

Each skill reads the Pegasus reference and the closest existing workflow before writing
anything, so what comes out follows established patterns rather than invented ones. The
scaffold also includes a script that runs each step outside Pegasus, so you can check
the science on your laptop and leave only the distribution problems for the cluster.

## Wherever you are starting from

Scripts are the easiest case, but they are not the common one. Across the workflows
we have built, the starting material has been all of these:

| You have | The way in | Workflows built this way |
|---|---|---|
| A sentence describing what you want | Just ask | soilmoisture, airquality, earthquake |
| A hand-drawn sketch of boxes and arrows | Paste the photo in | napkin-vibe |
| A published paper with a methods section | Point at the PDF | seaice |
| A Nextflow pipeline | `/pegasus-convert` | mag, proteinfold, rnaseq |
| A Snakemake pipeline | `/pegasus-convert` | tnseq |
| Working code in notebooks or scripts | `/pegasus-scaffold` | s2-segmentation, seaice |
| A specification and some datasets | `/pegasus-scaffold` | medical-imaging-fl |

The sketch case surprises people. A photograph of a whiteboard is a perfectly good
input. The drawing is read directly, the stages and the branching are pulled out of it,
and what comes back is a workflow with that shape. There is no transcription step.

Four of the workflows above started from nothing more than a sentence. The actual
prompts were:

```
I need a workflow for soil moisture to answer: should you water or not?

Can you add ML to the soil moisture workflow?

Create an air quality workflow similar to the orcasound workflow

Build a workflow using earthquake.usgs.gov data
```

Four workflows, about twenty-two prompts, three and a half days. The second one is the
useful example: an existing workflow gained a machine-learning stage from a single
sentence, because the workflow was already described in a form that could be extended
rather than rewritten.

## Streamflow workflow tutorial

<video controls preload="metadata" style="width:100%;border-radius:1rem;border:1px solid rgba(255,255,255,0.08);background:#000;margin:1rem 0">
  <source src="assets/videos/streamflow-workflow-tutorial-720p30.mp4" type="video/mp4">
</video>

## Try it

```
/plugin marketplace add pegasus-isi/claude-plugin-marketplace
/plugin install pegasus-ai@scitech
/pegasus-help
```

Then describe your own scripts the way the prompt above does.

Workflows built this way, all runnable:
[airquality](https://github.com/pegasus-isi/airquality-workflow),
[earthquake](https://github.com/pegasus-isi/earthquake-workflow),
[tnseq](https://github.com/pegasus-isi/tnseq-workflow),
[mag](https://github.com/pegasus-isi/mag-workflow),
[soilmoisture](https://github.com/pegasus-isi/soilmoisture-workflow),
[gwas-qc](https://github.com/pegasus-isi/gwas-qc-workflow),
[rnaseq](https://github.com/pegasus-isi/rnaseq-workflow),
[proteinfold](https://github.com/pegasus-isi/proteinfold-workflow),
[s2-segmentation](https://github.com/pegasus-isi/s2-segmentation-workflow),
[medical-imaging-fl](https://github.com/pegasus-isi/medical-imaging-fl-workflow),
[sra-search](https://github.com/pegasus-isi/sra-search-pegasus-workflow).

A companion post covers [keeping a longer build on track](blog.html?post=spec-driven-scientific-workflows).

*The `pegasus-ai` plugin is part of the [SciTech Claude Code Plugin Marketplace](https://github.com/pegasus-isi/claude-plugin-marketplace). Funded by the National Science Foundation under award [2513101](https://www.nsf.gov/awardsearch/showAward?AWD_ID=2513101).*
