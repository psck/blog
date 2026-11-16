---
title: "Soup Without the Chicken: Notes from Google Cloud Summit"
date: 2026-11-11
draft: false
tags: ["AI Agents", "Google Cloud", "Software Delivery", "Memory"]
summary: "Software delivery across enterprise organizations looks a lot like a crowded kitchen where dozens of engineering teams are independently trying to cook the exact same pot of soup."
---

Conferences are good for tracking vendor roadmaps and better for checking keynote slides against operational reality (which tends to arrive late and uninvited). After spending the day comparing architectures, sitting through technical sessions, and trading notes in the hallways, one pattern kept coming back.

## Everyone Is Cooking the Same Soup

Software delivery across enterprise organizations looks a lot like a crowded kitchen where dozens of engineering teams are independently trying to cook the exact same pot of soup.

By now most of us agree on what belongs on the table. It's so familiar that nobody bothers to read the recipe aloud, yet plenty of organizations have not put even half of it in place. Others skip the standard way and build a proprietary platform from scratch, convinced that one more bespoke orchestration layer will finally put developers in flow. And even with the basics on the stove, the way every decent chicken soup begins, is it enough to beat instant ramen?

I saw at least four presentations by good and inspiring speakers, all telling how much fun it was to cook things on their own. Every demo was a toy, a game-like greenfield project, and the agents built it alone, so it looked like magic (it did to me too, in the beginning). Now take the same thing into a real company, where nobody tolerates a mistake, and the first day of the new platform gets the same verdict from the whole floor. *"This doesn't taste right."*

I believe there are two reasons for that. The first is that we keep serving grandma's dish, the old Agile manifesto of people and interactions over tools and processes, in a kitchen where the role of blender and human suddenly got reversed.

![Look at my jira](assets/LookAtMe.jpg)

So most setups copy the human process onto the agents, with roles, a scrum master, and a Jira board (sic!) to report to. That is eating soup with a fork. Maybe the recipe is fine and we just need new cutlery, and [Anthropic's approach to SDLC](https://claude.com/resources/articles/the-ai-native-sdlc-playbook), maybe combined with OpenSpec (which I'm a big fan of this week), surely looks like one.

The second is information and memory management, and for me that is the chicken in this soup. Take it out and what remains is hot water with good intentions, however many pipelines you simmer around it. The SDLC is one big state problem of code, commits, and decisions, yet every new agent session starts from zero, and stuffing in more tokens won't fix what no prompt can hold. Maybe it was my choice of talks, but this topic was present only in homeopathic doses, with one fantastic exception: [Vitalii Ratushnyi's session](https://vitalii.connectingdots.fyi/memory), which cut straight through the hype.

## Putting the Chicken Back in the Pot

Vitalii started with a line we all repeat: *the context window is RAM, and memory is the disk (not quite)*. That *"not quite"* is where the whole talk lives. Real RAM reads the same at every address, while a context window rots in the middle. On the [ContextArena](https://contextarena.ai) benchmark and in his own evals, recall stops improving around 400k tokens while the bill keeps climbing, so a million-token window is mostly a brochure number. And unlike a disk, nothing pages in by itself until you write the retrieval. Even the smartest model on the market still wakes up on Tuesday with no idea what we decided on Monday.

The [Prime Agent paper](https://arxiv.org/abs/2608.23552) puts hard numbers on how much that costs. On ARC-AGI-3, the same model jumps from 30.2% in the standard harness to 95.5% in Prime Agent (right at the 95.4% human baseline) simply because the harness keeps its state on disk instead of dropping it between turns. Same weights, different kitchen.

I also liked how quickly he killed the usual "just slap RAG on it" answer. Chunking a chat log and ranking embeddings by similarity works fine for fuzzy prose, but code runs on exact names and hashes. Swap two characters in a commit hash and vector search still ranks it first, which means a near miss is a miss. Similarity is a ranking, not memory.

What works instead is surprisingly simple and lives right next to the code. Instead of waiting for a huge window to auto-compact when the model has already forgotten half the context, you use a deliberate [handoff](https://github.com/mattpocock/dictionary-of-ai-coding): a short, structured briefing written for the next session while the agent still remembers *why* it made a choice, like a shift note for the next cook rather than a blurry summary of the whole chat. And instead of a black-box database, a single [`/init-memory-bank`](https://gist.github.com/MisreadableMind/e0e0e4cf) command scans the repo and sets up a `.memory-bank/` folder in plain Markdown. A small core of rules (under 10k tokens) loads on every run, the rest is read on demand, and every update is just a text diff you can revert in Git when the agent learns the wrong lesson.

The trick is keeping three things separate in that bank: the current **facts**, the **timeline** of what changed, and the **graph** of why a rule exists. Most tools only store the first one, so a decision you reversed yesterday looks new today and the agent repeats the same mistake. Add a small, fast model like [Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev) to pick the right lines word for word instead of paraphrasing them, and your commit hashes stay intact.

A model should fail because the task was too hard, not because the harness dropped the state. Right now, everyone brings a soup to the demo. Almost nobody brings the chicken.

## References

- Slide deck, "Agent memory in a nutshell: do's and don'ts" (Lisbon AI 2026): [vitalii.connectingdots.fyi/memory](https://vitalii.connectingdots.fyi/memory)
- Init command gist: [gist.github.com/MisreadableMind/e0e0e4cf](https://gist.github.com/MisreadableMind/e0e0e4cf) (also on slide 08)
- Handoff definition, dictionary of AI coding: [github.com/mattpocock/dictionary-of-ai-coding](https://github.com/mattpocock/dictionary-of-ai-coding)
- GitHub: [github.com/MisreadableMind](https://github.com/MisreadableMind)
- LinkedIn: [linkedin.com/in/v-ratyshnyi](https://linkedin.com/in/v-ratyshnyi)
- Benchmark: ContextArena, MRCR 8-needle leaderboard, [contextarena.ai](https://contextarena.ai)
- Paper: Prime Agent, [arxiv.org/abs/2608.23552](https://arxiv.org/abs/2608.23552) (as listed on the deck)
- System architecture: [typesafe.ai](https://typesafe.ai/blog/introducing-system-one-models-and-jev), System One models and Jev
- Harmix post on memory layers: [manager.harmix.ai/blog/spent-3000-on-memories](https://manager.harmix.ai/blog/spent-3000-on-memories)
- Video breakdown of Prime Agent: [youtube.com/watch?v=RbglwViPqcY](https://youtube.com/watch?v=RbglwViPqcY)
- Presenter: Vitalii Ratushnyi, Lead Research Engineer, Harmix.ai
