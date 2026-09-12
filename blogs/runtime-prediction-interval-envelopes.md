---
title: "Beyond Point Estimates: Predicting HPC Job Runtime as an Interval"
date: 2026-08-26
author: Pegasus AI Team
category: task-performance
tag: Research
description: A single runtime estimate is not enough for HPC scheduling. We show how representing execution time as a lower–central–upper envelope, combined with a strategy that adapts to how much history is available per task, cuts under-prediction by up to 6.7× compared to standard methods.
---

Every job submitted to an HPC cluster needs a runtime estimate. Get it wrong in one direction and the job gets killed before it finishes. Get it wrong in the other and you waste allocation, block the queue, and frustrate everyone downstream.

The standard answer is a single number — a walltime limit. The standard result is that it is either too tight or too conservative, and neither outcome is good. This post describes a different approach: instead of predicting one number, predict three.

## The problem with point estimates

Consider a single workflow task, `hdf_trigger_merge_ID6`, executed 178 times across different conditions. Its runtime ranged from tens of seconds to several thousand — more than two orders of magnitude. The median was stable. The tails were not.

This is not unusual. Across production workflow traces collected from Pegasus-WMS over three months on ACCESS HPC resources, the same pattern appeared repeatedly: stable medians, heavy tails, and outliers in every input-size and CPU-configuration group.

A single predicted runtime cannot represent this. It either covers the tail and wastes allocation, or it targets the median and gets preempted. The right representation is a range.

## A three-point runtime envelope

<div style="margin:2rem 0;padding:1.5rem;background:rgba(255,255,255,0.03);border:1px solid rgba(255,255,255,0.08);border-radius:1rem;overflow:hidden">
<svg viewBox="0 0 680 260" xmlns="http://www.w3.org/2000/svg" style="width:100%;display:block;font-family:inherit">
  <!-- shaded envelope band -->
  <polygon points="60,200 160,155 260,120 360,105 460,98 560,90 580,90 580,48 460,55 360,62 260,70 160,100 60,148" fill="rgba(56,189,248,0.08)" />
  <!-- upper bound -->
  <polyline points="60,148 160,100 260,70 360,62 460,55 560,48" fill="none" stroke="#38bdf8" stroke-width="1.5" stroke-dasharray="5,3" />
  <!-- central estimate -->
  <polyline points="60,200 160,155 260,120 360,105 460,98 560,90" fill="none" stroke="#38bdf8" stroke-width="2.5" />
  <!-- lower bound -->
  <polyline points="60,230 160,205 260,185 360,175 460,168 560,162" fill="none" stroke="#38bdf8" stroke-width="1.5" stroke-dasharray="5,3" />
  <!-- x axis -->
  <line x1="50" y1="245" x2="590" y2="245" stroke="rgba(255,255,255,0.15)" stroke-width="1"/>
  <!-- y axis -->
  <line x1="50" y1="30" x2="50" y2="245" stroke="rgba(255,255,255,0.15)" stroke-width="1"/>
  <!-- x tick labels -->
  <text x="60" y="258" fill="#64748b" font-size="11" text-anchor="middle">Job 1</text>
  <text x="160" y="258" fill="#64748b" font-size="11" text-anchor="middle">Job 2</text>
  <text x="260" y="258" fill="#64748b" font-size="11" text-anchor="middle">Job 3</text>
  <text x="360" y="258" fill="#64748b" font-size="11" text-anchor="middle">Job 4</text>
  <text x="460" y="258" fill="#64748b" font-size="11" text-anchor="middle">Job 5</text>
  <text x="560" y="258" fill="#64748b" font-size="11" text-anchor="middle">Job 6</text>
  <!-- legend -->
  <line x1="60" y1="20" x2="90" y2="20" stroke="#38bdf8" stroke-width="2.5"/>
  <text x="95" y="24" fill="#94a3b8" font-size="11">Central estimate</text>
  <line x1="220" y1="20" x2="250" y2="20" stroke="#38bdf8" stroke-width="1.5" stroke-dasharray="5,3"/>
  <text x="255" y="24" fill="#94a3b8" font-size="11">Upper bound (walltime)</text>
  <line x1="420" y1="20" x2="450" y2="20" stroke="#38bdf8" stroke-width="1.5" stroke-dasharray="5,3"/>
  <text x="455" y="24" fill="#94a3b8" font-size="11">Lower bound</text>
  <!-- y-axis label -->
  <text x="20" y="145" fill="#64748b" font-size="11" transform="rotate(-90,20,145)">Runtime (s)</text>
  <!-- title -->
  <text x="340" y="15" fill="#38bdf8" font-size="12" text-anchor="middle" font-weight="600" letter-spacing="0.05em">THREE-POINT RUNTIME ENVELOPE</text>
