---
layout: post
title: How far does a small model get? Porting Java to Python with Haiku
date: 2026-11-12
description: "A weekend benchmark: how far does Claude Haiku get alone at porting Java to Python, and how much does a good harness add?"
tags: llm coding-agents claude-agent-sdk benchmark
categories: ai
thumbnail: assets/img/2026-11-12-haiku-java-to-python/haiku-harness-chart.png
giscus_comments: false
related_posts: false
---

I run a lot of my agentic coding on local models on my DGX Spark, and open-weight models and open-source harnesses such as pi or opencode keep getting better. But my eureka moment came earlier, in December 2025, when I first used Claude Opus 4.5. Wow, this is not just a fancy autocomplete! It gets things done and made projects possible that I couldn't have done before. For example, I had it write the same 2D Ising model Monte Carlo simulation in about 20 languages. Since then I've been paying for the Claude Pro plan.

With Claude, I mostly use the big models, Opus or Fable, and sometimes Sonnet. I never bothered with Haiku, the cheapest one. But on agentic projects, Opus and Fable burn through the Pro plan's weekly limit fast. If nobody else pays for the tokens and I don't want to wait for the reset, I either pay big bucks myself or give Haiku a chance. Opus costs 40 times as much per token!

So I had two questions: how far does Haiku get on its own? And how much does a good harness add? By harness I mean everything around the model: instructions, tools and checks ([Birgitta Böckeler's article](https://martinfowler.com/articles/harness-engineering.html) is a good introduction). To find out, I spent a weekend letting Haiku port small Java repos from [RepoTransBench](https://github.com/DeepSoftwareAnalytics/RepoTransBench), a research benchmark, to Python.

Software migration was new territory for me. I've done plenty of refactoring, read Fowler's *Refactoring*, and I follow the boy scout rule: leave the code in better shape than you found it. But I had never moved a codebase to another language.

The setup was Claude Code, driven by the Claude Agent SDK, with Haiku 5.5. The agent gets a Java repo plus Python tests to check its work against, and it's graded on a second set of hidden tests it never sees. My first exploratory runs surprised me: Haiku migrated repos of a few hundred lines of code (LOC) on its own, all tests green. That left nothing for a harness to improve. So I added bigger ones. In the end I used six repos, from a few dozen to about 2,400 LOC. Results vary from run to run, so every setup ran three times: about 90 measured runs, and about 16 dollars of credit in total. That fit within my plan's monthly Agent SDK credit.

Two lessons came before any number meant something: First, isolate the agent. On an early run it searched my whole disk (`find /`) and found the dataset, hidden tests included. Since then every run gets its own Docker container.

Second, test the tests. Many of the benchmark's Python tests are AI-generated and were never run against real code. By the benchmark's own scoring, 47 of its 146 Java-to-Python tasks get full marks with an empty implementation, and some tests contradict the Java code they're meant to check. So every repo had to score zero without code, and I dropped two whose tests contradicted the Java source. Still, garbage in, garbage out: read the numbers as hints, not proof.

On its own, Haiku passed about 75 % of the hidden tests, at about 4 cents per run. I honestly didn't expect that from the smallest model. Where it failed, it was mostly missing pieces rather than wrong logic: classes it wrote but never exported, or a helper that no visible test calls.

The code itself wasn't very pythonic. It followed the Java structure closely: same files, same functions, same if/else chains. In one repo, the Java code leaves the JSON output to the Gson library. Haiku rebuilt Gson's escaping and Java's 32-bit float formatting by hand, almost 50 lines of it:

<details markdown="1">
<summary>Haiku's hand-rolled Java-style float formatting</summary>

```python
def _to_f32(x):
    try:
        return struct.unpack("f", struct.pack("f", x))[0]
    except OverflowError:
        return float("inf")


def _float_to_string(value):
    """Mimic Java's Float.toString for a non-negative value."""
    ...
```

</details>

Feed it `123456789.12` and you get `1.2345679E8`, exactly like Java. A pythonic port would use `json.dumps` and print `123456789.12`. In another repo, Java's `equals()` never became `__eq__`, so `==` returned `False` for two equal objects.

Python's flexibility allows that: object-oriented, procedural and functional code are all valid, so a 1:1 port runs right away, even if it looks ugly. It's also the opposite of the boy scout rule: a faithful port carries all the litter over. Cleaning up is a second step, refactoring under tests, as Fowler describes it. Maintaining code like this won't be fun for a human. If agents end up doing that job, I doubt that clean-code rules made for human readers apply to them. Maybe they are totally fine with spaghetti code?! Time will tell.

The biggest effect came from defining precisely what "done" means. A parser, not a model, lists every Java class and method, and the agent has to say where each one lives in Python, or why it isn't needed. That way nothing gets forgotten. With only tests as the goal, Haiku may stop once the visible ones are green and skip whatever they don't cover. On the hardest repo, the pass rate jumped from 31 % to 69 %.

Two more ideas added little: an automatic check that sends the agent back if anything is unmapped or a visible test fails, and Sonnet as a reviewer with a fresh pair of eyes. A check can only enforce what it can see. The reviewer found plausible edge cases, just not the ones the hidden tests check: 0.7 percentage points for 2.5 times the cost.

Taken together, the harness took Haiku from about 75 % to about 82 % of the hidden tests. That's roughly where Sonnet lands without any harness (83.5 %), at about 12 instead of 20 cents per run. With six small repos that's a hint, not proof, and most of the gain came from one repo.

{% include figure.liquid path="assets/img/2026-11-12-haiku-java-to-python/haiku-harness-chart.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Share of hidden tests passed against average cost per run: Haiku alone, Haiku with each harness step, and Sonnet alone" caption="Share of hidden tests passed against average cost per run, over six repos with three runs each. Each Haiku step builds on the one before." %}

So, back to my two questions. On its own, Haiku gets surprisingly far: about three quarters of the tests for a few cents. With a precise spec around it, about as far as Sonnet.

But this was a toy experiment. A project like this makes sense when there's a real reason to leave Java, when the code is small and well bounded, and when you can check its behavior. It doesn't when there are no tests and no way to run the old system. A real migration also needs more than passing tests: run the old and the new code side by side on real inputs and compare. The port should even keep the old bugs and undocumented behavior! Making it pythonic comes after that. Haiku does the typing for a few cents. Checking the results is the real work, and it takes domain knowledge.
