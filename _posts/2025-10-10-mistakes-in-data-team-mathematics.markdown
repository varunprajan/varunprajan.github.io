---
layout: post
title:  "Mistakes in data team mathematics"
date:   2025-10-10 00:00:59 -0500
categories: jekyll update
---

## Introduction

Data scientists are self-interested, but they are also expected to serve the interests of the data team. These two goals conflict, sometimes in entertaining ways. In what follows, I will discuss a few examples. Each centers on a “mistake in mathematics”: in which the sums of costs and benefits that accrue to individuals are different than those that accrue to the broader organization.

(I’ve borrowed the phrase “Mistakes in X Mathematics” from one of my favorite books, [*Reasons and Persons*](https://en.wikipedia.org/wiki/Reasons_and_Persons), by Derek Parfit. “5 Mistakes in Moral Mathematics” is a lovely chapter that discusses how we might make mistakes when putting utilitarian thinking into practice, and how we should correct those mistakes.)

## Saving money

In a data org with “big” data, spending money is easy. You build pipelines to process data and to store it somewhere. You spend money both on processing power and storage, and, if your business is relatively successful, each increases as the number of customers grows. If the customer base grows linearly, storage grows quadratically (assuming an unlimited retention period).

As a corollary, saving money is also relatively easy: you simply turn off or destroy the things you once spent money on. Instead of producing a dataset with a retention period of 5 years, you change the retention to 1 year, or 90 days. Or you move “old” data into long-term storage, making it more annoying to query. Or you stop producing it altogether, because no one has used it recently. You can also make data easier to write, but harder to read. Instead of having the same table partitioned three different ways, for three different query use cases, you partition it only one way. In doing so, you’ve saved ~67% of the money you formerly spent to produce it.

At most companies I’ve worked at, many engineers are devoted to such cost savings. These projects are pursued because they directly impact gross margin, because the savings are relatively easy to quantify, and because implementation tends to be straightforward. Building a pipeline is the hard part. Modifying it or turning it off is easy. I’ll frequently see announcements in Slack channels to the effect of “we saved $300k (annualized) by doing X”. Data engineers are promoted on the basis of such projects, and such announcements. Everyone loves to see a promotion package with a dollar amount in it. If an engineer can so easily save the company hundreds of thousands or even millions of dollars, they’re basically paying for themselves, right?

We commit several mistakes of mathematics by accepting these $ estimates at face value. I don’t mean to cast aspersions on all cost-saving work: there are some projects that save money without degrading functionality (or, even better, they enhance it). But the more typical case is where *something* is sacrificed: data retention, ease of writing a query, query processing time — or even the ability to write the query at all.

Take the example where we generate savings by changing the partitioning scheme of the data. We eliminate redundant copies of the data, but we also make it more time-consuming to query. How should we evaluate costs and savings in this case? Simply looking at the savings side is naive. Among the queries that we were able to run before, some will take longer to run, and others might timeout. It would also be deceptive to add up the processing cost associated with queries on the new dataset, and compare it to the processing cost of corresponding queries on the old dataset. The different might appear negative, even if the cost, in the counterfactual sense, has gone up. (Take the extreme case, where queries on the new dataset are impossible to run — eventually, everyone will stop using that dataset, and costs will appear to be zero.)

This kind of “[censoring](https://en.wikipedia.org/wiki/Censoring_(statistics))”, where we can’t measure what we’re missing out on, is all too common. By not having data that goes back years, we can’t do long-term analyses of our business or of past experiments. Any analyses we run will be based on shorter-term data, with its attendant bias. This is a real cost, one that will show up nowhere in the AWS Cost Explorer reports. Think about it the other way around: a business would spend money to unlock a capability that allowed it to make better decisions. So why isn’t the same logic applied when it saves money to diminish or destroy such capabilities?

It’s also worth noting that analyses of long-term data tend not to be run regularly: they tend to be costly, both in terms of processing power and people power. Data engineers who rely on usage reports to determine which datasets are “valuable” or not are probably deceiving themselves. A dataset that is used once to uncover a counterintuitive finding is probably more valuable than that one that is used daily to power a basic dashboard.

The last point, which is probably obvious, is that the costs and benefits of cost savings are borne unequally by different parties. Data engineers — the data producers — can easily claim credit for the savings. But data scientists, analysts, and product managers — the data consumers — may find it difficult to devise their own accounting. How should we try to measure the value of analyses we *might have* pursued, and the decisions we *might have* driven, if we had something we no longer have?

## Grunt work

There are two kinds of projects in data orgs: the ones that will get you promoted, and the ones that won’t. I’ll call the former category “promo work”, and the latter category “grunt work”. Those reading this post probably already know what promo work entails. Conducting an analysis that gets read by someone in the C-suite. Changing the culture of how experiments are run in the broader organization — e.g., to measure cumulative impact. And so on.

It’s work that has both “scope” (beyond your immediate squad) and “impact” (preferably of the dollars and cents variety). It makes sense, of course. If you want to be a staff or principal level IC, your influence should extend beyond the confines of your squad. And, as such a contributor, you probably won’t be doing many small-scale, low-impact projects, so why should your promotion package be evaluated on the basis of such projects?

That being said, the mathematics of grunt work and promo work has always struck me as slightly questionable. There are two related problems.

First is the “last touch attribution” problem. Doing a groundbreaking analysis requires high quality data, good data tooling, and time. It is often the case that the person taking credit for the analysis did not build the dataset, or the data tooling. These people rarely get credit. In some sense, this is “[last touch attribution](https://mountain.com/blog/last-touch-attribution/)”: the person who puts the shiny bow and wrapping paper on a present is credited as the thoughtful bearer of gifts. All the other people involved are less acknowledged. (By no means is this me complaining about having to do grunt work so others can get promoted. In fact, I think I’ve more often been the beneficiary of this dynamic.)

We should try to change the mathematics of credit and blame in data orgs. You should get credit for discovering problems with data quality, or helping to build data tools, or making a really nice-to-use, clean dataset. Of course, the pendulum can swing too far the other direction, where everyone tries to develop tools that they want others to use. But my point is that we should be more precise about apportioning success. If you couldn’t have done your project without others, that should also count towards *their* promotion packages, or at least their bonuses.

The second problem is the “categorical imperative” problem. (Slightly ironic, I suppose, since I mentioned utilitarianism earlier.) [We should](https://en.wikipedia.org/wiki/Categorical_imperative) “act only according to that maxim whereby you can at the same time will that it should become a universal law.” There is often the weird dynamic in data teams where, because of the incentives provided for promotion, no one wants to do grunt work, and everyone wants to promo work. After all, the former doesn’t really help make your promotion case. (One of my VPs once said something like, “you should say no to 80% of the work given to you.”)

The problem is that this doesn’t really work as a “universal law”, or at least it doesn’t work well. It’s difficult for *everyone* on a team to be 100% on promo work, and 0% on grunt work, but any other [equilibrium](https://en.wikipedia.org/wiki/Nash_equilibrium) feels unstable, from the individual perspective. I firmly believe that some grunt work, though certainly not all, is socially valuable. Documenting how you set up your local environment, or helping a new colleague get acquainted with the datasets relevant to them, or finding a bug in the setup of a particularly gnarly experiment, are all useful activities for a data team. Without grunt work, data teams become less efficient and more prone to making mistakes. Data takes longer to find, is less easy to use, and is less trustworthy. But, to reiterate, the individuals on a team are not necessarily incentivized to make the team, as a whole, function better.

## Greener pastures

Perhaps the most embarrassing realization I had in tech is that staying at a company too long is frowned upon. If you’re good, you’ll eventually leave for somewhere better. So if you stay, that probably implies that you’re not good enough to leave, or at least that you’ve lost the desire to pursue that goal. My father has been happily working at his one and only “real” job (university professor) for 42 years now. I still haven’t yet hit 4 years anywhere.

That everyone in tech is looking to leave (at least with one eye) creates strange and self-fulfilling dynamics. Workers care less about building organizations, such as unions, that might benefit them in the long term, because there is no long term. Compensation packages are often structured to maximize compensation in the first four years of a worker’s employment. And that encourages workers to stick around only for four years, before their “cliff” arrives.

If a worker believes they will leave in a few years, then they should also try to make themselves as employable for a *future* employer in those few years. Workers should continue to interview, and to [practice](https://interviewing.io/) and [train](https://leetcode.com/) for interviews, to keep themselves sharp. This might take away from the work itself, but that’s the nature of the [principal-agent problem](https://en.wikipedia.org/wiki/Principal%E2%80%93agent_problem). Workers might also have reasons to reject the tools that management provides. If using an in-house tool instead of an open-source one makes you 10% more effective at your current job, but 10% less effective at other jobs, you might not want to adopt it. 

I think about this topic in connection with LLM tools. One fear I’ve heard repeatedly from my friends in tech is that using AI tools might breed dependence. Even if these tools are effective, we might grow reliant upon them, to the extent that we can’t do our jobs without them. If using an AI coding assistant makes you 30% faster at coding, but 30% worse (or more!) at answering coding questions in an interview, is that a deal worth taking? (Of course, it’s not quite as simple as this. You can train specifically for tech interviews to build your skills back up. But investing that kind of time isn’t always easy, particularly as you get older and have more obligations.)

Reading descriptions of how people use LLM assistants nowadays is fascinating. Engineers seem more like conductors, orchestrating the joint action of many members of an orchestra. For example, read this passage from "[How I'm using coding agents in September, 2025](https://blog.fsck.com/2025/10/05/how-im-using-coding-agents-in-september-2025/)”

> Next up, I open a *new tab* or window in the same working directory and fire up another copy of claude. I tell it something like `Please read docs/plans/this-task-plan.md and <whatever we named the design doc>. Let me know if you have questions.`
> 
> 
> It will usually say that the plan is very well crafted. Sometimes it'll point out mistakes or inconsistencies. Putting on my PM hat, I'll then turn around and ask the "architect" session to clarify or update the planning doc.
> 
> Once we've sorted out issues with the plan, I'll tell the "implementer" Claude to `Please execute the first 3-4 tasks. If you have questions, please stop and ask me. DO NOT DEVIATE FROM THE PLAN.`
> 
> The implementer will chug along.
> 
> When it's done, I'll flip back to the "architect" session and tell it `The implementer says it's done tasks 1-3. Please check the work carefully.`
> 
> I'll play PM again, copying and pasting reviews and Q&A between the two sessions. Once the architect signs off, I'll tell the implementer to update the planning doc with its current state.
> 

To my mind, that doesn’t sound like coding at all! (Or at least the coding I’m used to.) Even if this is what coding is like in the future, most tech interviews are still stuck in the past. Few companies allow interviewees to use an LLM agent, let alone multiple ones ([Meta appears to be](https://www.404media.co/meta-is-going-to-let-job-candidates-use-ai-during-coding-tests/) one recent exception.). Anthropic [doesn’t even want](https://www.entrepreneur.com/business-news/ai-startup-anthropic-to-job-seekers-no-ai-on-applications/486645) you to use its own tools when writing your resume or cover letter to apply to one of its jobs!

Should I, as a middle-aged tech worker, commit fully to this new paradigm of tech work? My employer — and, from what I’ve seen, most employers in big tech — say yes. *404 Media* [reported that](https://www.404media.co/meta-tells-workers-building-metaverse-to-use-ai-to-go-5x-faster/) “A Meta executive in charge of building the company’s metaverse products told employees that they should be using AI to “go 5X faster”.” Jack Dorsey, CEO at Block, feels similarly. Dorsey recently sent an email to all Block employees, saying that those who don’t adopt these tools aren’t “living up to their potential”. He quoted from a [Simon Willison blog post](https://simonwillison.net/2025/Oct/7/vibe-engineering/) titled “Vibe Engineering”:

> If you’re going to really exploit the capabilities of these new tools, you need to be operating *at the top of your game*. You’re not just responsible for writing the code—you’re researching 
approaches, deciding on high-level architecture, writing specifications, defining success criteria, [designing agentic loops](https://simonwillison.net/2025/Sep/30/designing-agentic-loops/), planning QA, managing a growing army of weird digital interns who will absolutely cheat if you give them a chance, and spending *so much time on code review*.
> 

I’m not going to lie — that sounds slightly daunting, and I often think it would be easier if I stuck to my usual way of doing things (which has worked, more or less, for the last 10 years or so).

Whether the cost justifies the benefit depends on the time horizon over which I look. Optimizing for my current employer will lead me to different decisions than optimizing for my future employers. And, of course, it is difficult to predict what tech will even look like in 5 or 10 years. One thing I feel confident saying is that the mathematics are not straightforward. It is mistake to say that if these tools, after some steep learning curve, make you 5%, or 50% (or 500%??) more productive, then you should adopt them. From the employer’s perspective, the math is clear. From the individual’s perspective, not so much. One big issue, besides *future* employability, is that productivity gains in America [benefit workers much less](https://www.epi.org/productivity-pay-gap/) than they “should”. If you’re 50% more productive at your job, your employer is rewarded with 50% more work. But your salary probably won’t increase by 50%. (Another thing that inarguably matters is what your *coworkers* do. There is a collective action problem of sorts. If everyone except for you becomes much more productive, that’s a problem for you. No one wants to be the slowest fish. But if adoption of AI tools lags, that buys everyone more time.)

## Summary

Employment in tech and data, just like all employment, suffers from principal-agent problems. The mathematics changes, depending on your perspective; the self-interest of individuals in an organization does not always align with the interest of the organization itself. In this essay, I’ve explored how those conflicts manifest for data organizations and tech workers.

I don’t always side with the workers, but I understand why they pursue self-interest. After all, everyone else does it. An employer doesn’t care about you, despite how much they might talk about “family” and “community”. So why should we care about them?

If a data team wants its members to pursue goals that are beneficial for the team as a whole, it needs to better align the incentives. When degrading or deprecating functionality, you should have to account the cost savings *and* for the lost benefits. When you engage in promo work, credit should be apportioned according to all “touches”, not just the last one. When equilibria are unstable — when everyone feels the incentive to abandon grunt work — organizations should change the landscape of rewards.

And the same goes for tooling and “AI”. When you receive a communiqué from a leader on high, about how essential it is to invest your time learning how to use AI tools, you know that it’s in their best interest. But it’s not necessarily in yours. As long as most tech interviews and tech hiring remain “conventional”, so should tech workers. Organizations that truly believe in AI should follow Meta’s lead and allow for agents everywhere, including in hiring. (And they should also understand that new hires won’t necessarily have the same skills they used to.) And they should provide workers that are 50% more productive with 50% more compensation.