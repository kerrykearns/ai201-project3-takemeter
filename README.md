# ai201-project3-takemeter

A fine-tuned text classifier that evaluates discourse quality in r/soccer comments, distinguishing between tactical analysis, hot takes, and emotional reactions.

---

## Community Choice

I chose **r/soccer** (reddit.com/r/soccer), an 8.7-million-member community dedicated
to global football discussion. It is an excellent fit for a classification task because
its discourse spans a genuinely wide spectrum in a way that is meaningful to the
community itself, regular participants actively distinguish between substantive tactical
arguments and unsupported opinions. High-quality analysis gets upvoted and praised;
unsupported hot takes get called out; raw reactions fill match threads. The distinctions
I am measuring are ones the community itself cares about, not categories imposed from
outside.

---

## Label Taxonomy

### `analysis`
A structured argument about football using specific, verifiable evidence statistics,
tactical observations, historical comparisons, or concrete match examples to support
a claim. The argument depends on the evidence; removing it would collapse the claim.

**Example 1:** "Rodri's absence from City last season proves he was their single most
important player , their PPDA dropped from 9.8 to 14.2 without him and they lost 7 of
11 league games."

**Example 2:** "Van Dijk's importance to Liverpool is measurable , without him in
2020-21 their goals conceded rate doubled from 0.89 to 1.78 per game and they finished
8th. That's one player accounting for roughly a 0.9 GA difference per match."

---

### `hot_take`
A bold, confident opinion about a player, team, league, or rule stated without specific
supporting evidence. The post asserts a strong claim but does not argue for it with
verifiable facts or data.

**Example 1:** "Mbappe is already better than prime Ronaldo. The guy is 25 and has done
everything. People need to stop coping."

**Example 2:** "The Premier League is the most overrated league in the world. Tactically
it's a joke compared to Serie A."

---

### `reaction`
An immediate emotional response to a specific match event, result, or news , expressing
a feeling in the moment without making a broader argument or general claim about football.

**Example 1:** "WHAT A GOAL. I cannot believe what I just watched. Absolute scenes"

**Example 2:** "That penalty was never in a million years. Robbery."

---

### Decision Rule for Hard Cases
If a post contains a statistic or fact but the claim would still stand without it, label
it `hot_take` not `analysis`. If a post sounds like a general opinion but is tied to a
specific match moment, label it `reaction` not `hot_take`.

---

## Dataset

**Source:** 215 examples manually curated to authentically represent r/soccer discourse
patterns across match threads, discussion posts, and tactical comment sections.

**Labeling process:** Each example was read individually and assigned a label using the
definitions above. Cases that gave genuine pause were documented with notes in the CSV
explaining the decision made.

**Label distribution:**

| Label | Count | Percentage |
|---|---|---|
| reaction | 75 | 34.9% |
| hot_take | 70 | 32.6% |
| analysis | 70 | 32.6% |
| **Total** | **215** | **100%** |

No single label exceeds 70% of the dataset.

**Train / Validation / Test split (70/15/15, stratified):**
- Train: 150 examples
- Validation: 32 examples
- Test: 33 examples

---

### Difficult-to-Label Examples

**Case 1:** *"Zidane was a better manager than Guardiola. Three UCL titles back to back
proves it."*
Could be `analysis` because it cites trophy count. Decided `hot_take` , the trophy
count is decorative. Remove it and the claim is unchanged: "Zidane was a better manager
than Guardiola." Decision rule: evidence must be load-bearing, not decorative.

**Case 2:** *"Football managers matter far less than people think. The squad determines
80% of results."*
The 80% figure sounds like data. Decided `hot_take` , the number is invented opinion
with no source or methodology behind it.

**Case 3:** *"That substitution makes zero sense. Manager has lost the plot."*
Sounds like a general opinion on manager quality. Decided `reaction` , it is tied to a
specific in-game moment, not a general claim about the manager's ability.

**Case 4:** *"The reason Man United have struggled under multiple managers post-Ferguson
is structural: the recruitment process changed from manager-led to committee-led in 2013
and research shows misaligned recruitment authority correlates with trophy droughts."*
Strong claim about a club , could be `hot_take`. Decided `analysis` , the structural
change cited is specific and verifiable, and the argument collapses without it.

**Case 5:** *"Messi's 2021 Ballon d'Or was earned primarily through Copa America
performances , his xG+xA per 90 in that tournament was 1.14, which is historically
elite even by his own standards."*
Could be `hot_take` defending Messi. Decided `analysis` , remove the xG+xA figure and
the claim has no legs. The evidence is load-bearing.

---

## Fine-Tuning Pipeline

**Base model:** `distilbert-base-uncased` (HuggingFace)
**Training platform:** Google Colab (T4 GPU)
**Training libraries:** transformers, datasets, scikit-learn

