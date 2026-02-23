## 🧠 ArbitRoute — Complete Roadmap (with Evaluation Metrics)
What ArbitRoute is (one-line)

ArbitRoute is a benchmarking platform that evaluates LLM routers—systems that decide which LLM to use per query—across accuracy, cost, latency, robustness, and optimality.

You are benchmarking decisions, not models.

## 1️⃣ Core Abstraction (Everything Depends on This)
1.1 The basic evaluation loop

For every query:

Router receives a query

Router selects a model

Selected model generates an answer

ArbitRoute measures:

correctness

token usage → cost

latency

routing behavior vs alternatives

All metrics are aggregations of this loop.

1.2 First-class entities

Design these explicitly:

Entity	Meaning
Query	Prompt + ground truth
Model	LLM + price + latency
Router	Policy choosing a model
Run	Router × Dataset × Model pool
Log	One query’s full execution record

## 2️⃣ Dataset Layer (Ground Truth Source)
2.1 Dataset schema (canonical)

Use JSONL:

{
  "id": "q_0142",
  "query": "Who wrote The Republic?",
  "ground_truth": "Plato",
  "category": "philosophy"
}

Important:

Dataset contains no model outputs

Dataset is immutable once evaluation starts

2.2 Difficulty computation (offline preprocessing)

Difficulty is not subjective — it’s empirical.

Procedure:

Run all models once on each query

Count how many answer correctly

% models correct	Difficulty
≥ 80%	Easy
30–80%	Medium
< 30%	Hard

Store difficulty in metadata.

➡ Used later for Difficulty Bars

## 3️⃣ Model Pool Layer
3.1 Model registry

Every model must declare:

{
  "model_name": "gpt-4",
  "price_per_1k_tokens": 0.06,
  "avg_latency_ms": 900
}

This registry is frozen per evaluation run to ensure fairness.

3.2 Unified model interface

All models expose:

(answer, tokens_used, latency_ms)

This enables:

cost calculation

latency metrics

optimal cost analysis

## 4️⃣ Router Interface (What Users Submit)
4.1 Router contract

Every router implements:

select_model(query, metadata) → model_name

Metadata may include:

category

difficulty

cost budget

latency budget

Routers cannot see ground truth.

## 5️⃣ Evaluation Engine (Data Collection Phase)
5.1 Execution loop

For each router and query:

model = router.select_model(query)
answer, tokens, latency = model.generate(query)
correct = grade(answer, ground_truth)
5.2 Per-query log record

This is the most important artifact:

{
  "query_id": "q_0142",
  "selected_model": "gpt-4",
  "correct": true,
  "tokens": 42,
  "cost": 0.0025,
  "latency_ms": 870
}

Every metric is computed only from these logs.

## 6️⃣ Evaluation Metrics (Explicit & Mapped)

Now the key part: what ArbitRoute actually measures.

🔹 1. Accuracy

What it measures: raw answer quality

Formula:

accuracy = (# correct answers / total queries) × 100

Interpretation:

Ignores cost and latency

“How often does this router get the answer right?”

🔹 2. Cost per 1K queries

What it measures: monetary efficiency

How:

Sum token cost over all queries

Normalize to 1000 queries

Interpretation:

Lower = cheaper routing strategy

🔹 3. Latency overhead score

What it measures: router decision speed (not model generation)

Formula:

score = 1 / (L_router − 10)

Where:

L_router = average routing decision time (ms)

10 ms = assumed baseline

Interpretation:

Higher score → faster router logic

🔹 4. Generalization (Optimal Accuracy) Score

Question answered:
“How close is the router to always using the best possible model?”

Formula:

score = a_achieved / a_max

Where:

a_achieved = router accuracy

a_max = accuracy of the single best model in pool

Interpretation:

1.0 → matches best model quality

< 1.0 → quality sacrificed for cost/latency

🔹 5. Optimal Cost Score

Question answered:
“How much extra money does the router spend vs the cheapest correct choice?”

For each query:

score_i = cost_opt / cost_actual

Average over all valid queries.

Interpretation:

1.0 → always cheapest correct model

0.5 → spending ~2× more than necessary

🔹 6. Optimal Selection Score

Stricter than optimal cost

Formula:

score = (# times router chose cheapest correct model) / (# valid queries)

Interpretation:

Measures exact optimal hits, not near misses

🔹 7. Robustness (Stability) Score

What it measures: routing consistency under noise

Procedure:

Generate perturbed queries (paraphrase, typos)

Compare selected model vs original

Formula:

score = 1 − (# changed selections / total selections)

Interpretation:

High = stable routing logic

Low = brittle router

🔹 8. Arena Score (Overall Score)

Purpose: trade off accuracy vs cost

Steps:

Normalize cost in log space → cheapness score C_r

Combine with accuracy A_r

Formula:

S = (1 + β) A_r C_r / (β A_r + C_r)

Where:

β = 0.1 (slight preference for cost)

Interpretation:

High only if router is both accurate and cost-efficient

## 7️⃣ Visualization Layer
7.1 Difficulty Bars

How computed:

Group queries by difficulty

Compute accuracy per group

What it shows:

How router performance degrades from Easy → Hard

7.2 Spider (Radar) Graph

Axes (all normalized 0–100):

Accuracy

Arena score

Optimal cost

Optimal selection

Robustness

Latency score

What it shows:

Trade-offs at a glance

Balanced vs skewed routers

## 8️⃣ Backend Architecture
Recommended stack

API: FastAPI

Workers: Celery / Ray

Storage:

PostgreSQL → metadata

Parquet / JSONL → logs

Cache: Redis

## 9️⃣ Fairness & Reproducibility

Fixed decoding parameters

Frozen model prices

Deterministic seeds

No adaptive retrials

This is critical for leaderboard credibility.

## 🔟 Development Order (Very Important)

Do NOT start with UI

Correct order:

Dataset schema

Model interface

Router interface

Logging

Metric computation

Visualizations

Public leaderboard

🧩 Final mental model

Think of ArbitRoute as:

A physics lab for LLM routing policies
Same environment, same models — only decisions differ.

If you want next, I can:

design the database schema

sketch a 2-week MVP

write metric computation pseudocode

help you frame the paper / README
