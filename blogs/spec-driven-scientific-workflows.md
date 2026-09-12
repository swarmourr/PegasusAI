---
title: Write the Spec First: A Worked Example
date: 2026-09-08
author: Pegasus AI Team
category: ai-workflows
tag: Best Practices
description: How writing a specification document before any code saves time, surfaces hidden assumptions, and produces better scientific workflows — shown end-to-end on a real GBIF species comparison pipeline.
---

There is a simple way to get better results from an AI assistant on a piece of real
work. Ask it to write down what it is going to build, read that, fix it, and only then
ask for the code.

That is the whole idea. This post shows it on a small workflow I built for the purpose,
including the parts that went wrong.

The workflow compares where and when several species have been recorded, using public
data from GBIF. It needs no account and no API key, so you can run it yourself.

## The first prompt

```
Before writing any code, write SPEC.md for a small Pegasus workflow that
compares where and when several species have been recorded, using public
GBIF occurrence data. No API key.

Cover:
  - Goal, in a paragraph.
  - Inputs: where the data comes from, and whether each source is
    required or optional.
  - Stages: each step, its inputs and outputs, and what it should do
    when an input is missing or empty.
  - Constraints: what the workflow must do. Number them.
  - Non-constraints: what it does not need to do, so you don't build
    it. Number these too.
  - Validation criteria: for each one, how I would check it passed.
  - Open design issues: anything you cannot decide for me. Give me the
    options and let me choose.

No code yet. Where you are unsure, ask me instead of picking.
```

Two things in there do most of the work.

The first is "no code yet". Without it you get a working workflow and a description
written to match, which is not the same thing as a design.

The second is the last line. An assistant that is unsure will still choose something, and
it will choose plausibly, and you will never know it was a choice. Asking it to stop and
ask turns silent guesses into a short list of questions.

## What came back

A 123-line document. Short enough to read properly, which matters more than it sounds:
a specification nobody reads is just a slower way of writing code.

The shape it proposed:

![How the species workflow is put together](assets/img/blog/species-workflow-shape.svg)

For each species, fetch the records and summarise them. Then one step merges everything
into a comparison. Two jobs per species plus the merge.

**The constraints were numbered**, which turns out to be the useful part. Not because
numbering is tidy, but because later you can say "that is required by C6" instead of
having the same argument twice.

> **C1**: Require no credentials. A reader with a clean machine can run it.
>
> **C6**: A species with zero records is not an error. Write an empty file with the
> correct header, log it, and carry on, so one absent species cannot fail the run.

C6 is a decision about what the workflow is *for*. It compares species, so failing
everything because one name returned nothing would be the wrong behaviour. No amount of
good engineering gets you that answer. You have to decide it.

**The non-constraints saved the most time.** This is the section people leave out:

> **N3**: Correct for sampling bias. Occurrence records reflect how much people look, as
> much as where a species lives. The output describes records, not distributions, and
> says so.
>
> **N4**: Draw a real map. A scatter of coordinates on plain axes is enough. Coastlines
> would pull in a heavyweight geospatial library for no analytical gain.

Look at what N4 prevents. "Plot where the species has been recorded" reasonably suggests
a map with coastlines. That means a geospatial stack, a much bigger container, and a
longer build, none of which makes the result better. One sentence removed all of it.

**The validation criteria came with pass conditions.** Not "check the table looks right",
but something a person who did not write the code could verify:

> **V4**: The comparison table has exactly one row per requested species, including any
> that returned no records.
>
> **V5**: A run including a deliberately absent name completes successfully, with that
> species shown in the table with a zero count.

V5 changed how the thing got tested. Because the criterion was written down, a fake
species name went into the very first run. Without it, that failure mode gets discovered
by accident, months later, by somebody else.

## Reading it before building

This is the step that gets skipped. It took about five minutes and it is where the value
is.

Three things to look for:

1. **Requirements that are really preferences.** Anything written as a rule will be
   defended for the rest of the project, so be sure you meant it.
2. **Missing non-constraints.** Whatever the document does not rule out, the assistant
   may build.
3. **Criteria that do not actually check anything.** "Produces a table" is not a
   criterion. V4 above is.

