# TakeMeter - planning.md

## Community

**Communities** :r/smashbros r/NintendoSwitch, r/supersmashbros, r/SuperSmashBros ( all four related to Super Smash Bros. games )

I chose this community because gaming post about Smash Bros clearly devides into three labels. It's a good fit for classification
because the discourse varies significantly — some posts questions, while others posts news about tournaments and its results, character reveals, developer patches, hot takes and regulars clearly distinguish between them.

## Labels

| Label      | Definition                                                                                                                        |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------- |
| news       | Posts that provide information about tournaments and results, annoucements, developer news, character reveals.                    |
| hot_take   | a bold, emotional and controversial opinion that takes our attantion or spark debate. Occassionaly in those posts includes videos |
| discussion | Casual, personal posts where users ask questions and express a feeling in the moment                                              |

## Hard Edge Cases

Most ambiguous boundary is between hot_take and discussion. A post like "Wow Everyone is Doing Matchup Charts, Here is my Matchup Chart for Aegis and in Depth Analysis of the Characters" could either way because because it's a casual question but it spark the debate.

**Resolution rule:** When a post fits both, I will assign it to `discussion` if
it casual question to get help, suggestion or answer from other users, and `hot_take`
if it include own claim inside their post and spark debate within this post.

## Data Collection Plan

**Source** - r/smashbros, r/NintendoSwitch, r/supersmashbros, r/SuperSmashBros scraped via Reddit JSON API filtered to Hot and Top

**Target per label:**

- `discussion`: ~70 examples
- `hot_take`: ~70 examples
- `news`: ~60 examples

**If a label is underrepresented after 200 examples:** I will search r/smashbros r/NintendoSwitch, r/supersmashbros, r/SuperSmashBros using keywords associated with that label (e.g., "patches" for
news through search) and expand the date range back previous year if needed

### Target Distribution

| Label       | Target Count | Examples                                                                                                                                                                                                                    |
| ----------- | ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| discussions | 70 examples  | 1. Uh... what happened..? 2. What stage is this?                                                                                                                                                                            |
| hot_take    | 70 examples  | 1. How society sees the smash bros franchise 2. Tips for intense Boss Battles?                                                                                                                                              |
| news        | 60 examples  | 1. Smash Bros. creator Masahiro Sakurai was voted #1 favorite video game creator in a Famitsu survey. The survey had more than 1,500 participants. 2. Patchwork 2026: A Love Letter Ultimate Singles - Preliminary Seeding. |

|

## Evaluation Metrics

- **Accuracy** - baseline sanity check, but insufficient alone because with 70/20/10% class split, a model would predict most of these labels that scores 70% accuracy while other two labels would be completly ignored
- **Macro F1** - neeeded to treat each label equally; it matters because all three labels types are equally important for building the perfect score
- **Per-class F1** - it is needed because if `news` posts would be much rarer, then model would ignore them entirely and still post the decent overall score
- **Confusion Matrix** - to see which labels the model confuse most often

## Definitinon Of Success

The classifier is **genuinely useful** if it achieves:

- Macro F1 ≥ 0.70 on the held-out test set
- No single label with F1 < 0.60

**"Good enough for deployment"** means a moderator could trust the classifier
to pre-sort a post queue with fewer than 1 in 4 errors, which I would verify
by checking that the confusion matrix shows no label being misclassified more
than 30% of the time.

These criteria are objective: at the end of the project I would look at the
evaluation output and answer yes/no without interpretation.

## Baseline Results

🎯 Baseline accuracy: 0.714 (evaluated on 28/30 parseable responses)

Per-class metrics (baseline):
precision recall f1-score support

        news       1.00      0.57      0.73         7
    hot_take       0.67      0.73      0.70        11

discussion 0.67 0.80 0.73 10

    accuracy                           0.71        28

macro avg 0.78 0.70 0.72 28
weighted avg 0.75 0.71 0.71 28

Hypothesis: The model struggled most with hot_take.
I think this is because hot_take and discussion look similar since both are personal posts

## Fine-Tune Results

🎯 Fine-tuned model accuracy: 0.367

Per-class metrics (fine-tuned model):
precision recall f1-score support

        news       0.67      0.22      0.33         9
    hot_take       0.33      0.09      0.14        11

discussion 0.33 0.80 0.47 10

    accuracy                           0.37        30

macro avg 0.44 0.37 0.32 30
weighted avg 0.43 0.37 0.31 30

Hypothesis: The model struggled most with hot_take.
I think this is because hot_take and discussion look similar since both are personal posts

## AI Tool Plan

### Label stress-testing

I will use Claude to help wth my label definitions and edge case description and ask it to
generate 10 posts sitting at the boundary between `label_one` and `label_two`.
If I can't cleanly classify all 10, I'll revise the definitions before annotating.

### Annotation assistance

- I will collect manually these example and would not use an LLM to pre-label examples

### Failure analysis

After evaluation I'll give Claude my list of misclassified examples and ask it
to identify patterns (e.g., "does the model consistently confuse X for Y when
the post is short?"). I'll verify any pattern it flags by manually checking at
least 5 examples that fit the pattern before including it in my write-up.
