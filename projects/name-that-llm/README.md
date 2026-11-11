<p align="center"><a href="https://www.kaggle.com/competitions/name-that-llm"><img src="assets/kaggle-banner.png" width="560" alt="Official Kaggle artwork for Name That LLM"></a></p>

# Name That LLM

Could I tell two versions of Gemini Flash apart from a single paragraph? In one development run, my classifier confused Gemini 3.7 Flash and 3.8 Flash 32 times. That pair helped shape how I approached Kaggle’s *Name That LLM*, a competition to identify which of 18 language models wrote a passage. I built a classifier that first picks one of six providers, then follows a small tree of decisions within that provider. [Competition](https://www.kaggle.com/competitions/name-that-llm)

My first provider-first ensemble scored **0.619 balanced accuracy on the public leaderboard**. Balanced accuracy gives each model equal weight by averaging its recall. I used that metric throughout because a classifier could otherwise gain ground on a common model while getting worse on a rare one.

## What I could learn from writing style

I started with 2,699 labeled passages responding to 150 prompts. I kept whole prompts together when splitting the data. If answers to the same question appeared on both sides of a split, shared vocabulary could make the attribution task easier than it would be on a new question.

| Input to a linear SVM | Balanced accuracy on held-out prompts |
|---|---:|
| Character sequences from the opening sentence | 0.134 |
| 220 linguistic features from the full passage | 0.233 |
| Character sequences from the full passage | 0.311 |

The full paragraph gave a simple character model a substantial advantage. I kept a lexical classifier in the ensemble and added explicit measurements of punctuation, function words, sentence rhythm, and how the opening differed from the ending. The 220-feature baseline used Lee and Lee’s Linguistic Feature Toolkit. The later tree uses its own smaller passage-style extractor. [LFTK paper, BEA 2023](https://aclanthology.org/2023.bea-1.1/)

## From a provider to an exact model

I used public task-trained Qwen adapters to extract evidence from a passage, then fitted a router and specialist classifiers around those representations. To design the branches, I measured which labels the development model confused with one another. That gave closely matched pairs, including the two Gemini Flash versions, their own decision. Each of the 12 splits has a ridge classifier trained on the labels on either side of that split.

![The six-provider decision tree and its 18 model labels. Google has a separate branch for Gemini 3.7 Flash versus 3.8 Flash.](assets/provider-tree.png)

One experiment helped me decide how much authority to give a specialist. I trained extra classifiers on punctuation, function words, and paragraph openings and endings for two difficult pairs. Overriding the tree on those pairs fixed **23** previously incorrect predictions and broke **26** correct ones. I left the overrides out.

## Where the tree helped

I compared the first provider-first model with the specialist tree on two separate panels of 15 prompts each. Each prompt had responses from all 18 models. I replayed each panel with 18, 9, 3, or just 1 response per prompt available at inference. The classifier inferred related passages from their text.

| Evaluation | First provider-first model | With the specialist tree |
|---|---:|---:|
| Reserved panel, weighted mix | 0.653 | **0.668** |
| Confirmation panel, weighted mix | 0.660 | **0.670** |
| Confirmation panel, all 18 responses available | 0.674 | **0.696** |
| Confirmation panel, one response available | **0.656** | 0.644 |

The last two rows explain why I track the mix. Comparing answers to the same question helped the tree distinguish writers. Given a lone passage, the earlier model did better. For the weighted rows I assigned 15%, 25%, 30%, and 30% to the four group sizes. Kaggle keeps the hidden mix private. Both panels had also informed earlier baseline work, so I treat the improvements as local comparisons.

## The 27B experiment

I kept the tree topology and replaced the five primary 8B adapter passes with one pass through a public task-adapted Qwen3.8-27B. Every specialist can reuse that pass. A separately fitted classifier on Qwen3-Embedding-8B contributes a second set of model scores. The offline Kaggle notebook fits the downstream heads and writes one exact model label per test ID.

The released 27B adapter had already trained on all 2,699 official passages. Holding out some of those passages afterward would give a misleading estimate of the adapter’s performance. The 0.619 leaderboard result above belongs to the earlier ensemble. [Original 8B adapter by prvsiyan](https://www.kaggle.com/datasets/prvsiyan/ntl-qwen3-8b-lora24-two-stage-adapters/versions/4) · [27B inference release by Vladislav Lavrentev](https://www.kaggle.com/code/lightningv08/0-681-qwen3-embedding-lora-qwen3-8-27b)

Competition artwork courtesy of Kaggle. [Original banner](https://www.kaggle.com/competitions/170484/images/header)