</svg>
<p style="text-align:center;color:#64748b;font-size:0.75rem;margin:0.5rem 0 0">Each job receives a lower bound, central estimate, and upper bound — the envelope adapts to each task type.</p>
</div>

We formulate runtime prediction as three values:

- **Lower bound** — the optimistic estimate (5th percentile)
- **Central estimate** — expected execution time
- **Upper bound** — the conservative safe limit (95th percentile)

The upper bound replaces the static walltime. The central estimate is what you would use for scheduling cost. The lower bound can flag jobs that finish suspiciously fast.

All three come from one model, trained jointly:

```
ŷ_central = f_c(h)
ŷ_lower   = ŷ_central − softplus(f_l(h))
ŷ_upper   = ŷ_central + softplus(f_u(h))
```

The softplus ensures the bounds never cross. The model is a residual neural network trained on pre-execution features: task identity, input size, CPU count, processor frequency, memory, and execution site — nothing measured after the job starts.

## The real challenge: uneven history

Not every task type has 178 executions. Some have one. Some have none.

A single model trained on the dataset's majority will behave poorly on the minority. The proposed framework handles this explicitly, by routing each incoming job to one of four strategies based on how much training history the task type has:

| Historical support | Strategy |
|---|---|
| ≥ 10 executions (well-supported) | Learned model with task-specific runtime reference |
| 1–9 executions (sparse) | Empirical quantiles from available observations |
| 0 executions (zero-shot) | Learned model with hierarchical fallback reference |
| Stable behavior in training (rule-based) | Empirical quantiles |

Every path produces the same lower–central–upper representation, so the rest of the system does not need to know which path was taken.

<div style="margin:2rem 0;padding:1.5rem;background:rgba(255,255,255,0.03);border:1px solid rgba(255,255,255,0.08);border-radius:1rem;overflow:hidden">
<svg viewBox="0 0 680 280" xmlns="http://www.w3.org/2000/svg" style="width:100%;display:block;font-family:inherit">
  <!-- title -->
  <text x="340" y="20" fill="#38bdf8" font-size="12" text-anchor="middle" font-weight="600" letter-spacing="0.05em">HISTORY-AWARE ESTIMATION STRATEGY</text>
  <!-- start node -->
  <rect x="270" y="35" width="140" height="32" rx="6" fill="rgba(56,189,248,0.12)" stroke="#38bdf8" stroke-width="1.2"/>
  <text x="340" y="56" fill="#e2e8f0" font-size="12" text-anchor="middle">Incoming Job</text>
  <!-- arrow down -->
  <line x1="340" y1="67" x2="340" y2="88" stroke="rgba(255,255,255,0.2)" stroke-width="1.2" marker-end="url(#arr)"/>
  <!-- decision diamond -->
  <polygon points="340,88 405,112 340,136 275,112" fill="rgba(56,189,248,0.06)" stroke="#38bdf8" stroke-width="1.2"/>
  <text x="340" y="108" fill="#94a3b8" font-size="10.5" text-anchor="middle">executions</text>
  <text x="340" y="121" fill="#94a3b8" font-size="10.5" text-anchor="middle">available?</text>
  <!-- ≥10 branch right -->
  <line x1="405" y1="112" x2="570" y2="112" stroke="rgba(255,255,255,0.2)" stroke-width="1.2" marker-end="url(#arr)"/>
  <text x="487" y="107" fill="#64748b" font-size="10" text-anchor="middle">≥ 10</text>
  <rect x="570" y="96" width="100" height="32" rx="6" fill="rgba(34,197,94,0.08)" stroke="rgba(34,197,94,0.4)" stroke-width="1.2"/>
  <text x="620" y="114" fill="#86efac" font-size="10.5" text-anchor="middle">Learned Model</text>
  <text x="620" y="125" fill="#64748b" font-size="9.5" text-anchor="middle">(task-specific ref)</text>
  <!-- 1-9 branch left -->
  <line x1="275" y1="112" x2="110" y2="112" stroke="rgba(255,255,255,0.2)" stroke-width="1.2" marker-end="url(#arr)"/>
  <text x="192" y="107" fill="#64748b" font-size="10" text-anchor="middle">1–9</text>
  <rect x="10" y="96" width="100" height="32" rx="6" fill="rgba(251,191,36,0.08)" stroke="rgba(251,191,36,0.4)" stroke-width="1.2"/>
  <text x="60" y="114" fill="#fde68a" font-size="10.5" text-anchor="middle">Empirical</text>
  <text x="60" y="125" fill="#64748b" font-size="9.5" text-anchor="middle">Quantiles (sparse)</text>
  <!-- 0 branch down -->
  <line x1="340" y1="136" x2="340" y2="162" stroke="rgba(255,255,255,0.2)" stroke-width="1.2" marker-end="url(#arr)"/>
  <text x="350" y="155" fill="#64748b" font-size="10" text-anchor="start">0</text>
  <!-- zero-shot decision -->
  <polygon points="340,162 405,186 340,210 275,186" fill="rgba(56,189,248,0.06)" stroke="#38bdf8" stroke-width="1.2"/>
  <text x="340" y="182" fill="#94a3b8" font-size="10.5" text-anchor="middle">family</text>
  <text x="340" y="195" fill="#94a3b8" font-size="10.5" text-anchor="middle">match?</text>
  <!-- yes right -->
  <line x1="405" y1="186" x2="520" y2="186" stroke="rgba(255,255,255,0.2)" stroke-width="1.2" marker-end="url(#arr)"/>
  <text x="462" y="181" fill="#64748b" font-size="10" text-anchor="middle">yes</text>
  <rect x="520" y="170" width="110" height="32" rx="6" fill="rgba(34,197,94,0.08)" stroke="rgba(34,197,94,0.4)" stroke-width="1.2"/>
  <text x="575" y="188" fill="#86efac" font-size="10.5" text-anchor="middle">Learned Model</text>
  <text x="575" y="199" fill="#64748b" font-size="9.5" text-anchor="middle">(family/config ref)</text>
  <!-- no down -->
  <line x1="340" y1="210" x2="340" y2="236" stroke="rgba(255,255,255,0.2)" stroke-width="1.2" marker-end="url(#arr)"/>
  <text x="350" y="228" fill="#64748b" font-size="10">no</text>
  <rect x="270" y="236" width="140" height="32" rx="6" fill="rgba(34,197,94,0.08)" stroke="rgba(34,197,94,0.4)" stroke-width="1.2"/>
  <text x="340" y="254" fill="#86efac" font-size="10.5" text-anchor="middle">Global Median Fallback</text>
  <!-- arrow marker -->
  <defs>
    <marker id="arr" markerWidth="7" markerHeight="7" refX="6" refY="3.5" orient="auto">
      <polygon points="0,0 7,3.5 0,7" fill="rgba(255,255,255,0.2)"/>
    </marker>
  </defs>