**Training configuration:**
- Epochs: 3
- Learning rate: 2e-5
- Batch size: 16
- Weight decay: 0.01
- Warmup steps: 50

**Key hyperparameter decision:** I kept the default learning rate of 2e-5 rather than
increasing it. With only 150 training examples, a higher learning rate risks overfitting
quickly , the model would memorize the training set rather than learning generalizable
boundaries between labels. 3 epochs was sufficient for convergence on this dataset size
without overtraining.

---

## Baseline

**Model:** llama-3.3-70b-versatile via Groq API (zero-shot, no task-specific training)

**Prompt used:**
You are a football discourse classifier for r/soccer comments.

Assign each comment to exactly one of the following three categories.
reaction: An immediate emotional response to a specific match event, result, or news.

hot_take: A bold confident opinion stated without specific supporting evidence.

analysis: A structured argument using specific verifiable evidence to support a claim.
Decision rules:

If a post contains a stat but the claim stands without it, label it hot_take not analysis.
If a post sounds like opinion but is tied to a specific match moment, label it reaction.

Respond with ONLY one word: reaction, hot_take, or analysis.

**How results were collected:** The prompt was run once per test example using the
Groq API at temperature=0. All 33 responses were parseable.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy | Test Set Size |
|---|---|---|
| Zero-shot baseline (llama-3.3-70b-versatile) | **0.970** | 33 |
| Fine-tuned DistilBERT | **0.697** | 33 |

The baseline outperformed the fine-tuned model by 27.3 percentage points.

---

### Per-Class Metrics

**Fine-tuned DistilBERT:**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 1.00 | 0.82 | 0.90 | 11 |
| hot_take | 0.50 | 1.00 | 0.67 | 10 |
| reaction | 1.00 | 0.33 | 0.50 | 12 |
| **macro avg** | **0.83** | **0.72** | **0.69** | **33** |

**Zero-shot baseline (llama-3.3-70b-versatile):**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 1.00 | 1.00 | 1.00 | 11 |
| hot_take | 0.91 | 1.00 | 0.95 | 10 |
| reaction | 1.00 | 0.92 | 0.96 | 12 |
| **macro avg** | **0.97** | **0.97** | **0.97** | **33** |

---

### Confusion Matrix (Fine-Tuned Model)

Rows = true label. Columns = predicted label.

|  | predicted: analysis | predicted: hot_take | predicted: reaction |
|---|---|---|---|
| **true: analysis** | 9 | 2 | 0 |
| **true: hot_take** | 0 | 10 | 0 |
| **true: reaction** | 0 | 8 | 4 |

**Key pattern:** Every wrong prediction was classified as `hot_take`. The model never
predicted `analysis` or `reaction` when it was wrong , it always defaulted to `hot_take`.
This is a complete collapse onto one label under uncertainty.

---

### Wrong Prediction Analysis

**Error 1:**
> *"High defensive lines work in the Premier League because pitch sizes at most grounds
> are 105x68 minimum , that's the full UEFA standard. Smaller pitches in lower divisions
> are why pressing doesn't transfer as easily between tiers."*
> True: `analysis` → Predicted: `hot_take` (confidence: 0.37)

This is a genuine analysis post , it makes a specific structural argument about pitch
dimensions and their tactical consequences. The model likely failed because the post
opens with a confident declarative statement ("High defensive lines work...") which
superficially resembles hot_take language. The evidence comes later in the sentence.
DistilBERT's attention may have weighted the opening tokens too heavily. This is a
labeling boundary problem , the post is unambiguously analysis, but its opening reads
like assertion.

**Error 2:**
> *"Last minute winner away from home. The scenes!!! Unreal."*
> True: `reaction` → Predicted: `hot_take` (confidence: 0.36)

This is a clear reaction , pure emotional response to a match event. The model failed
because it is a short post with no obvious emotional markers like all-caps or emojis
that the model likely associated with `reaction`. "The scenes" and "Unreal" are
enthusiastic but not structurally different from the assertive language in hot_take
posts. The model has not learned that brevity plus exclamation can signal reaction , it
needed more diverse short reaction examples to learn this boundary.

**Error 3:**
> *"Draw at home to bottom of the table. Unacceptable."*
> True: `reaction` → Predicted: `hot_take` (confidence: 0.39)

"Unacceptable" is a strong opinion word that appears frequently in hot_take posts. The
model has correctly learned that hot_take uses assertive language , but it has
overgeneralized that pattern to include short emotional reactions that use a single
strong word. The fix would be more training examples of short reaction posts that use
opinion-sounding words but are clearly tied to a specific match event.

---

### Sample Classifications

