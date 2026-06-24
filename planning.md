# TakeMeter Planning

Course: AI201 | Applications of AI Engineering
Project: Week 3 Show - TakeMeter

## 1. Community

Chosen community: `r/DeadlockTheGame`

Reddit link: https://www.reddit.com/r/DeadlockTheGame/

Why this community:

- It is a public Reddit community focused on Valve's game Deadlock.
- The community is active enough to support collecting at least 200 public posts/comments.
- The subreddit has a mix of gameplay feedback, strategy discussion, quick reactions, jokes, and community commentary, which gives the classifier meaningful discourse boundaries to learn.

What kind of discourse happens there:

- Players discuss heroes, balance changes, patches, mechanics, team composition, match experience, and game design.
- Some posts make concrete suggestions or report design problems.
- Some posts open broader discussion about the game's meta or player experience.
- Some posts are mostly quick reactions, jokes, hype, complaints, or low-depth opinions.

## 2. Label Taxonomy

Define 2-4 labels. Each label should be mutually exclusive and grounded in community behavior.

| Label | Definition | Clear Example | Edge Case |
|---|---|---|---|
| `game_feedback` | A post/comment that proposes, critiques, or requests a specific gameplay, balance, UI, hero, map, or systems change. | "Seven's ult warning should be changed because it tells enemies to hide, which works against the ability." | A post can be funny while still being feedback. If it contains a concrete game-change proposal, label it `game_feedback`. |
| `discussion` | A post/comment that invites or contributes to a substantive conversation about heroes, strategy, design, meta, player behavior, or game direction without mainly requesting a specific change. | "Which characters currently have the least coherent kits, and why?" | If the post asks for opinions but also proposes a concrete fix, label it `game_feedback`. |
| `reaction` | A quick emotional response, joke, complaint, hype post, meme-like take, or low-depth opinion that does not contain enough reasoning to count as discussion or feedback. | "Valve I beg you" with no deeper explanation or concrete proposal. | If a reaction includes enough reasoning to start a real conversation, label it `discussion`. |

## 3. Hard Edge Cases

What types of posts/comments may be ambiguous?

- Posts that mix jokes with real design feedback.
- Posts that ask a discussion question but also include a specific proposed change.
- Short complaints that may look like feedback but do not identify a fix or specific system.
- Meme or fan-content posts where the text includes a real game opinion.

Decision rules:

- If the text includes a concrete requested change, critique of a specific mechanic, or actionable design suggestion, label it `game_feedback`.
- If the text mainly asks or answers a broader game question with some reasoning, label it `discussion`.
- If the text is mainly emotional, humorous, hype, complaint, or low-context opinion, label it `reaction`.
- Do not label based only on Reddit flair; use the actual post/comment text.

## 4. Data Collection Plan

Source:

- Public posts and comments from `r/DeadlockTheGame`: https://www.reddit.com/r/DeadlockTheGame/

Target count:

- At least 200 public posts/comments.

Target distribution:

- No single label should be more than 70% of the dataset.
- Aim for at least 20% representation for each label if possible.

Current dataset status:

- `data/labeled_posts.csv` contains 200 labeled public Reddit examples.
- Dataset provenance details are documented in `DATASET_NOTES.md`.
- Current columns: `title`, `body`, `label`, `url`.
- Current label distribution:
  - `game_feedback`: 67 examples
  - `discussion`: 67 examples
  - `reaction`: 66 examples
- No label exceeds 70% of the dataset.
- Each label has at least 20% representation.
- All 200 rows have a unique Reddit post URL.
- All 200 rows have a unique title/body pair.
- The `body` field is blank for 41 title-only or media/link-first posts.
- All blank-body rows were checked; none have titles shorter than 25 characters.

Collection process:

- Read 30-40 recent and top posts before finalizing the labels.
- Initial scouting pass completed in `data_scouting.md`.
- Collect public post titles and self-text.
- Exclude private messages, deleted content, and posts where the text is only an image/video with no meaningful text.
- Store examples in `data/labeled_posts.csv` with `title`, `body`, `label`, and `url` columns.
- Record difficult labeling decisions in the Difficult Examples Log below.

If one label is underrepresented:

- Search the subreddit by flair and topic to intentionally collect more examples from the missing label.
- For `game_feedback`, look for posts with feedback, suggestion, balance, patch, hero, or mechanic language.
- For `discussion`, look for question-style posts and longer comment threads.
- For `reaction`, look for short opinion, hype, complaint, or joke posts.

## 5. Evaluation Metrics

Required metrics:

- Overall accuracy
- Per-class precision, recall, or F1
- Confusion matrix

Why these metrics matter:

- Overall accuracy gives the headline result.
- Per-class metrics show which labels the model handles well or poorly.
- The confusion matrix shows which labels are being mixed up.

## 6. Definition Of Success

Success threshold:

- Fine-tuned model reaches at least 70% overall accuracy on the held-out test set and has no class with F1 below 0.60.

A useful classifier should:

- Separate actionable game feedback from general discussion and quick reactions.
- Avoid collapsing everything into the majority label.
- Perform better than the zero-shot Groq baseline on at least overall accuracy or the weakest class.

## AI Tool Plan

### Label Stress-Testing

How AI will be used to test label boundaries:

- AI will be asked to generate borderline examples between `game_feedback`, `discussion`, and `reaction`.
- The generated edge cases will be used to tighten definitions and decision rules before labeling the full dataset.

### Annotation Assistance

Will AI pre-label examples?

- Possibly, but only as a first pass if manual labeling becomes slow.

Human review plan:

- Every label must be reviewed and corrected manually.

### Failure Analysis

How AI will help analyze wrong predictions:

- After evaluation, AI may be used to group wrong predictions into error patterns, such as feedback being confused with discussion or reactions being confused with discussion.

Manual verification plan:

- Every AI-suggested error pattern will be checked against the actual examples before being included in the README.

## Difficult Examples Log

Track at least 3 difficult examples and the final decision.

| Text / Summary | Possible Labels | Final Label | Reason |
|---|---|---|---|
| Avoid-hero select voice-line suggestion, written partly as jokes but proposing a real feature. | `game_feedback`, `reaction` | `game_feedback` | The post is humorous, but it contains a concrete feature suggestion and examples. |
| "Characters with the most incoherent/boring kits in the game?" | `discussion`, `game_feedback` | `discussion` | The post asks for opinions and reasoning about kit quality; it does not request one specific change. |
| Short hype/comment such as "WE NEED THIS" in a feedback thread. | `reaction`, `game_feedback` | `reaction` | The surrounding thread is feedback, but the individual comment is only an emotional response. |