</svg>
<p style="text-align:center;color:#64748b;font-size:0.75rem;margin:0.5rem 0 0">The strategy router adapts estimation based on available task history — zero-shot tasks receive hierarchical fallback references.</p>
</div>

### Zero-shot: when a task has never been seen before

Zero-shot is the hardest case. There is no task-specific history to draw on.

The solution is a hierarchical runtime reference. When no task-level history exists, the model falls back to a family-level reference derived from the task name prefix. If that is also missing, it matches the incoming job to a configuration bucket defined by input size, CPU count, and execution site. If nothing matches, it uses the global training median.

This gives the model a runtime-scale prior without requiring a single previous execution of the incoming task type.

## Integration into Pegasus-WMS

The framework sits between the Pegasus Planner and HTCondor:

```
Workflow Specification
       ↓
Pegasus Planner → .dag + .sub files
       ↓
Runtime Predictor (ML model)
       ↓
Patch .sub with +MaxRuntime
       ↓
HTCondor → Compute Cluster
```

No changes to the workflow specification. No changes to the execution path. The predicted upper bound is written into the job description before submission, where HTCondor can use it for scheduling.

## Results

The evaluation used 262,401 jobs across 223 task types, with a held-out temporal test set from a different execution period.

**Zero-shot tasks** (37 task types, 6,976 jobs — never seen in training):

| Method | Coverage | Under-prediction | Winkler |
|---|---|---|---|
| Task Median | 52.5% | 28.9% | 12091.5 |
| Random Forest | 72.3% | 12.9% | 2659.9 |
| MOGB | 79.5% | 9.3% | 2307.1 |
| **Ours** | **85.8%** | **4.3%** | **1911.8** |

