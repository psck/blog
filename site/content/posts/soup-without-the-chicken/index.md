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

So most setups copy the human process onto the agents, with roles, a scrum master, and a Jira board (sic!) to report to. That is eating soup with a fork. Maybe the recipe is fine and we just need new cutlery—and [Anthropic's approach to SDLC](https://claude.com/resources/articles/the-ai-native-sdlc-playbook), maybe combined with OpenSpec, which I'm a big fan of this week, surely looks like one.

The second is information and memory management—the chicken in this soup. Take it out and what remains is hot water with good intentions, however many pipelines you simmer around it. The SDLC is one big state problem—code, commits, and decisions—yet every new agent session starts from zero, and stuffing in more tokens won't fix what no prompt can hold. Maybe it was my choice of talks, but structured memory and token economics showed up only in homeopathic doses, with one fantastic exception: [Vitalii Ratushnyi's session](https://vitalii.connectingdots.fyi/memory), which cut straight through the hype.

## Putting the Chicken Back in the Pot

Vitalii started with a comparison we all repeat: *the context window is RAM, and memory is the disk*—and then spent the rest of the talk showing why that's not quite true. Real RAM reads the same at every address; a context window rots in the middle. On the [ContextArena](https://contextarena.ai) 8-needle benchmark and in his own evals, recall stops improving around 400k tokens while the bill keeps climbing, which makes a million-token window mostly a brochure number. And unlike a real disk, nothing pages in by itself until you build the retrieval. Even the smartest model on the market still wakes up on Tuesday with no idea what you decided on Monday.

How much does that missing state actually cost? The [Prime Agent paper](https://arxiv.org/abs/2608.23552) puts a number on it. On ARC-AGI-3, the exact same model weights jump from 30.2% in the standard harness to 95.5% in Prime Agent (right at the 95.4% human baseline) simply because the harness keeps structured state on disk instead of dropping it between turns. Same brain, different kitchen.

And no, "just slap RAG on it" is not the recipe either. Chunking a transcript and ranking embeddings by cosine similarity works fine for fuzzy prose, but software engineering runs on exact identifiers. Swap two characters in a commit hash and a vector search will happily serve you the wrong one first. Similarity is a ranking, not memory.

What actually works looks much closer to how good engineers already run a repo:

- **Handoffs instead of blind compaction.** Waiting for a 1M window to auto-compact means letting a tired model guess what mattered after its recall has already faded. A deliberate [handoff](https://github.com/mattpocock/dictionary-of-ai-coding) is a short, structured briefing written for the next session while the agent still remembers *why*—a shift note for the next cook, not a blurry summary of the whole chat.
- **Plain-text memory next to the code.** Instead of hiding state in a black-box database, Vitalii's [`/init-memory-bank`](https://gist.github.com/MisreadableMind/e0e0e4cf) command scans an existing repo—manifests, Git log, CI, linters—and builds a `.memory-bank/` folder in plain Markdown. A small core (`index.md` and hard rules, under 10k tokens) loads on every session; the rest is fetched only when needed. Because every memory write lands between turns as a versioned diff, a bad lesson is one `git revert` away instead of a database migration.
- **Three shapes of state, not one.** Most memory tools only store flat **facts** (*what holds right now*). Without a **timeline** (*what happened in order*), a reversed decision looks live and the next session happily repeats a fix you rejected yesterday. Without a **graph** (*why a rule exists and what it broke*), nobody knows if a constraint can be safely changed. And when you pull from that bank, small fast models like [Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev) can classify and pick lines verbatim in milliseconds—so a commit hash survives intact instead of getting paraphrased by an LLM.

A model should fail because the task was too hard, not because the harness forgot what happened ten minutes ago.

Right now, everyone brings a pot of soup to the demo. Almost nobody brings the chicken.

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
