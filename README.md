# TakeMeter

AI201 Week 3 project: fine-tuned text classifier for discourse quality in an online community.

## Project Overview

This project builds a classifier that labels posts or comments from a chosen online community. The goal is to design a clear label taxonomy, collect and annotate real examples, fine-tune a model, compare it against a zero-shot Groq baseline, and analyze where the model fails.

## Community

This project uses public posts and comments from `r/DeadlockTheGame`, a Reddit community focused on Valve's game Deadlock.

The classifier is designed to separate three common forms of community discourse:

- actionable game feedback
- substantive game discussion
- quick reactions, jokes, hype, complaints, or low-depth takes

## Label Taxonomy

| Label | Definition | Examples |
|---|---|---|
| `game_feedback` | Proposes, critiques, or requests a specific gameplay, balance, UI, hero, map, or systems change. | A post suggesting a hero ability, map layout, UI element, or balance value should change. |
| `discussion` | Invites or contributes to a substantive conversation about heroes, strategy, design, meta, player behavior, or game direction. | A post asking which character kits are least coherent and why. |
| `reaction` | Quick emotional response, joke, complaint, hype post, meme-like take, or low-depth opinion. | A short complaint or joke that does not include enough reasoning to count as discussion or feedback. |

## Dataset

Dataset file: `data/labeled_posts.csv`

Requirements:

- At least 200 public posts/comments.
- Columns: `text`, `label`, `url`.
- No private or authenticated-only data.
- No single label should dominate the dataset.

Planned source: `r/DeadlockTheGame`

Dataset status:

- Total rows: 200
- `game_feedback`: 67
- `discussion`: 67
- `reaction`: 66
- Every row has a unique Reddit post URL.
- The `text` column contains the post title plus body when body text was available.
- Some examples are title-only or media/link-first Reddit posts, but their titles were reviewed to make sure they were meaningful enough for classification.
- The `url` column keeps a source link for each example.

## Fine-Tuning Approach

Base model: `distilbert-base-uncased`

Training environment: Google Colab T4 GPU

Starter notebook copy: https://colab.research.google.com/drive/1kwbFbip1-_vlnPodOOgjcR87K2M9j7dF

Hyperparameter notes:

- Epochs: 3
- Learning rate: `2e-5`
- Train batch size: 16
- Eval batch size: 32
- Weight decay: `0.01`
- Warmup steps: 50

The model was trained on a 70% / 15% / 15% split:

- Train: 140 examples
- Validation: 30 examples
- Test: 30 examples

The test set was stratified and contained 10 examples from each label.

## Baseline Comparison

Baseline model: Groq `llama-3.3-70b-versatile`

Baseline prompt summary:

- The prompt described `r/DeadlockTheGame`, defined the three labels, gave one example per label, and instructed the model to return only one valid label string.
- The baseline was run on the same locked 30-example test set as the fine-tuned model.

## Evaluation Results

| Model | Overall Accuracy | Notes |
|---|---:|---|
| Groq zero-shot baseline | 0.800 | Best overall result; handled `reaction` especially well. |
| Fine-tuned DistilBERT | 0.367 | Regressed compared with baseline; mostly collapsed toward `game_feedback`. |

Fine-tuning produced a regression of `-0.433` compared with the Groq baseline. This result is important because it shows that fine-tuning is not automatically better than prompting. With only 200 examples and overlapping label boundaries, the smaller fine-tuned classifier did not learn the distinctions as well as the prompted LLM.

## Per-Class Metrics

### Groq Zero-Shot Baseline

| Label | Precision | Recall | F1 | Support |
|---|---:|---:|---:|---:|
| `discussion` | 0.67 | 0.80 | 0.73 | 10 |
| `game_feedback` | 1.00 | 0.60 | 0.75 | 10 |
| `reaction` | 0.83 | 1.00 | 0.91 | 10 |

### Fine-Tuned DistilBERT

| Label | Precision | Recall | F1 | Support |
|---|---:|---:|---:|---:|
| `discussion` | 0.00 | 0.00 | 0.00 | 10 |
| `game_feedback` | 0.36 | 1.00 | 0.53 | 10 |
| `reaction` | 1.00 | 0.10 | 0.18 | 10 |

