# planning.md — TakeMeter: r/soccer Discourse Classifier

## Community

I chose **r/soccer** (reddit.com/r/soccer), an 8.7-million-member community dedicated to global football discussion. It's an excellent fit for a classification task because its discourse spans a genuinely wide spectrum from instant emotional reactions to match events, to bold opinion posts, to structured tactical arguments backed by data. Regular participants in r/soccer actively distinguish between these types of posts: high-quality analysis gets upvoted and praised; unsupported hot takes get called out; raw reactions fill match threads. This means my labels are grounded in distinctions the community itself cares about, not just distinctions I'm imposing from outside.

---

## Labels

### `analysis`
A comment or post that makes a structured argument about soccer using specific, verifiable evidence, statistics, tactical observations, historical comparisons, or concrete match examples to support a claim.

**Example 1:** "Rodri's absence is massive for City. Their pressing intensity dropped from 68% PPDA to 54% PPDA without him last season, and they lost 7 of 11 games. He's not just a DM, he's their entire defensive structure."

**Example 2:** "Spain's tiki-taka only worked because Busquets won the ball back within 6 seconds consistently. Stats show they recovered possession in the final third 34% more than any other team at Euro 2012."

---

### `hot_take`
A bold, confident opinion about a player, team, or match stated without specific supporting evidence. The post asserts a strong claim but does not argue for it with verifiable facts.

**Example 1:** "Mbappe is already better than Ronaldo ever was. The guy is only 25 and he's already won everything. People are being delusional."

**Example 2:** "The Premier League is genuinely the worst tactically coached league in Europe. Every team just kicks it long and runs. La Liga is on another level."

---

### `reaction`
An immediate emotional response to a specific match event, result, or news expressing a feeling in the moment without making a broader argument or claim.

**Example 1:** "WHAT A GOAL. I cannot believe what I just watched. Absolute scenes at the Bernabeu tonight"

**Example 2:** "I'm shaking. We actually did it. After everything this season, we actually did it. Up the Arsenal."

---

## Hard Edge Cases

**The `hot_take` vs `analysis` boundary** is the hardest case. Example:

> "Bellingham is the best midfielder in the world right now. His numbers this season are insane, 23 goals and 12 assists from midfield."

This *looks* like it cites evidence but the stat is vague decoration, not a structured argument. 

**Decision rule:** If you can remove the "evidence" and the post still makes the same claim with the same force, it's a `hot_take`. Evidence only counts toward `analysis` if it is specific, verifiable, AND structurally necessary to the argument (i.e., the claim depends on it).

The `hot_take` vs `reaction` boundary is easier: reactions are tied to a *specific, recent event*. Hot takes are general claims about players/teams/leagues that could be written any day.

---

## Data Collection Plan

**Source:** r/soccer comment sections, specifically:
- Match threads (top-level comments) excellent for `reaction`
- "Discussion" flair posts and their comment sections excellent for `hot_take` and `analysis`
- Weekly "Tactical Analysis" or long-form posts excellent for `analysis`

**Target distribution:** ~70 examples per label (≈210 total), aiming for a near-even 33/33/33 split.

**If a label is underrepresented:** I will deliberately seek out posts from that label type e.g., if `analysis` is underrepresented, I'll visit posts with "tactical" or "stats" keywords, or browse r/soccer's "Analysis" flair.

**Collection method:** Manual copy-paste into a CSV with columns: `text`, `label`, `notes`. I will collect at least 30–40 examples first and review them against my labels before committing to annotating the full 200.

---

## Evaluation Metrics

I will use:
- **Overall accuracy** (required baseline comparison)
- **Per-class F1 score**  because my classes are roughly balanced and F1 captures both precision and recall in one number, which matters here since missing a `reaction` is just as bad as missing an `analysis`
- **Confusion matrix** to identify *which specific label pairs* the model confuses

Accuracy alone is insufficient because even with 3 balanced classes, a model could game accuracy by getting one class right while failing completely on another. F1 per class catches that.

---

## Definition of Success

The fine-tuned model is "good enough" if:
- Overall accuracy ≥ 70% on the test set
- No single class F1 score is below 0.55 (i.e., it's learning all three distinctions, not ignoring one)
- Fine-tuned model outperforms the zero-shot Groq baseline by at least 10 percentage points overall

These thresholds are realistic for 200 training examples on a subjective 3-class task.

---

## AI Tool Plan

1. **Label stress-testing:** I will give Claude my label definitions and ask it to generate 10 posts that sit at the boundary between `hot_take` and `analysis`. If any of those posts are genuinely hard to classify, I'll tighten my decision rule before annotating 200 examples.

2. **Annotation assistance:** I may use Claude to pre-label batches of 20–30 posts at a time using my label definitions, then review and correct every prediction before accepting it. I will track which examples were pre-labeled in my notes column.

3. **Failure analysis:** After fine-tuning, I will paste my list of wrong predictions into Claude and ask it to identify common themes (post length, label pair, presence of sarcasm, etc.), then verify those patterns myself by re-reading the examples.

---

## Stretch Features

- **Deployed Interface:** Build a simple web interface using the fine-tuned model that takes a new post as input and displays the predicted label and confidence score.