The document also listed the questions it could not answer, which is exactly what the
last line of the prompt asked for. One of them:

> **Where to resolve species names.** Doing it on the submit machine would let the
> generator name files after the official species ID. Doing it inside the job means the
> submit machine needs no internet access at all. **Chosen: inside the job.**

That has no correct answer in the abstract. It depends on where you plan to run. It took
a minute to settle on paper and it never came up again.

## The second prompt

```
Implement SPEC.md as a Pegasus workflow.
```

That is the whole thing. It can be one line because everything it needs was decided
already.

## What happened when it ran

Five species, one of them invented, on a cluster:

| | |
|---|---|
| Jobs | 23 |
| Succeeded | 23 |
| Failed | 0 |
| Retries | 5 |
| Wall time | 3 min 52 s |

The five retries are the interesting number. GBIF refused several connections partway
through the run. The jobs waited, tried again, and finished. Constraint C5 had asked for
exactly that, and I would have dismissed it as boilerplate if it had not just saved the
run.

![Comparison across five species](assets/img/blog/species-comparison.png)

On the left, every real species hits the record cap, and the invented one sits at zero
without having taken anything down with it. That is V4 and V5 passing on real
infrastructure rather than in principle.

On the right is the actual result. *Bombus terrestris* has the most northerly records of
the four, and Britain as its top recording country, which is what you would expect of a
European bumblebee.

## What the specification got wrong

Two things, and they are worth more than the successes.

**It said nothing about how arguments are passed.** Every fetch job died immediately.
Pegasus separates job arguments by spaces, and a species name like `Danaus plexippus`
has a space in it, so the job received two arguments where it expected one and exited
before writing anything at all. Nothing in the specification anticipated this. It is
now constraint C8.

**A criterion passed while the thing it was meant to guarantee failed.** One rule said
every record's year must fall inside the requested range. It did. But 900 records
requested for 2015 to 2024 all came from 2024, because GBIF returns its own ordering and
the cap takes whatever comes first. The year range decides which records are *eligible*.
It does not spread the sample across them. That is now written down as a non-constraint,
along with how it was found.

Writing things down first did not prevent either problem. What it did was make them
visible, and give them somewhere to live once they were found. Two of the eight
constraints in that document were written after something broke.

## Why this is worth an hour

Now the general case.

**Changing a document is cheap. Changing code is not.** The same disagreement costs a
paragraph before the build and a rewrite afterwards.

**It forces the dull questions early.** Is this input required? What happens when a
source is down? Short questions with short answers, and if you skip them you meet them
later as failures on a cluster.

**Saying what you do not want stops over-building.** Given an open goal, an assistant
builds the ambitious version of it, and you spend your review time removing work you
never asked for.

**A colleague can review a specification.** They probably cannot review the workflow
code. If a scientific assumption is wrong, the document is where somebody notices.

**It survives the session.** An assistant remembers nothing from last time. A document in
the repository is the only thing that does.

## A starting template

Delete whatever does not apply:

```markdown
# SPEC: <project>

## 1. Goal
   what this must accomplish, in a paragraph
## 2. Inputs
   where the data comes from, and whether each source is required
## 3. Stages
   the steps and how they connect
## 4. Constraints: what it MUST do
   numbered, each one checkable
## 5. Non-constraints: what it need NOT do
   numbered; the section that saves the most time
## 6. Expected outcomes
   what a successful run produces
## 7. Validation criteria
   a table: id, criterion, how you would check it
## 8. Open design issues
   the options, the choice, and why; leave open ones marked open
## 9. Deliverables
   the files you expect at the end
```

The workflow, its specification and the run outputs are all in the repository, if you
want to see what the finished document looks like next to the code it produced.

A companion post covers [building a workflow with the pegasus-ai skills](blog.html?post=build-a-pegasus-workflow-with-claude-skills).

*The `pegasus-ai` plugin is part of the [SciTech Claude Code Plugin Marketplace](https://github.com/pegasus-isi/claude-plugin-marketplace). Funded by the National Science Foundation under award [2513101](https://www.nsf.gov/awardsearch/showAward?AWD_ID=2513101).*
