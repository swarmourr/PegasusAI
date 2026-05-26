---
title: "ML-Driven Resource Prediction in Scientific Workflows"
date: "2026-05-10"
author: "Pegasus AI Team"
description: "How machine learning models predict resource requirements and optimize execution plans for complex scientific workflows."
---

# ML-Driven Resource Prediction

One of the most persistent challenges in scientific computing is answering a deceptively simple question: **how much compute does this job actually need?**

Under-provision and the job fails or runs agonizingly slowly. Over-provision and you waste expensive cluster time. Pegasus AI tackles this with trained ML models that predict resource requirements before a workflow even starts.

## The Training Data Problem

ML models are only as good as their training data. Pegasus WMS has been running real scientific workflows for over two decades, generating a rich corpus of runtime statistics:

- Wall-clock time per task type
- Peak memory usage
- CPU utilization curves
- I/O bandwidth profiles
- Failure rates per resource

This historical data is the foundation of our prediction pipeline.

## Model Architecture

We use a **gradient-boosted ensemble** approach trained on workflow task features:

```python
features = [
    'task_type',
    'input_data_size_gb',
    'num_input_files',
    'cluster_type',
    'historical_avg_runtime'
]

model = XGBRegressor(n_estimators=300, max_depth=6)
model.fit(X_train, y_runtime)
```

For memory prediction we use a separate model, since memory and runtime don't always correlate.

## Results

In our initial benchmarks on real genomics workflows:

| Metric | Baseline (User Estimate) | Pegasus AI Prediction |
|--------|--------------------------|----------------------|
| Over-provisioning | 42% of jobs | 11% of jobs |
| Job failures | 8.3% | 2.1% |
| Avg. wall-clock efficiency | 61% | 88% |

## What's Next

We are expanding prediction to cover **multi-task dependencies** — where the resource needs of downstream tasks depend on the outputs of upstream ones. This requires graph-aware ML, which we cover in a future post.

---

*Questions? Reach out at pegasusai@isi.edu*
