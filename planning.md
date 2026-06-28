# TakeMeter — planning.md

## Community Selection
**Selected Community:** `r/leagueoflegends`.

**Why this community fits the task:**
As someone who has been part of the League of Legends community for a long time, I am highly familiar with how intensely players react to systemic meta shakeups. Every season brings massive balance overhauls, and the player base naturally fragments over how these changes alter the core loop of the game. Discourse on the subreddit covers a vast range of highly debated topics, from the mechanical impact of major system changes and optimal item build paths, to how tweaks in map timing alter overall team macro and lane priority. 

Because players experience these updates with wildly different levels of game knowledge and emotional investment, the quality of text discourse varies significantly. While some players write structured, numbers-driven breakdowns evaluating statistical efficiency, others confidently spin broad, unbacked narratives about the state of the ladder being "cooked," and tilted players simply use the platform to vent raw frustration after a bad loss. This natural friction between strategy (`analysis`), confident opinion (`hot_take`), and venting (`reaction`) makes `r/leagueoflegends` an incredibly rich environment for a text classification task.


## Labels
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

## Hard Edge Cases
(`analysis` vs. `hot_take`): Posts that use specific game jargon, patch numbers, or item names—which initially makes them look like analysis—but use those terms purely as hyperbole or window dressing to make an unbacked claim about the game's balance.
* **How I will handle it:** If the post actually explains *how* or *why* those numbers impact a mechanic or strategy, it is `analysis`. If the numbers are just used to exaggerate a complaint without a logical, step-by-step breakdown, it defaults to a `hot_take`.

(`hot_take` vs. `reaction`): Posts that are highly aggressive, brief, and emotional, which makes them feel like a reaction, but technically make a definitive statement about the state of a role, champion, or the meta.
* **How I will handle it:** I will look at the scope. If the post makes a universal claim about the game or player base (e.g., asserting a specific role is overpowered), I will label it a `hot_take` regardless of how toxic the tone is. I will reserve `reaction` strictly for posts describing a user's personal feelings, a specific match they just played, or pure, contentless venting.

## Data Collection Plan
### Where will you collect examples?
I will collect examples from public threads on r/leagueoflegends. My primary sources will be the comment sections of "State of the Game" discussions, official Patch Notes megathreads (e.g., current season patches), and community-driven meta-analysis threads.

### How many per label?
My initial goal is to collect 200 total examples. I will aim for a balanced distribution of approximately 65–70 examples per label (analysis, hot_take, reaction). This ensures that no single class dominates the training set, which prevents the model from developing a bias toward predicting the majority class.

### What will you do if a label is underrepresented after 200 examples?
If my initial collection leaves one label significantly underrepresented (specifically, if any label accounts for less than 20% of the dataset), I will specifically look for posts/comments under that label to balance out the distribution.

## Evaluation Metrics
Since discourse classification is subjective, overall accuracy alone is not enough. A model could achieve high accuracy by mostly predicting the most common class while performing poorly on important categories like analysis. To get a better picture of performance, I will use the following metrics:

* **Precision**: Measures how often the model's predictions for a class are correct. This is especially important for the `analysis` category because posts labeled as analysis should genuinely contain thoughtful discussion rather than simple complaints or reactions.

* **Recall**: Measures how many actual examples of a class the model successfully identifies. This is important for `reaction` and `hot_take` posts, since low recall would mean the model is missing content it was designed to detect.

* **F1-Score**: Combines precision and recall into a single metric. This will be the primary measure of success because it balances both types of errors and prevents strong performance on one class from hiding weak performance on another.

* **Confusion Matrix**: Shows which classes the model commonly confuses. For example, if `hot_take` posts are often labeled as `analysis`, it suggests the model struggles to distinguish between genuine analysis and posts that only appear analytical. This helps identify areas for improvement in the labeling guidelines and prompt design.

## Definition of Success
For this classifier to be considered a functional tool for community moderation or content discovery, it must move beyond simple statistical accuracy and demonstrate that it can navigate the nuance of human discourse.

* **Performance Target**: I define "good enough" for initial deployment as an overall F1-Score of > 0.70. This baseline ensures the model is not merely defaulting to common patterns but is actively learning the semantic boundaries between the labels.