## Confusion Matrix

Fine-tuned DistilBERT confusion matrix:

| True \ Predicted | `discussion` | `game_feedback` | `reaction` |
|---|---:|---:|---:|
| `discussion` | 0 | 10 | 0 |
| `game_feedback` | 0 | 10 | 0 |
| `reaction` | 1 | 8 | 1 |

The confusion matrix shows that the fine-tuned model predicted `game_feedback` for 28 of the 30 test examples. It correctly identified all 10 true `game_feedback` examples, but missed every `discussion` example and only identified 1 of the 10 `reaction` examples. This suggests the model learned a broad "Deadlock post equals feedback" shortcut instead of learning the intended discourse boundaries.

## Wrong Predictions Analysis

| Text / Summary | Correct Label | Predicted Label | Why It Failed |
|---|---|---|---|
| "Do anybody know how to do the long wall jump with shiv?" | `discussion` | `game_feedback` | The post asks for technique help, but because it mentions a specific hero/mechanic, the model treated it like actionable game feedback. |
| "Was this a outplay? Im fairly new to the game but i thought this was impressive lol" | `reaction` | `game_feedback` | This is a short emotional/media-style reaction, but the model over-weighted gameplay words and ignored the low-depth tone. |
| "Who is the no. 1 mo and krill player?" | `discussion` | `game_feedback` | The post asks for a community opinion, but the model treated the named hero/player topic as if it were feedback. |
| "Vindicta is so much fun..." | `reaction` | `game_feedback` | The post expresses enjoyment and hype, but the presence of a hero name made it look like gameplay feedback to the model. |

## Sample Classifications

| Text / Summary | Predicted Label | Confidence | Correct? | Notes |
|---|---|---:|---|---|
| "Remove camps..." | `game_feedback` | N/A | Likely yes | Concrete request to remove or change neutral camps. |
| "Who is the no. 1 mo and krill player?" | `game_feedback` | 0.39 | No | Should be `discussion`; asks for community opinion. |
| "Vindicta is so much fun..." | `game_feedback` | 0.35 | No | Should be `reaction`; mainly expresses enjoyment. |

Confidence is included where the Colab printed it for wrong fine-tuned predictions. The baseline notebook output did not export per-example confidence.

## Reflection

What the model learned versus what was intended:

- The intended task was to separate actionable feedback, broader discussion, and quick reactions. The fine-tuned model mostly learned that many Deadlock posts mention mechanics, heroes, bugs, balance, or gameplay situations, and it over-applied `game_feedback`. It did not learn the difference between "talking about a mechanic" and "requesting or critiquing a mechanic."

One way the spec helped:

- The spec made the label boundaries clear enough to diagnose the model's failure. Because `planning.md` defined edge cases, it was easier to explain why wall-jump questions should be `discussion`, why hype posts should be `reaction`, and why bug/control issues should be `game_feedback`.

One way implementation diverged from the spec:

- The original goal was for fine-tuning to improve over the baseline, but the final results showed the opposite. The implementation still followed the required pipeline, but the smaller fine-tuned model and limited dataset were not enough to beat the prompted Groq baseline.

Next steps if improving the model:

- Add more examples for `discussion` and `reaction`, especially examples that mention heroes/mechanics but are not feedback.
- Add more short reaction posts so the model learns tone and depth, not only topic words.
- Clean borderline labels and consider a narrower taxonomy if the categories remain too overlapping.
- Try a larger model, more epochs with early stopping, or class-balanced sampling after improving the data.

## AI Usage

1. AI was used to help design and stress-test the label taxonomy, especially the boundary between `game_feedback`, `discussion`, and `reaction`.
2. AI was used to help structure the project notes, dataset documentation, and final README evaluation report.
3. AI was used to inspect the Colab results and summarize model failure patterns from the wrong predictions.

Annotation assistance disclosure:

- AI helped with planning, formatting, and review, but labels were reviewed against the actual post text. The final dataset reflects the project taxonomy rather than unreviewed automatic labels.

## Demo Video

Demo file/link: https://www.loom.com/share/10a7024f5b8643bd993958deba44c706