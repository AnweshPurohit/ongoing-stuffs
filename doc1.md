## What to Do Now (Dataset → Leaderboard)
Step 0 — Mental model (important)
From this point on:
Parquet files = ground truth universe
Everything else (metrics, charts, leaderboard) is derived from them

Do not change datasets while building logic

Lock them and move forward.

## Step 1 — Inspect & Understand Each Dataset (mandatory)

Before coding anything fancy, you must map each file to its purpose.

### 1.1 full_0000.parquet → Core evaluation dataset

This is your main benchmark.

It should contain (conceptually):

query / prompt

ground truth

category

difficulty (maybe)

model outputs or model correctness info

 This dataset powers:

Accuracy

Cost per 1K

Latency score

Optimal accuracy

Optimal cost

Optimal selection

Arena score

Difficulty bars

Spider graph

This is 80% of ArbitRoute.

### 1.2 sub_10.parquet → Lightweight / fast evaluation split

This is usually:

a subset of full_0000

meant for:

quick testing

debugging routers

CI / sanity checks

👉 Use this for:

Router dry-runs

Fast leaderboard previews

Developer mode

Do not mix it into final rankings.

### 1.3 Robustness_0000.parquet → Perturbation dataset

This is special.

It likely contains:

original query ID

perturbed variants

maybe mapping to original query

👉 This dataset powers:

Robustness (stability) score ONLY

Do not compute other metrics from it.

## Step 2 — Define Your Internal Canonical Schema

Even if parquet columns are messy, you must normalize them internally.

Create one internal record format:
```
{
  "query_id": "...",
  "query": "...",
  "ground_truth": "...",
  "category": "...",
  "difficulty": "easy|medium|hard",
  "model_name": "...",
  "model_answer": "...",
  "correct": true,
  "tokens": 123,
  "cost": 0.0021,
  "latency_ms": 812
}
```
Why this matters

Metrics become simple

Visualization becomes trivial

You can swap datasets later without rewriting logic

👉 First task in code: parquet → canonical rows

## Step 3 — Build the Evaluation Log Generator

This is the engine of ArbitRoute.

### 3.1 One router run = one log table

For each router:

Iterate over full_0000.parquet

For each query:

Router selects a model

Pull that model’s result

Log everything

Result:

One evaluation log per router

Think of it as:

router_name → evaluation_logs.parquet
### 3.2 Important rule

🚫 Do not recompute model answers
✅ Use the existing model outputs in the dataset

ArbitRoute benchmarks routing decisions, not generation.

## Step 4 — Implement Metrics as Pure Functions

This is where your leaderboard power comes from.

Each metric should look like:
```
def compute_accuracy(logs): ...
def compute_cost_per_1k(logs): ...
def compute_latency_score(logs): ...
def compute_optimal_cost(logs, all_models): ...
```
Metrics to implement (checklist)

From full_0000.parquet:

Accuracy

Cost per 1K queries

Latency overhead score

Generalization (optimal accuracy) score

Optimal cost score

Optimal selection score

Arena score

From Robustness_0000.parquet:

Robustness score only

⚠️ No metric should:

read UI state

depend on other metrics

mutate data

## Step 5 — Precompute Leaderboard Rows

For each router:
```
{
  "router_name": "MyRouterV1",
  "accuracy": 74.0,
  "arena_score": 68.2,
  "cost_per_1k": 0.56,
  "latency_score": 43.0,
  "optimal_accuracy": 88.7,
  "optimal_cost": 64.9,
  "optimal_selection": 63.8,
  "robustness": 91.2
}
```
Store this in:

PostgreSQL (recommended)

or a cached parquet/JSON

👉 This is what your leaderboard page reads from

## Step 6 — Power the Visualizations

Now the UI becomes easy.

### 6.1 Difficulty Bars

Source:

full_0000.parquet

Process:

group queries by difficulty

compute accuracy per bucket

one bar per router per difficulty

### 6.2 Spider (Radar) Graph

Source:

precomputed leaderboard metrics

Process:

normalize metrics to [0, 100]

each axis = one metric

plot polygon per router

No extra math needed here.

## Step 7 — Robustness Evaluation Pipeline

This is separate from the main run.

Process:

For each original query:

Compare model chosen for:

original query

perturbed variants

Count changes

Output:

One scalar robustness score per router

Cache it — it’s expensive.

## Step 8 — Leaderboard Logic
Ranking rule

Sort routers by:

Arena score (primary)

Accuracy (tie-break)

Cost (secondary tie-break)

Display:

top routers

sortable columns

compare mode (2–3 routers)

## Step 9 — Developer Workflow (very important)
Use sub_10.parquet for:

router development

debugging

fast iteration

Use full_0000.parquet for:

official runs

leaderboard updates

Never mix them.

## Step 10 — Final Project Structure (suggested)
```
arbitroute/
│
├── data/
│   ├── full_0000.parquet
│   ├── sub_10.parquet
│   └── Robustness_0000.parquet
│
├── routers/
│   ├── baseline.py
│   ├── cost_aware.py
│   └── learned_router.py
│
├── evaluation/
│   ├── runner.py
│   ├── metrics.py
│   └── robustness.py
│
├── leaderboard/
│   ├── scores.parquet
│   └── api.py
│
└── frontend/
```
🔑 What you should do today

If I had to compress this into today’s TODO list:

Load all three parquet files

Print schemas and sample rows

Normalize into one internal format

Implement accuracy + cost first

Generate your first leaderboard row

Once that works, everything else is incremental.

If you want next, I can:

walk through one parquet file column-by-column

help you write the first evaluation script

design the router baseline set

help you define submission rules
