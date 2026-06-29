# TakeMeter - README.md

## Community Selection
**Selected Community:** `r/leagueoflegends`.

**Why this community fits the task:**
As someone who has been part of the League of Legends community for a long time, I am highly familiar with how intensely players react to systemic meta shakeups. Every season brings massive balance overhauls, and the player base naturally fragments over how these changes alter the core loop of the game. Discourse on the subreddit covers a vast range of highly debated topics, from the mechanical impact of major system changes and optimal item build paths, to how tweaks in map timing alter overall team macro and lane priority. 

Because players experience these updates with wildly different levels of game knowledge and emotional investment, the quality of text discourse varies significantly. While some players write structured, numbers-driven breakdowns evaluating statistical efficiency, others confidently spin broad, unbacked narratives about the state of the ladder being "cooked," and tilted players simply use the platform to vent raw frustration after a bad loss. This natural friction between strategy (`analysis`), confident opinion (`hot_take`), and venting (`reaction`) makes `r/leagueoflegends` an incredibly rich environment for a text classification task.

## Label Taxonomy
**`analysis`** — The post makes a structured argument backed by statistics, historical comparison, or tactical observation. Evidence is specific and verifiable.
* **Example 1:** 
  > *"Crit should be removed and ADCs get their base stats improved to compensate, so they have a power curve closer to every other champion class. Because adcs scale with 3 stats, AD AS and crit, their damage output is always way higher than other carries, and so they have to be crap early game to balance out how strong they are late game, and a whole role of kassadins and kayles is always going to feel bad when everyone else has at least some early game agency"* ([Source](https://www.reddit.com/r/leagueoflegends/comments/186sun5/comment/kbawi2i/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button))
* **Example 2:**
  > *" IMO jungle needs more balancing relative to lanes. Being 2-3 levels behind top lane and 1 level behind mid lane while being ahead in ganks, cs, and objectives is ridiculous. Very easy to lose control of a game when you have an inting top lane especially. Then it’s virtually impossible to catch up to their top lane unless they massively fumble. The reduced objectives/feats is definitely a welcome change. "* ([Source](https://www.reddit.com/r/leagueoflegends/comments/1sld2td/comment/og5nuqc/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button))

**`hot_take`** — A bold, confident opinion stated without supporting evidence. The claim might be true, but the post asserts rather than argues.
* **Example 1:**
  > *"Top lane is currently the strongest lane in soloq for anyone below grandmaster (99.9% of the player base)."* ([Source](https://www.reddit.com/r/leagueoflegends/comments/186sun5/comment/kbay9oj/))
* **Example 2:**
  > *"Unless you are at the very top of the ladder, any "balance issue" you have is actually a skill issue. The fact that there are people ranked higher than you means they know something you don't, so figure it out and adapt instead of complaining into the wind. You have no control over game balance, so why worry about it?"* ([Source](https://www.reddit.com/r/leagueoflegends/comments/186sun5/comment/kbad7z6/))

**`reaction`** — An immediate emotional response to a specific event. Little to no argument — the post is expressing a feeling in the moment.
* **Example 1:**
  > *"I'd be totally fine if Riot remove some champs. Game would 10x better without them."* ([Source](https://www.reddit.com/r/leagueoflegends/comments/186sun5/comment/kbb2yi4/))
* **Example 2:**
  > *"People who build heart steel on assassins are pussies and need to reevaluate their lives"* ([Source](https://www.reddit.com/r/leagueoflegends/comments/186sun5/comment/kbf278h/))


## Data Collection Source
All data was collected from `r/leagueoflegends` by manually searching keywords likely to surface posts across all three label types. Keywords included terms like *season 2026 thoughts*, *why I quit league*, *unpopular opinions*, *ranked*, and similar phrases. Comments were pulled from the resulting threads, targeting a mix of tones and post lengths to ensure variety across the dataset.

## Labeling Process
Data was collected and labeled in two phases.

For the first 100 examples, I labeled everything myself. On examples where I was unsure, I cross-referenced with Gemini Flash-Lite as a second opinion to help resolve edge cases and keep my labels consistent.

For the second 100, I scraped comments in batches of 25 and had Claude label them, then reviewed each batch myself before moving on. Working in smaller batches also made it easier to catch label imbalances early — if one class was accumulating too fast, I could adjust what I was scraping before the distribution got too skewed.

## Label Distribution
The final dataset contains 200 labeled examples with a reasonably balanced distribution across all three classes.

| Label | Count | Percentage |
| -------- | -------- | -------- |
| analysis | 72 | 36% |
| hot_take | 66 | 33% |
| reaction | 62 | 31% |
| **Total** | **200** | **100%** |

* **Train/Validation/Test Split:** The dataset was stratified to maintain this distribution across splits (140 training, 30 validation, 30 test), ensuring the model encountered a consistent ratio of each discourse type throughout the learning process.

## Examples that were difficult to label
1. Row 168 — "I hate season 11" (labeled `hot_take`)
The post starts with strong emotion, which made me consider `reaction`. But `reaction` should respond to a specific event, and this doesn't. Instead, the author argues that the game's meta rewards passive play and punishes roaming. Since those claims are opinions without evidence, I labeled it `hot_take`, although it was a close call.

---
2. Row 173 — Jaksho/Heartsteel rant (labeled `reaction`)
This almost became `analysis` because the author explains why healing is stronger, saying tanks and bruisers stay alive long enough to trigger more healing. However, the post is mostly based on one personal game and ends as a frustrated rant rather than a broader analysis. I kept it as `reaction`, but this was the label I was least confident about.

---
3. Row 152 — ADC power vs. agency (labeled `analysis`)
The author makes a clear distinction between power (damage) and agency (impact on the game), giving the post a structured argument. Even though it doesn't include statistics or other evidence, the reasoning is organized enough that I labeled it `analysis`. Still, `hot_take` would also have been a reasonable choice.

## Fine-Tuning Approach

### 1. Base Model

- **Model:** `distilbert-base-uncased`
- **Rationale:** DistilBERT offers an optimal balance of performance and efficiency for smaller, community-specific datasets. Compared to full-sized BERT, it is 40% smaller and 60% faster, making it well-suited for constrained infrastructure like a T4 GPU while maintaining comparable accuracy for downstream classification tasks. The uncased variant was chosen so the model focuses on semantics regardless of capitalization, which is common in informal, unedited Reddit discourse.

### 2. Training Setup

- **Infrastructure:** Google Colab T4 GPU using the `Trainer` API from Hugging Face `transformers`.
- **Tokenization:** Maximum sequence length of 256 tokens — long enough to capture the nuance of Reddit comments while filtering out excessive noise.
- **Data Split:** Stratified across training (70%), validation (15%), and test (15%) sets to maintain a consistent label distribution throughout training.
- **Training Loop:** 3 epochs, which proved sufficient to capture patterns across 140 training examples without significant overfitting.

### 3. Hyperparameter Decisions

- **Learning Rate (2e-5):** Industry standard for fine-tuning BERT-family models — low enough to preserve pre-trained knowledge, high enough to adapt weights to our specific taxonomy.
- **Weight Decay (0.01):** Adds light regularization to prevent the model from overfitting to the vocabulary of the training set.
- **Batch Size (16):** Chosen to fit within T4 GPU memory constraints while maintaining stable gradient updates.
- **Warmup Steps (50):** Allows the model to stabilize its initial weights before applying the full learning rate — especially important for small, high-variance datasets.

## Baseline description
I established a zero-shot baseline using the Groq API (Llama-3). I fed my 30 test examples into the model without providing any training data. The model's raw text outputs were then evaluated against my manual ground-truth labels to calculate the baseline accuracy (0.700), precision, and recall.

### Baseline Prompt
I used the following system prompt to instruct the model on my specific taxonomy:
```
SYSTEM_PROMPT = """
You are classifying <posts> from r/leagueoflegends.
Assign each post to exactly one of the following categories.

<analysis>: The post makes a structured argument backed by statistics, historical comparison, or tactical observation. Evidence is specific and verifiable.
Example: "Crit should be removed and ADCs get their base stats improved to compensate, so they have a power curve closer to every other champion class. Because adcs scale with 3 stats, AD AS and crit, their damage output is always way higher than other carries, and so they have to be crap early game to balance out how strong they are late game, and a whole role of kassadins and kayles is always going to feel bad when everyone else has at least some early game agency"

<hot_take>: A bold, confident opinion stated without supporting evidence. The claim might be true, but the post asserts rather than argues.
Example: "Top lane is currently the strongest lane in soloq for anyone below grandmaster (99.9% of the player base)."

<reaction>: An immediate emotional response to a specific event. Little to no argument — the post is expressing a feeling in the moment.
Example: "I'd be totally fine if Riot remove some champs. Game would 10x better without them."

Respond with ONLY the label name.
Do not explain your reasoning.

Valid labels:
<analysis>
<hot_take>
<reaction>
"""
```

---

## Evaluation Report

The following report evaluates two models on a 30-example held-out test set across three classes: `analysis`, `hot_take`, and `reaction`.

### Baseline Model

The baseline uses a zero-shot prompt with no fine-tuning. It achieves solid overall accuracy but struggles heavily with `hot_take`, capturing only 20% of true positives.

Accuracy: 0.700
| Class | Precision | Recall | F1-Score | Support |
| -------- | -------- | -------- | -------- | -------- |
| analysis | 0.77 | 0.91 | 0.83 | 11 |
| hot_take | 1.00 | 0.20 | 0.33 | 10 |
| reaction | 0.60 | 1.00 | 0.75 | 9 |
| macro avg | 0.79 | 0.70 | 0.64 | 30 |
| weighted avg | 0.80 | 0.70 | 0.64 | 30 |

---

### Fine-Tuned Model

The fine-tuned model was trained on `distilbert-base-uncased` using the hyperparameters below. While overall accuracy stays the same as baseline, it shows more balanced performance across classes — especially improving `reaction` recall and reducing overconfidence on `hot_take`.

#### Hyperparameters
```python
TrainingArguments(
    output_dir="./takemeter-model",
    num_train_epochs=3,
    per_device_train_batch_size=16,
    per_device_eval_batch_size=32,
    learning_rate=2e-5,
    weight_decay=0.01,
    warmup_steps=50,
    eval_strategy="epoch",
    save_strategy="epoch",
    save_total_limit=1,
    load_best_model_at_end=True,
    metric_for_best_model="accuracy",
    logging_steps=10,
    report_to="none",
)
```

Accuracy: 0.700
| Class | Precision | Recall | F1-Score | Support |
| -------- | -------- | -------- | -------- | -------- |
| analysis | 0.64 | 0.82 | 0.72 | 11 |
| hot_take | 0.75 | 0.30 | 0.43 | 10 |
| reaction | 0.75 | 1.00 | 0.86 | 9 |
| macro avg | 0.71 | 0.71 | 0.67 | 30 |
| weighted avg | 0.71 | 0.70 | 0.66 | 30 |


### Confusion Matrix (Fine-Tuned Model)

The confusion matrix breaks down where the model succeeds and where it gets confused. `hot_take` is the most problematic class — 5 of 10 true hot takes were misclassified as `analysis`.

| True \ Predicted | analysis | hot_take | reaction |
| :--- | :---: | :---: | :---: |
| **analysis** | 9 | 1 | 1 |
| **hot_take** | 5 | 3 | 2 |
| **reaction** | 0 | 0 | 9 |


### Error Analysis

The three examples below highlight cases where the model's predictions differed from the true label. Each one reveals a different type of challenge — tone vs. intent, structure vs. opinion, and annotation ambiguity.

#### Example 1
* **Text:** 
> zac is broken. idc what everone says wholesome blob or whatever. for building full tank he still can really good damage. his healing is insane when u take into account his tankiness. also this bastards cc is illegal since it cant be reduced by tenacity. and fuck his E
* **True Label:** `hot_take` 
* **Predicted Label:** `reaction` (confidence: 0.34)
*   **Why the Boundary is Hard:** The post contains highly emotional and vulgar language (e.g., "fuck his E," "this bastard's CC"), which strongly resembles the tone typically associated with reaction posts. However, beneath the emotional wording, the author is making a clear argument about game balance, making it a hot_take rather than a simple emotional response.
*   **Labeling vs. Data Problem:** This is a **data problem**. The model appears to over-associate high-intensity negative sentiment with the reaction label, even when the primary purpose of the post is to argue a controversial opinion.
*   **Proposed Fix:** Include more training examples where emotionally charged or vulgar language is used to support a game-balance argument. This would help the model learn that strong emotional expression alone does not distinguish a reaction from a `hot_take`.


#### Example 2
* **Text:**
> on god supports even in plat/emerald don't understand wave states, lvl 2 or even the game. they literally get backpacked to their elo since the role is so disgustingly broken that even braindead players can climb. its also the only role where you can go 0/15 and not be blamed when it's obvious ur high-key inting white getting nothing done. Furthermore, when a support is better than the other one it's so disgustingly obvious more so than other roles. please change my mind!
* **True Label:** `hot_take` 
* **Predicted Label:** `analysis` (confidence: 0.35)
*   **Why the Boundary is Hard:** Despite its inflammatory content, the post is organized as a structured argument. Transitional phrases such as "Furthermore" and the step-by-step reasoning resemble the style of an analysis, even though the overall purpose is to express a provocative and highly subjective opinion.
*   **Labeling vs. Data Problem:** This is a **boundary problem**. The model has learned to rely heavily on formal writing cues, causing it to mistake structured opinions for analytical discussions.
*   **Proposed Fix:** Expand the training set with more examples of provocative opinions presented using logical structure and formal reasoning. This would encourage the model to focus on communicative intent rather than writing style alone.

#### Example 3
* **Text:**
> The balance team does a pretty good job with the hand they are dealt. Sometimes they make mistakes, but they have a huge challenge and handle it better than almost any other comparable game.
* **True Label:** `hot_take` 
* **Predicted Label:** `analysis` (confidence: 0.34)
*   **Why the Boundary is Hard:** The post is calm, balanced, and evidence-oriented, all of which are characteristics typically associated with analysis. Its controversial element lies in the opinion itself rather than its tone, making it difficult for the model to distinguish from a genuine analytical discussion.
*   **Labeling vs. Data Problem:** This is an **annotation consistency problem**. Based on the current label definitions, this example could reasonably be classified as either hot_take or analysis, suggesting ambiguity in the annotation guidelines rather than a modeling failure.
*   **Proposed Fix:** Clarify the label definitions by explicitly stating whether well-reasoned but controversial opinions should be classified as `hot_take`. If not, relabel examples like this as analysis to improve annotation consistency and reduce ambiguity during training.

### Sample Classifications

The examples below show a mix of correct and incorrect predictions from the test set.

#### Example 1
> "Balance itself is really good imo. However I feel that the game starts to be too complex. I've been high diamond/low master from S3 and it during the yearly climb grind, it feels like the level where you can start to count that majority of players know all the ""basic macro mechanics of the game"" have risen a lot. There can be some mechanically godlike player playing a complex champion in mid/high diamond, but they completely lack the game knowledge to know how to react to macro level stuff around the map. As someone who plays simpler champions and have been focusing on the macro side a lot more, that tilts me to a no end. From my experience I'd say that during S3, even majority of high gold players had a basic macro level knowledge needed to play the game correctly. Sometimes their execution lacked, but they still mostly knew what they should be doing. Nowadays I feel that this cutoff is somewhere in Master, and there are still some people even there who have no idea how some basic stuff works."

**True Label:** `analysis` | **Predicted:** `analysis` | **Confidence:** 0.746

The post traces how macro knowledge requirements have shifted across seasons and elos, building a reasoned argument with personal data points — a clear fit for `analysis`.

#### Example 2
> "I think almost all games are winnable, even with huge gold deficits. Though the potential of this is greatly influenced by elo and champions. Maokai being 2k gold down is a lot less problematic than Irelia being 2k gold down for instance. It also depends on how the gold is spread around. If the ADC that's 3k gold ahead keeps making cocky moves then it will also probably matter less than a midlane control mage that stays back. Rather than ask yourself 'was this winnable' try to focus on what you could've done better. It makes you feel better because it keeps you in control of the game. In the end, it would'be been easier if your ADC went 20/0 instead of 0/20. Yet the only thing you can (easily) influence is your own plays. Perhaps your adc was always going to lose. Yet that well timed roam of yours at 7 minutes in could've made him go 3/8 instead of 2/13."

**True Label:** `analysis` | **Predicted:** `analysis` | **Confidence:** 0.709

The post breaks down game mechanics systematically — comparing champions, gold distributions, and player agency — which matches the structured reasoning style of `analysis`.

#### Example 3
> "I hate the game why iam always play with feeders i demoted to bronze lost 10 matches always have bad teams dead 10 times or 12 why why 🤬"

**True Label:** `reaction` | **Predicted:** `reaction` | **Confidence:** 0.845

The raw emotional venting with no argument or reasoning makes this an easy `reaction` call, and the high confidence reflects that.

#### Example 4
> "In terms of balance i really appreciate the game right now more than ever. I kinda dislike the swifty boot abusers right now, but thats something everyone has access to so im kinda fine with it. Imo the Smurf/Secondary Account stuff has gotten out of hand. Im on 300 games this split and i got on average 3 to 4 smurfs in my lobbies. Like 80% of the time its said players which completely ruin the game by stomping or getting stomped. Its just never a fun experience to have those guys in your or in the enemy team because it never feels like the win or lose is deserved."

**True Label:** `hot_take` | **Predicted:** `reaction` | **Confidence:** 0.662

The model likely latched onto the negative experience framing ("never a fun experience") and misread it as emotional venting, missing that the author is making a broader subjective opinion about smurfing as a systemic issue.


## Reflection
After evaluating the model on the test set, I found a clear gap between how I intended the labels to work and how the model actually learned to classify posts. My taxonomy was based on the author's intent (whether they were trying to explain, argue, or vent), but the model relied mostly on surface-level language patterns.

---
### What the Model Learned
Instead of learning intent, the model learned shortcuts based on common word patterns.

* **Jargon = Analysis**: I defined `analysis` as logical reasoning about game mechanics. However, the model learned to associate technical terms like wave states, itemization, patches, and meta with `analysis`, even when the post was really just an opinion or rant.
* **Strong Emotion = Reaction**: I defined `reaction` as an emotional response to an event or experience. Instead, the model treated any post with strong negative language or profanity as a `reaction`, even if the author was making a broader argument.
* **Calm Tone = Analysis**: I defined `hot_take` as a subjective or controversial opinion. However, when a hot take was written in a calm, reasonable tone, the model often labeled it as `analysis` because it confused a professional writing style with objective reasoning.

### What the Model Missed
The biggest weakness was that the model did not understand why the author wrote the post.

* It struggled to tell the difference between someone explaining a game mechanic (`analysis`) and someone using that mechanic to support an opinion (`hot_take`).
* It also had trouble recognizing that a hot take does not have to be emotional. Calm, well-written opinions were often mistaken for analysis.
* Finally, the model focused on emotional words instead of the actual claim being made. For example, a statement like "Zac is broken" is a debatable opinion about game balance, but the model often classified it as a `reaction` because of the negative language.

---
Overall, my taxonomy depends on understanding the author's intent, while the model mainly learned patterns in vocabulary, tone, and sentiment. This caused it to rely on superficial features instead of the meaning behind each post.

To improve future versions, I would include more difficult training examples that challenge these shortcuts—for example, emotional posts that are actually reasoned arguments (hot_take) and calm, objective-sounding posts that are still emotional reactions (reaction). This would encourage the model to learn the author's intent rather than relying on keywords or writing style alone.

## Spec Reflection 

**One way the spec helped you during implementation:** The spec's requirement to define a clear label taxonomy before collecting data pushed me to think carefully about where the boundaries between categories were before I started labeling. Having those definitions written out early made it much easier to stay consistent, especially when I hit ambiguous posts that could have gone in multiple directions.

**One way your implementation diverged from the spec, and why:** The spec suggested labeling the full dataset manually, but I used Claude to label the second half in batches of 25 and then reviewed each batch myself. I made this call because after labeling the first 100 examples, I had a strong enough sense of the taxonomy that I trusted AI-assisted labeling with human review to be just as consistent — and it let me hit 200 examples without burning out on annotation fatigue.

## AI Usage

**Instance 1 — Data Labeling (Claude Sonnet 4.6)**

- *What I gave the AI:* My label taxonomy and definitions, along with batches of 25 scraped Reddit comments.
- *What it produced:* A label (`analysis`, `hot_take`, or `reaction`) for each comment in the batch.
- *What I changed or overrode:* I reviewed every batch myself before accepting the labels and corrected any that felt off based on my own judgment. A few borderline posts — especially ones where emotional tone and structured opinion overlapped — I relabeled after review.

**Instance 2 — Error Pattern Analysis (Gemini Flash-Lite)**

- *What I gave the AI:* The misclassified examples from my fine-tuned model's Colab evaluation output, including the true label, predicted label, and confidence score for each wrong prediction.
- *What it produced:* A breakdown of possible patterns and reasons behind the errors — for example, that the model was over-associating formal writing style with `analysis` and strong emotion with `reaction`, regardless of the author's actual intent.
- *What I changed or overrode:* I used Gemini's observations as a starting point and cross-checked them against the actual examples myself. Some patterns it flagged were confirmed by the data; others I filtered out because they didn't hold up across multiple examples.