---
draft: true
title: "Surviving the LLM Era as a Software Engineer"
date: 2026-03-04
---

I haven't written a line of code since the beginning of 2026. I'm still employed as a software engineer.

That's not entirely true — I've typed plenty of words into LLM prompts, reviewed generated code, and occasionally tweaked a line here and there. But if you asked me to point to code I *wrote*, in the traditional sense of staring at an editor and producing logic from scratch, I'd struggle to find any. Coding agents do that part now.

What I do instead is critique. An LLM proposes an approach, and I tell it why it won't work — the edge case it missed, the constraint it doesn't know about, the simpler solution it overlooked. A colleague pitches an idea, and I poke at the assumptions until we land on something that actually solves the business problem. The value I provide isn't in the typing. It's in knowing what's worth building and recognizing what's unsound. John Whitaker's essay [The Idea Is The Software](https://johnowhitaker.dev/essays/distributables.html) resonated with me for exactly this reason — when an LLM can turn any well-specified idea into working code, the idea *is* the hard part.

---

We had a hackathon at work last Friday. I walked around looking at what people built, and something struck me: many of the projects were essentially Claude Code skills. The entire creation process was writing steps for an LLM to execute. The LLM did all the heavy lifting — the scaffolding, the wiring, the implementation details.

What differentiated the good projects from the forgettable ones wasn't coding ability. It was the quality of the idea. Did you identify a real problem? Did you think through how it should work? Could you steer the LLM toward something coherent? The people who built the most impressive things weren't necessarily the strongest coders. They were the ones with the clearest thinking about what to build and why.

As Simon Willison puts it, [code is cheap](https://simonwillison.net/guides/agentic-engineering-patterns/code-is-cheap/). When code is cheap, the competition shifts to ideas.

---

So if you take coding ability away from a software engineer, what's left?

Maybe it's [system design](https://muratbuffalo.blogspot.com/2026/01/rethinking-university-in-age-of-ai.html) — understanding how pieces fit together, what trade-offs matter at scale, where the complexity actually lives. That's harder to automate because it requires holding a lot of context about the real world that doesn't fit neatly into a prompt.

Or maybe it's something more basic. As Nadh puts it: [code is cheap, show me the talk](https://nadh.in/blog/code-is-cheap/). Show me the prompt that produces the artifact. Show me the thinking process. Show me how you reasoned through the constraints and arrived at your conclusion. In a way, most software engineers in the US are massively overpaid for their coding abilities — but perhaps not for their ability to think clearly about hard problems.

Maybe that's what the job becomes. Thinking clearly, communicating precisely, and knowing enough about systems to ask the right questions. Until LLMs beat us at that too.
