## ✅ Accuracy

Question it answers:

“How often does this router get the answer right?”

Plain meaning:

If there are 100 questions

and answers are correct for 74

accuracy = 74%

No cost, no speed — just correctness.

## 💰 Cost per 1K queries

Question it answers:

“How expensive is this router?”

Plain meaning:

Add up money spent across questions

Normalize it to 1000 questions

Lower = better.

This punishes routers that:

always use very expensive models.

## ⏱️ Latency score

Question it answers:

“How slow is the router’s decision-making?”

Important:

This is not model response time

This is router thinking time

If the router:

is simple → fast → high score

is complex → slow → low score

## 🎯 Optimal accuracy (generalization score)

Question it answers:

“How close is this router to always using the best model?”

Plain meaning:

Imagine the best single LLM you have

Compare router accuracy to that model

If:

best model gets 90%

router gets 81%

score = 81 / 90 = 90%

This shows:

how much quality the router sacrifices.

## 💸 Optimal cost score

Question it answers:

“Is the router wasting money when a cheaper correct model exists?”

Plain meaning:

For each question:

find the cheapest model that answered correctly

compare it with what the router chose

If router often chooses:

more expensive models unnecessarily → bad score

## 🧠 Optimal selection score

Question it answers:

“Did the router choose the exact cheapest correct model?”

This is stricter than optimal cost.

Optimal cost allows “almost cheapest”

Optimal selection demands perfect choice

Good for measuring precision of decision-making.

## 🔒 Robustness score

Question it answers:

“Does the router behave consistently when the question changes slightly?”

Plain meaning:

Same question, slightly reworded

Did the router still pick the same model?

If tiny changes cause big decision flips:

router is unstable

score goes down

## 🏆 Arena score (overall score)

Question it answers:

“Is this router good overall?”

This score combines:

accuracy

cost efficiency

Meaning:

High accuracy but insanely expensive → not great

Very cheap but wrong often → not great

Balanced → high arena score

This is what you sort the leaderboard by.

## 📊 Visualizations (What users see)
### Difficulty Bars

Show:

performance on easy / medium / hard questions

Simple meaning:

“Does this router collapse on hard problems?”

### Spider (Radar) Graph

Each spoke = one metric:

accuracy

cost efficiency

robustness

latency

optimality

Big balanced shape = good router
Weird spiky shape = trade-offs