### The Deployment Threshold: 
Accuracy on analysis is the critical path for a "real" community tool. For this classifier to be truly useful, it must meet the following conditions:

* **High Precision for analysis**: I require a precision score of at least 80% for the `analysis` label. If this tool were used to highlight "High-Quality Discussion," a low precision would frustrate users by flooding the feed with incorrect, surface-level takes. I am willing to tolerate lower recall for analysis (meaning the model might miss some good posts) in exchange for high precision (meaning the posts it does flag are almost always high-quality).

* **Failure Tolerance**: It is acceptable for the model to occasionally confuse a `hot_take` with a reaction—as both serve as "noise" in a quality-focused filter—but it must strictly avoid flagging pure, emotional reaction posts as analysis.

* **Meaningful Improvement**: If the classifier consistently provides a better filter than a simple keyword-based approach (e.g., searching for "because," "since," or "therefore"), it provides a tangible service to community members who are looking to bypass surface-level venting in favor of constructive strategic discussion.

## AI Tool Plan
### Label Stress-Testing:
To test whether my labels were clear and consistent, I asked Gemini to generate 8 comments that sat between two categories. When I classified them, I initially mislabeled 3 of the 8 comments, showing that my original definitions were not precise enough.

To reduce ambiguity, I created three simple labeling rules:

* **Numbers Don't Equal Analysis**: Mentioning statistics, item names, or patch versions does not automatically make a post analysis. If the information is being used to support an emotional complaint rather than explain a game mechanic, the post is still a `hot_take`.
* **Role/System Complaints Are Hot Takes**: Posts attacking a specific role (such as Jungle or Support) or game system (such as matchmaking or bounty mechanics) are always `hot_take`. The reaction label is reserved for posts about the player's personal experience or emotions.
* **Analysis Requires Cause and Effect**: A post is only `analysis` if it clearly explains how one game mechanic causes another outcome. Posts that make claims without explaining the reasoning behind them are classified as `hot_take`.

### Annotation Assistance
To balance efficiency and accuracy, I will use both AI (Claude) and human review when labeling the dataset. The 200 comments will be divided into two groups:

* **Pipeline A (AI First)**: For the first 100 comments, Claude will generate initial labels in batches of 25. I will then review every label and make corrections based on my decision rules.
* **Pipeline B (Human First)**: For the remaining 100 comments, I will label them myself. Claude will then act as a quality-checker and flag any labels that may not follow my rules.
* **Tracking Labels**: The dataset will contain a collumn `pre_labeled` and it will be true or false whether the AI labeled it first or not.
* **AI Transparency**: In the final report, I will include an AI usage section explaining this two-step verification process and emphasizing that all final labels were reviewed and approved by a human.

### Failure Analysis
After evaluating the model, I will review all misclassified comments to understand where it struggles. I will provide the incorrect predictions to Claude 3.5 Sonnet to identify which labels are being confused and suggest possible reasons. I will then manually verify these patterns by looking for shared characteristics, such as word choice, tone, or comment length.

The findings from this error analysis will be included in the final report to explain the model's limitations and highlight the challenges of classifying different types of community discussions.

## Examples that were difficult to label
1. Row 168 — "I hate season 11" (labeled `hot_take`)
The post starts with strong emotion, which made me consider `reaction`. But `reaction` should respond to a specific event, and this doesn't. Instead, the author argues that the game's meta rewards passive play and punishes roaming. Since those claims are opinions without evidence, I labeled it `hot_take`, although it was a close call.

---
2. Row 173 — Jaksho/Heartsteel rant (labeled `reaction`)
This almost became `analysis` because the author explains why healing is stronger, saying tanks and bruisers stay alive long enough to trigger more healing. However, the post is mostly based on one personal game and ends as a frustrated rant rather than a broader analysis. I kept it as `reaction`, but this was the label I was least confident about.

---
3. Row 152 — ADC power vs. agency (labeled `analysis`)
The author makes a clear distinction between power (damage) and agency (impact on the game), giving the post a structured argument. Even though it doesn't include statistics or other evidence, the reasoning is organized enough that I labeled it `analysis`. Still, `hot_take` would also have been a reasonable choice.