| Post | True Label | Predicted | Confidence |
|---|---|---|---|
| "Rodri's absence from City last season proves he was their single most important player , their PPDA dropped from 9.8 to 14.2 without him and they lost 7 of 11 league games." | analysis | analysis | 0.91 |
| "Messi is the greatest of all time and the debate is over. There is no debate." | hot_take | hot_take | 0.88 |
| "FULL TIME. WE DID IT. UNBELIEVABLE SCENES." | reaction | reaction | 0.95 |
| "Draw at home to bottom of the table. Unacceptable." | reaction | hot_take | 0.39 |
| "High defensive lines work in the Premier League because pitch sizes at most grounds are 105x68 minimum." | analysis | hot_take | 0.37 |

**Why the first prediction is reasonable:** The Rodri post opens with a specific metric
(PPDA dropping from 9.8 to 14.2) and a concrete match record (7 of 11 games lost).
The model correctly identified this as `analysis` with high confidence because the
statistical evidence is front-loaded and clearly load-bearing , the claim cannot stand
without it.

---

### Reflection: What the Model Learned vs. What I Intended

I intended the model to learn the **structural distinction** between labels , whether a
post makes an evidenced argument, asserts without evidence, or reacts emotionally to an
event. What the model actually learned was **surface-level vocabulary signals** , stats
and tactical terms signal analysis, all-caps and emojis signal reaction, and everything
else defaults to hot_take.

This explains the failure pattern exactly. When a reaction post lacks all-caps or emojis
(like "Draw at home to bottom of the table. Unacceptable."), the model has no surface
signal to grab onto and collapses to hot_take. When an analysis post opens with a
confident declarative statement before presenting its evidence, the model reads the
opening tokens as hot_take language and never recovers.

The model did not learn the **functional role of evidence** in the argument , it learned
that certain words (PPDA, xG, percentage) predict analysis. This is a shallower
representation than I intended. With 150 training examples and a model that has no
prior football knowledge, this was probably inevitable. Fixing it would require either
significantly more training data (500+) or examples specifically designed to stress-test
the boundary , reaction posts that use opinion words, analysis posts that open with
assertions before presenting data.

The high baseline (97%) compared to the fine-tuned model (70%) also reveals something
important: a large pre-trained LLM understands the **communicative intent** of these
posts from pre-training on internet text. DistilBERT, starting from general language
understanding and trained on only 150 examples, learned a rougher approximation of that
intent.

---

## Spec Reflection

**One way the spec helped:** The spec's insistence on writing a decision rule for the
hardest edge case before annotating any data was the most valuable constraint in the
project. Being forced to define exactly when a stat counts as analysis vs. decoration
(the "remove the evidence and see if the claim collapses" test) made annotation
consistent across 215 examples and gave me a concrete framework to apply to wrong
predictions in the evaluation.

**One way implementation diverged:** The spec assumes the fine-tuned model will
outperform the zero-shot baseline, and frames the baseline as a floor to beat. In
practice the opposite happened , the baseline was nearly perfect and the fine-tuned
model underperformed it significantly. This is because the linguistic signals in the
dataset were too surface-level and consistent for a 70-billion parameter model to miss,
while DistilBERT with 150 training examples could only learn a rough approximation.
The divergence is itself informative: it reveals that the task as designed is easier for
a large pre-trained model than for a small fine-tuned one.


## AI Usage

**Instance 1: Label stress-testing:** Before annotating my dataset, I directed Claude
to generate 10 posts that sat at the boundary between `hot_take` and `analysis` using
my label definitions from planning.md as input. Claude produced several posts that
cited a single statistic inside an otherwise assertive claim , for example, "Bellingham
is the best midfielder in the world right now , his numbers this season are insane, 23
goals and 12 assists from midfield." I could not immediately classify this cleanly,
which revealed that my original definition did not specify what counted as genuine
evidence vs. decorative evidence. I tightened my decision rule as a result: evidence
only counts toward `analysis` if removing it would collapse the claim. I did not change
this rule after annotation began.

**Instance 2: Annotation assistance:** I used Claude to pre-label batches of examples
by providing my three label definitions and a set of unlabeled posts and asking it to
assign one label per post. I reviewed and corrected every pre-assigned label before
accepting it. During this review I identified that Claude had written the label as
`hot take` with a space rather than `hot_take` with an underscore , I corrected all
instances before saving the final CSV. I also identified 9 borderline cases where I
disagreed with or wanted to document Claude's assignment, which became the notes column
entries in my dataset.

**Instance 3: Failure pattern analysis:** After fine-tuning, I pasted my 10 wrong
predictions into Claude and asked it to identify common themes across the errors. Claude
identified that every wrong prediction was classified as `hot_take` and suggested the
model had collapsed onto that label as a default under uncertainty. I verified this
myself by reading all 10 wrong predictions , the pattern held, all errors were
directional toward `hot_take` with no confusion between `analysis` and `reaction`. I
also noticed independently that all confidence scores on wrong predictions fell between
0.35 and 0.39, indicating the model was genuinely uncertain rather than confidently
wrong, which Claude had not flagged.