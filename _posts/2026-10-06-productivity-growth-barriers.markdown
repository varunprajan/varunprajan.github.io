---
layout: post
title:  "What are the barriers to productivity growth at tech companies?"
date:   2026-10-06 05:07:59 -0500
categories: jekyll update
---

## Introduction

Anyone who uses AI for software work recognizes its potential to improve productivity. Some tasks that once took hours or days can now be accomplished in mere minutes. Model performance on benchmarks continues to improve at a rapid pace. AI also enables a mode of working where work can be done in the background, concurrent with other activities.

AI [“insiders”](https://tecunningham.github.io/posts/2025-10-19-forecasts-of-AI-growth.html) predict that these facts, along with improvements in AI model quality and usability, will translate into enormous increases in economic growth and productivity (3-30% annually, which translates into doublings, or more, of GDP over just decades)

Oddly, though, we haven’t seen these gains yet. (Which is not to say they won’t happen in the future.) Economics professor J. W. Mason [recently quipped](https://bsky.app/profile/jwmason.bsky.social/post/3mv76auof5s2k), “I feel like maybe before AI wipes out humanity and/or renders work obsolete, it might deliver a couple points of faster labor productivity growth.”

I am not an economist, and I am not inclined to make predictions about the future. But I did want to try to explore this question, as applied to my job (a product-focused data scientist): is AI truly making us more productive? What are the barriers to productivity growth? It’s useful to think about this topic at three levels: within task, within individual, and within firm.

## Within task

As mentioned above, AI can lead to huge speedups, of 2x or more, in certain individual tasks. I can tell Codex to turn a SQL query into a data pipeline, or to analyze why two different queries that should return the same results don’t, or to explore the data pipeline logic that populates a particular column in a table.

These tasks are not technically challenging, but they are tedious and time-consuming. They are also reasonably easy to verify — I can run the data pipeline myself, or I can understand the explanation for the discrepancy between queries. It is usually much faster to check an AI-generated answer than to write the code, or investigate the data, or read through the data lineage, myself.

Why might improvements to task-level productivity not translate directly into improvements to “overall” productivity? One important reason is that the **“atomic” task** is usually *not* something like “analyz[ing] why two different queries I expect to return the same results are actually different”. The actual task might instead be: “migrate a dashboard from ad-hoc queries to queries that use our semantic layer”. Analyzing discrepancies between queries is one part of that task. Other parts might be less amenable to automation, however. These include: having meetings with data engineers to understand how the semantic layer works, updating the dashboard (which might not yet have an MCP for agentic work), getting a review from a fellow data scientist, finding that the semantic layer doesn’t yet support X type of metric, working with the semantic layer team to fix this issue, etc. Crudely speaking, the more the task relies on other people, the less impressive the gains from AI become.

Another important reason is that AI can be used to do more **“side quests”**, not just to save time. Sometimes this improves quality of the final output, but quality is challenging to measure, and we might simply be wasting tokens. The net effect on task-level productivity can be small or even zero.

To take an example, suppose the task is to analyze why metric X is trending down. With AI tools, we can analyze that question more deeply than ever before. We can cut by more dimensions, generate more charts, churn out more slop text, and so on. While some of these additional cuts and charts and hypotheses might be useful, most won’t be. It actually requires a fairly proficient data scientist to figure out what, of the reams of information that AI produces, actually merits attention and contributes to a comprehensible narrative. For more junior data scientists, AI might actually drive negative productivity on tasks like these, and I’ve seen some evidence of this effect at my job. It’s easy to get lost in the interpretation of information you didn’t generate, given that generating such information is now trivial.

## Within individual

Ben Moll and Alex Imas, both professors of economics, wrote [a nice article](https://aleximas.substack.com/p/will-ai-soon-lead-to-double-digit) about the effects of AI on GDP and economic growth. They are skeptical of the thesis that AI will lead to double-digit growth rates for the economy at large. One of their arguments is that AI (indeed, automation in general) involves a reallocation of spending from high-productivity tasks to lower-productivity ones:

> In reality, when automation makes the automated thing cheap, people and firms do not buy proportionally more of it. Spending instead shifts to what is still scarce, the tasks still performed by humans. The automated part shrinks as a share of the economy and the human part grows. This is Baumol’s cost disease, or in the words of [Aghion, Jones and Jones](https://www.nber.org/papers/w23928): “growth may be constrained not by what we do well but rather by what is essential and yet hard to improve.”
> 

This statement also applies to the individual. Certain data science “sub-tasks”, like the ones mentioned above, have become cheap to do. This saves time. There are three ways to use the time that has been saved: to not work, to do more cheap (AI-amenable) tasks, or to do more expensive (human-only) tasks.

The middle option, doing more cheap (AI-amenable) tasks, is the “side quest” problem discussed above. We can code and analyze data a lot more cheaply than before, and it’s no surprise that internal dashboards, bespoke tools, data analyses, etc. have proliferated in the age of AI. There are probably drastically diminishing returns to these efforts, and I think the longer-term equilibrium is likely the last option: “do more expensive (human-only) tasks”. As Moll and Imas write, “the automated part shrinks...and the human part grows”. For data scientists, this means more time talking to product stakeholders and other cross-functional partners, more time thinking about what AI analyses mean and less time doing them, more time allocated to product-like tasks and less to engineering-like tasks. (Or, as I put it in another post, “more judgment and less technique”.)

Moll and Imas point out, rightly, that this shift incurs a cost. The “cost disease” is that as work shifts from higher productivity tasks to lower productivity ones, the productivity gains from automation are partially negated. This happens at the individual level just as at the level of the economy. AI is less capable of participating in conversations on Slack than it is at coding. So I’ll probably spend more time conversing on Slack, and less time coding.

One question is whether these additional tasks (regardless of whether they’re “cheap” or “expensive”) add value to the company. We talked about this problem above, with AI “side quests”. But it’s also true of non-AI, expensive “human” work. How much additional revenue will I make for the company by being more active in Slack conversations, or by thinking more like a PM, or by doing more proactive thinking?

Firms might find it easier to maximize revenue per headcount by cutting headcount than by using the time freed up by AI to do more stuff. In other words, depending on how drastic the diminishing returns to additional DS work are, reallocating the 20% of cheap time saved by AI to more tasks might not be worth it. It might make more sense, instead, to assume there’s only a finite number of useful things data scientists can be doing, and, if data scientists are X% more productive than before, we now need X% fewer of them. (I suspect X for data science is a lot lower than X for software engineering, fortunately for me.)

There are also productivity problems in the medium- and long-term that might not be obvious in the short-term.

One is that as I offload more cheap tasks to AI, I become less adept at doing these tasks myself. As I write less SQL, my ability to write SQL becomes worse; as I do fewer data investigations, I feel how rusty I am when doing them by hand. I’d mentioned before that the productivity-enhancing aspects of AI are mainly felt for tasks that are easy to validate. But my ability to validate has probably also become worse as my “feel” for the data has weakened, and as my technical muscles have atrophied. If, for 10-15 years, I invested in becoming technically adept, now I’m drawing down those investments, and perhaps somewhat rapidly. Individual contributors might be forced to spend time doing work themselves, if only to stay sharp. But of course that kind of “maintenance” takes away from productivity.

A second problem is that people with 10-15 years of experience will eventually retire (sooner rather than later, I hope). The entry-level people who will replace them will have spent most of their careers working with AI tools. It seems plausible that AI is most effective for people who know what they’re doing without AI. But there will be fewer and fewer of these people left as time passes. The people who do not know how to work without AI will need to reach a different equilibrium with these tools than the one that I reached.

## Within firm

Perhaps the biggest barriers to productivity are found at the firm level. In product-driven organizations, tasks like “migrating a dashboard to a new data source” and “writing data pipelines” do not directly drive revenue. Instead, what does is either shipping new products, or better versions of existing products.

The problems we discussed at the task and individual level compound at the firm level. Finishing an entire project often requires a lot of the “human part” and not as much of the “automated part”. As developer work becomes cheaper, I anticipate organizations cutting engineering headcount and increasing headcount in other areas, which will partially cancel out the savings from AI. At the firm level, there are even more side quests that AI will enable, with even more dubious effects on firm-level revenue and productivity.

There are also some unique challenges at the firm level. One that data scientists are particularly close to is experimentation. (Others include code maintenance and the need for organizational changes, but these are less relevant to data scientists.)

At Slack, feature development seems to be proceeding faster than ever before. It has become much easier to turn an idea into a working prototype, and then to production code that is experiment-ready. What hasn’t become faster, however, is experimentation. The laws of statistics still apply, and even with the various tricks we can play (CUPED, proxy metrics, etc.) we often have to run experiments for weeks or even months to power the primary metric -- to prove that the feature is indeed better.

To take a concrete example, albeit with made up numbers, if feature development used to take 8 weeks but now takes 2 weeks, but experimentation still takes 8 weeks, the time savings for engineers is 75%, but the overall time savings is only 37% or so. In fact, the situation can be even worse if experiments collide with one another and can’t be run in parallel. And there’s also the issue that, as mentioned before, each additional experiment is less likely to be successful than the average experiment, because of diminishing returns to feature development and in the quality of ideas. To summarize: there are several reasons that X% more features that are experimentation-ready might not translate to X% more features shipped, let alone to an X% gain in the primary metric, or X% more revenue for the company.

# Closing thoughts

There are several reasons that productivity gains for product development, broadly construed, might be much less than productivity gains for individual coding sub-tasks. These include:

- the fact that actual tasks involve non-coding, “human” components
- that time savings from AI can be reinvested in improvements to quality, but these improvements might be small or non-existent
- that time savings from AI can be reinvested in doing more work, but there can be severe diminishing returns
- that, in the medium- or long-term, reliance on AI might cause IC skills to deteriorate (skills like validating AI output)
- that experimentation does not see similar time savings or productivity improvements, and it might end up being the bottleneck for product development

The last bullet point is particularly interesting to me as a data scientist who has done a lot of experimentation. As I discussed elsewhere, experimentation in the B2B space is much more challenging than in the D2C space. Metrics are “lumpier”, and sample size constraints are much more severe. How should we think about changing experimentation practice, and perhaps even organizational structure, so that we don’t have dozens of engineers and engineering managers complaining that we (and our emphasis on statistical rigor) are blocking them from shipping all of this great new code they’re generating?
