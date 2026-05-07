---
layout: post
title:  "A vision of AI for data work"
date:   2026-05-07 05:07:59 -0500
categories: jekyll update
---

## Introduction

Let’s imagine you’re a product manager at Slack. You work on a feature, like [Slack canvases](https://slack.com/features/canvas), and you want to plot its daily active users, broken out by some dimension, like platform (iOS, Android, desktop, etc.), over time. (You hope the various lines are going up.)

Previously, you might have had to find a relevant dashboard or to ask a data scientist. Now, you can get an answer (if not *the* answer), basically immediately. You have Claude Code set up with an MCP that allows it to search schemas and tables in the data warehouse, and to run queries. You task it with this problem, and it goes off, finds what it thinks is relevant information, and returns a result.

Setting aside the question of whether the answer is correct, for now, one question is: Is this a good way to do work?

From your perspective, things are faster. Work feels more productive. Claude can massage the data into the format you want, or visualize it exactly the way you want. The experience is interactive and tailored in a way that dashboards aren’t. You could ask it to include other dimensions, or filters, and it will update its work. You can spend tens of minutes, even hours, asking it questions about the data, and it will have all the answers.

Perhaps you could have arrived at the same result working with your DS partner, but they have a large stack of other things to do, and it would have required many rounds of back and forth to align your vision with theirs. The dream has always been self-serve analytics, right? Now it’s finally here. (Plus, while Claude is thinking, you can ~~do other productive things~~ surf Bluesky.)

The point of this essay is to cast (a lot of) doubt on this way of working. It’s not so much that Claude would give you the wrong answer (although it might). It’s that this kind of approach doesn’t scale, and also doesn’t take advantage of AI’s unique capabilities. Here’s my thesis: AI “skills” and workflows for data should chain together **deterministic, accurate** tools. An MCP that allows an agent to write essentially arbitrary SQL is not such a tool. 

## Building agents for analytics

You might argue that “An MCP that allows an agent to write essentially arbitrary SQL” is a strawman. Companies don’t really do this. They try to build guardrails to prevent agents for analytics/data from producing garbage. Here’s a [typical example](https://openai.com/index/inside-our-in-house-data-agent/), from OpenAI (”Inside OpenAI’s in-house data agent”).

OpenAI’s data agent incorporates several sources of context. It uses table schemas and metadata (column descriptions, etc.). It "ingest[s] historical queries” to understand common usage patterns. It tries to understand how the table was built in the first place (what data pipeline produced it, what filters were applied, what data sources were consumed). It uses “human annotations”: “curated descriptions of tables and columns provided by domain experts, capturing intent, semantics, business meaning, and known caveats that are not easily inferred from schemas or past queries”. It can even layer in “institutional knowledge” from places like “Slack, Google Docs, and Notion”. The authors write

> Together, these layers ensure the agent’s reasoning is grounded in OpenAI’s data, code, and institutional knowledge, dramatically reducing errors and improving answer quality.
> 

I don’t doubt that an agent with such guardrails is (much) better than an agent without them. But one obvious question is: why do we need an agent to write SQL to answer this kind of question (”canvas DAU by platform”)? (In OpenAI’s blog post, the example was “ChatGPT WAU over time”, which feels even more trivial.)

Isn’t having a metric broken out by a dimension a well-understood, essentially solved problem in analytics? We build fact and dimension tables, have a semantic layer on top of them, and then write dead simple SQL on top of that. LookML allowed us to do this years ago.

The flexibility of agents is that, in theory, we no longer need to build fact and dimension tables (setting performance aside). We can have an agent generate 200 line long SQL queries that stitch together a bunch of different data sources, like a human would if they were writing SQL from scratch. And if we need to add another dimension, or another filter, we ask the agent to revise its query, and it turns the 200 lines into 250 lines. Business logic lives inside the sources of context mentioned above, instead of inside a dbt pipeline that an analytics engineer would build.

OpenAI has "600 petabytes of data across 70k datasets”. They seem to have decided that the way to corral this complexity is with an agent that can tie together disparate sources of context and write essentially arbitrary SQL across an ever-growing number of tables.

But another solution is to revisit data engineering principles. The dataset is source of truth. The context that would otherwise be needed by the agent is largely embedded inside it. The dataset *itself* is the kind of “deterministic, accurate tool” I discussed above.

In this vision, AI’s role becomes different. It is to assist the building of these tools, these sources of truth. And it is to turn natural language questions into straightforward queries that can be run on top of them. AI is simply an interface for a non-technical person to access and use something that is trusted. AI does not need vast gobs of context required to make the result trustworthy.

(These two visions aren’t mutually exclusive, either. [Block](https://engineering.block.xyz/blog/building-the-data-foundation-for-automated-analytics) solved the problem differently than OpenAI did. The Block team built one tool (”Block Data MCP”) for “deterministic accuracy” using their existing semantic layer, and another tool (”Query Expert MCP”) for “exploratory flexibility” in writing bespoke queries. The problem is when the two tools are conflated and thought of as one.)

## Other applications

Once you see this pattern, you can’t stop seeing it. Would you rather have an agent calculate 2 + 2, or have it give that instruction to a calculator? Would you rather give it a csv and generate insights directly, or have it write a pandas script to do the analysis? Would you rather have it calculate a p-value itself, or call a scipy method? Etc.

Again, the issue is not so much whether you can get the right answer. I’m sure agents can add 2 and 2 together, and they can, apparently, even count the number of Rs in “strawberry” now.

Rather, the issue is that the “brute force” way of doing things (”just ask Claude”) doesn’t scale, and, moreover, it is very difficult to verify. It’s much easier to check the result from a semantic layer than it is to check a 300 line bespoke query. It’s much easier to check a 50-100 line Python script than read through a Claude code transcript that shows its “reasoning”.

I had a PM recently express frustration to me. She mentioned how our in-house experimentation platform didn’t have the dimensions she wanted to cut experiment results, and so she ended up asking Claude. Claude wrote a very large, several hundred line query, stitching together experiment exposures, metrics, and this dimension. Then it generated p-values for each of the individual sub-groups (no multiple comparisons correction, naturally). God knows if any of it is correct (in my quick skimming, it appeared not to be), but it has the virtue of looking plausible, and the writeup had all the unwarranted confidence you’d expect from an LLM.

The tool-based approach would be: let’s use the experimentation platform as the source of truth. Let’s not replicate the subtle filters, joins, and other logic necessary for experiment analysis in a giant Claude-written SQL query. If a certain dimension or cut doesn’t exist, let’s use AI to build that logic faster, multiplying the power of our engineers. AI is not a way to bypass the experimentation platform; it’s a way to make it better and easier to access.

## Closing thoughts

The advantage of AI is its flexibility. The disadvantage of AI is also its flexibility. It seems capable of doing these complicated tasks — pulling bespoke metrics, analyzing and cutting experiment results — and it won’t say no if you ask it to. But, as a data scientist, you should ask yourself! Are you using AI to answer a question that a deterministic, accurate tool would answer better? If that’s the case, you should invest your energy into building that tool, rather than relying on Claude alone.

Non-technical stakeholders might not even know that’s a question they *should* ask. They want an answer, and they want it fast. Engineers have the “gate” of PR review to help prevent bad code from being added to their codebases, but data scientists don’t have an analogous gate to shut out bad data and insights. It is incumbent on us (data scientists) to work with stakeholders, understand their needs, and shape their workflows to avoid these “bad paths” discussed above. We cannot simply tell people their way of working with AI are wrong; we need to offer an alternative that is just as convenient to use.