<div style="margin:2rem 0;padding:1.5rem;background:rgba(255,255,255,0.03);border:1px solid rgba(255,255,255,0.08);border-radius:1rem;overflow:hidden">
<svg viewBox="0 0 680 260" xmlns="http://www.w3.org/2000/svg" style="width:100%;display:block;font-family:inherit">
  <!-- title -->
  <text x="340" y="20" fill="#38bdf8" font-size="12" text-anchor="middle" font-weight="600" letter-spacing="0.05em">ZERO-SHOT UNDER-PREDICTION RATE (%)</text>
  <!-- axes -->
  <line x1="150" y1="40" x2="150" y2="220" stroke="rgba(255,255,255,0.15)" stroke-width="1"/>
  <line x1="150" y1="220" x2="640" y2="220" stroke="rgba(255,255,255,0.15)" stroke-width="1"/>
  <!-- grid lines -->
  <line x1="150" y1="160" x2="640" y2="160" stroke="rgba(255,255,255,0.05)" stroke-width="1"/>
  <line x1="150" y1="100" x2="640" y2="100" stroke="rgba(255,255,255,0.05)" stroke-width="1"/>
  <line x1="150" y1="40" x2="640" y2="40" stroke="rgba(255,255,255,0.05)" stroke-width="1"/>
  <!-- y-axis labels -->
  <text x="140" y="224" fill="#64748b" font-size="10" text-anchor="end">0%</text>
  <text x="140" y="164" fill="#64748b" font-size="10" text-anchor="end">10%</text>
  <text x="140" y="104" fill="#64748b" font-size="10" text-anchor="end">20%</text>
  <text x="140" y="44" fill="#64748b" font-size="10" text-anchor="end">30%</text>
  <!-- bar: Task Median — 28.9% → (28.9/30)*180 = 173.4 -->
  <rect x="170" y="47" width="80" height="173" rx="4" fill="rgba(148,163,184,0.25)" stroke="rgba(148,163,184,0.4)" stroke-width="1"/>
  <text x="210" y="43" fill="#94a3b8" font-size="11" text-anchor="middle">28.9%</text>
  <text x="210" y="238" fill="#64748b" font-size="10" text-anchor="middle">Task Median</text>
  <!-- bar: Random Forest — 12.9% → (12.9/30)*180 = 77.4 -->
  <rect x="290" y="143" width="80" height="77" rx="4" fill="rgba(148,163,184,0.25)" stroke="rgba(148,163,184,0.4)" stroke-width="1"/>
  <text x="330" y="139" fill="#94a3b8" font-size="11" text-anchor="middle">12.9%</text>
  <text x="330" y="238" fill="#64748b" font-size="10" text-anchor="middle">Random Forest</text>
  <!-- bar: MOGB — 9.3% → 55.8 -->
  <rect x="410" y="164" width="80" height="56" rx="4" fill="rgba(148,163,184,0.25)" stroke="rgba(148,163,184,0.4)" stroke-width="1"/>
  <text x="450" y="160" fill="#94a3b8" font-size="11" text-anchor="middle">9.3%</text>
  <text x="450" y="238" fill="#64748b" font-size="10" text-anchor="middle">MOGB</text>
  <!-- bar: Ours — 4.3% → 25.8 -->
  <rect x="530" y="194" width="80" height="26" rx="4" fill="rgba(56,189,248,0.3)" stroke="rgba(56,189,248,0.6)" stroke-width="1.5"/>
  <text x="570" y="190" fill="#38bdf8" font-size="11" text-anchor="middle" font-weight="700">4.3%</text>
  <text x="570" y="238" fill="#38bdf8" font-size="10" text-anchor="middle" font-weight="600">Ours</text>
  <!-- lower is better label -->
  <text x="640" y="230" fill="#64748b" font-size="9" text-anchor="end">↓ lower is better</text>
</svg>
<p style="text-align:center;color:#64748b;font-size:0.75rem;margin:0.5rem 0 0">Under-prediction rate on zero-shot tasks (37 task types, 6,976 jobs). The proposed method reduces under-prediction by 6.7× vs. Task Median.</p>
</div>

The proposed approach reduces under-prediction from 28.9% (Task Median) to 4.3% — a 6.7× reduction — while achieving the tightest intervals of any method tested.

**Against a static 3600 s walltime** (the conventional default):

The static limit virtually eliminates under-prediction (0.11%) but at a mean excess of 3440 seconds per job. The predicted upper bound brings under-prediction to 0.77% while reducing mean excess to 1807 seconds — a **47.5% reduction** in wasted allocation, with near-equivalent safety.

**Sparse tasks** (1–9 training executions) show a different advantage: the empirical strategy produces an IWR of 0.667 versus 1.390 for Task Median, cutting interval width nearly in half while maintaining 93.4% coverage.

## What this means for workflow scheduling

The runtime envelope exposes a trade-off the cluster operator can act on:

- Use the **central estimate** for cost accounting and fair scheduling.
- Use the **upper bound** as the walltime limit — it behaves like a dynamic, job-specific 3600 s, but calibrated to each job rather than uniform across all of them.
- Use the **lower bound** to flag anomalously fast completions worth investigating.

For Pegasus users, this is transparent. The planner generates the same workflow. The predictor patches the submit files. The cluster sees a tighter, more accurate limit.

## The broader lesson

Execution time in production HPC is not well-described by a single number, and the amount of available history differs dramatically across task types. Both of these facts have to be addressed together — a model that is accurate on the majority is still unreliable on the tail, and a model that covers the tail with a fixed margin wastes allocation on the majority.

The combination of a runtime envelope and a history-aware estimation strategy handles both. The representation is consistent across all task types. The estimation adapts to what is actually known.

---

*This work is part of the Pegasus AI research program, funded by the National Science Foundation under award [2513101](https://www.nsf.gov/awardsearch/showAward?AWD_ID=2513101).*
