---
layout: post
title:  "The dual mandate of A/B testing"
date:   2024-09-14 23:07:59 -0500
categories: jekyll update
---

## Introduction

A/B testing has (at least) two goals. First, we’d like to pick and ship a winning "variant": one of the alternatives that we're testing against "control", the status quo. Second, we’d like to understand the performance of these variants, where "performance" is measured using one or more metrics. These goals are often at cross-purposes, a situation that echoes the [most famous “dual mandate”](https://www.npr.org/2022/10/09/1127650735/federal-reserve-dual-mandate-inflation-jobs-employment), that of a central bank.

Let’s take, for example, a test that revamps the onboarding flow for a new user. One purpose of the A/B test is to make a decision: should we ship the new flow, or continue to use the existing one? But we also likely want to say by *how much* the winner won. Whether the winning variant is 20% better or 2% better is immaterial to making a decision (provided both numbers are statistically significant). But the size of the effect can be important for other reasons.

First, the question of “how much” can inform future decision-making and prioritization of competing efforts. If we merely eked out a small gain, that might indicate further efforts in this area will be similarly marginal. Conversely, a large gain could suggest more investment is warranted. And a very large gain would imply that [something unexpected happened](https://www.exp-platform.com/Documents/TwymansLaw.pdf), spurring further investigation.

(Relatedly, product managers often want to say that some experiment generated X dollars for the company — and, as a consequence, they deserve more resources for more experiments. This clearly requires a different standard of evidence than what’s needed for mere decision-making.)

Second, assembling estimates of effect sizes for various metrics is part of the process of telling a *causal story* about the A/B test. If a metric went up, we want to understand why. That typically requires looking at the magnitude of movement of related metrics (and, in that process, we might uncover inconsistencies or bugs that change our understanding of the experiment). For example, if signups are down by 20% in the revamped onboarding flow, we might want to look at which step, or steps, in the funnel are primarily responsible for the dropoff. If we can identify these offenders, they might suggest avenues to improve future experiments.

[Ron Kohavi talks about](https://drive.google.com/file/d/1CgyJVd4SZJT1i5TzlnBGmzs94Rbu6uTQ/view) how the “success rate” of *experiments* differs from that of *ideas*. Ideas often require several iterations to manifest as a statistically significant improvement in an experiment. The insights uncovered in post-hoc analysis after the failed iterations can be the key to driving these improvements; otherwise, we’re condemned to brute-force search.

To sum up, A/B tests drive decisions and generate insights. This is the “dual mandate” of A/B testing. Typically, though, the amount of evidence needed to make a decision is much less than that needed to generate an insight. A decision is a binary thing: ship, or don’t. An insight, on the other hand, requires a reasonably precise estimate of one or more metrics. Often when a company tries to increase “experiment velocity”, they measure this velocity in terms of the number of experiments run or winners shipped. These kinds of metrics give short shrift to the second mandate of A/B testing: actually learning what happened, and why.

In what follows, I will walk through several scenarios in which we can use “tricks” to speed up decision-making in A/B testing at the expense of learning a more precise estimate. These are cases in which we favor the first mandate of A/B testing over the second. I don’t mean to imply that such tricks are necessarily bad; rather, I want to point out that there is "no free lunch", and by making decisions faster we often lose something in exchange.

## Multi-armed bandits

[Multi-armed bandits](https://en.wikipedia.org/wiki/Multi-armed_bandit) are sometimes conceived of as a substitute for A/B testing. Chris Stucchio, for example, wrote a [provocatively titled post](https://www.chrisstucchio.com/blog/2012/bandit_algorithms_vs_ab.html) called "Why Multi-armed Bandit algorithms are superior to A/B testing”.

(For those who don’t know: bandits are a procedure in which variants ("arms") get more or less exposures directed towards them based on how they are performing to date. They are typically used when the metric follows shortly after exposure: e.g., a user seeing an ad, and then clicking on it. Bandits steer traffic towards winners and away from losers; this contrasts with conventional A/B testing, in which exposures are allocated in a static way (usually an even split across all variants). Bandits can be shown to “minimize regret”: the cost associated with exposing users to an experiment variant known to be significantly worse.)

Later, Stucchio amended his statement, writing a [follow-up](https://www.chrisstucchio.com/blog/2015/dont_use_bandits.html) (also provocatively titled) called “Don't use Bandit Algorithms - they probably won't work for you”. In it, he noted that bandits make a number of strong assumptions that may not be satisfied in many practical applications (the previously noted "timescale" assumption, the assumption that effect sizes are constant over time, etc.). He argued that if you don’t understand the math well enough to verify the assumptions of bandits, you probably shouldn’t be using them.

But an additional problem with bandits is that they are designed largely for decision-making, not for learning. The goal of a bandit is not to produce the narrowest possible confidence interval for the effect size. It is instead to drive exposures towards the variant it believes to be better. If one variant is much better, then, as time marches on, it will end up getting most or all of the new experiment traffic. In a bandit, new traffic is used for exploitation: to maximize reward and minimize regret. In a conventional A/B test, new traffic is used for estimating the effect more precisely. Even if a bandit produces an estimate of the effect size (and in many cases, this isn’t even the point), it will be less certain than that from a conventional A/B test.

## Sequential testing/early stopping

An idea that is loosely related to multi-armed bandits is “sequential testing” or “early stopping”. (See [here](https://engineering.atspotify.com/2023/03/choosing-sequential-testing-framework-comparisons-and-discussions/) for an excellent overview.) Both procedures try to minimize the amount of time spent "exploring", and maximize that spent "exploiting". In sequential testing, we look at experiment results early, but in a statistically rigorous way. For example, instead of looking at the results just after the desired runtime is reached (say, 4 weeks), we look at the results 4 times, once per week. If we find statistically significant results after *any* look, we can stop the test early and ship the winner.

This probably sounds like “peeking”: the statistically *illegitimate* procedure in which you — well, not you, but someone who doesn’t know any better 😉 — look at experiment results before you’re “supposed to”. Peeking is bad because it inflates the likelihood of making a false positive error, usually to levels well above "alpha", the threshold for a single test (often 0.05). But the reason this happens is that, when a decision is made by peeking, the statistical test uses the same value of “alpha” as for a normal statistical test. In the legitimate version of peeking — sequential testing — we penalize alpha based on the number of peeks and their timing.

There are a variety of methods for implementing sequential testing, and more subtleties than I care to go into, but one underappreciated point is that the estimates of effect sizes from sequential testing are inflated. As Daniel Lakens [writes](https://lakens.github.io/statistical_inferences/10-sequential.html#reporting-the-results-of-a-sequential-analysis),

> A challenge when interpreting the observed effect size in sequential designs is that whenever a study is stopped early when is rejected, there is a risk that the data analysis was stopped because, due to random variation, a large effect size was observed at the time of the interim analysis. This means that the observed effect size at these interim analyses over-estimates the true effect size.
> 
> …
> 
> A similar issue is at play when reporting *p* values and confidence intervals. When a sequential design is used, the distribution of a *p* value that does not account for the sequential nature of the design is no longer uniform when H0 is true
>

When sequential testing is useful — when it allows you to stop early and make a quick decision — is exactly when it is most imprecise. In these cases, estimates of the effect size are too large in magnitude, and confidence intervals are wider than for later stopping. This isn’t too surprising, actually. When we stop early, we have much less sample size than when we stop late, so our ability to precisely identify the effect (the “power”) is also diminished. But, sadly, this means that the dual mandate is, once again, dueling: we can run experiments faster, but we also learn somewhat less from each. (Of course, this tradeoff can be justified in many cases. I only want to point out that it exists.)

Note: there are ways to *correct* effect size estimates and p-values for early stopping, and packages such as [rpact](https://www.rpact.org/) implement these techniques. Regardless, “conventional” A/B testing can’t be beaten if you want as narrow a confidence interval as possible, for a given sample size.

## Underpowered tests

Underpowered tests deservedly have a bad reputation. Kohavi [writes that](https://drive.google.com/file/d/1oK2HpKKXeQLX6gQeQpfEaCGZtNr2kR76/view) “Experiments with Low Statistical Power are NOT Trustworthy” (his emphasis, not mine). Of course, when an experiment has low power, our probability of detecting an effect, if it does exist, is much lower than for a properly powered experiment. (This is the definition of statistical power.) That being said, many tests, both in the tech sector and otherwise, are underpowered, and it is interesting to see how the “dual mandate” applies to these tests.

The framework of “Type S” and “Type M” errors is useful here (see [Gelman and Carlin, 2014](http://www.stat.columbia.edu/~gelman/research/published/retropower_final.pdf)). Type S errors are those in which the *sign* of the effect size estimate is wrong; Type M errors are those in which the *magnitude* of the effect is exaggerated. In industry, we usually set statistical power at 80%: we want an 80% chance of detecting an effect of a particular size. But, according to this framework, even experiments with power much less than 80% can have reasonable type S and type M errors, particularly the former. For example, if power = 20%, the probability of a sign error is still only 0.005, and the “exaggeration ratio” is a relatively modest 2.3.

The authors write,

> But when power gets much below 0.5, the exaggeration ratio becomes high (that is, statistically significant estimates tend to be much larger in magnitude than true effect sizes). And when power goes below 0.1, the Type S error rate becomes high (that is, statistically significant estimates are likely to be the wrong sign)
> 

This raises the question: what is the problem with operating in the “underpowered, but not grossly so” regime? We might define this as between 10% and 50% power, based on the comments above. 

The problem is not so much that we’d make incorrect decisions: if we identified a winner, it would be highly unlikely that we’re incorrect (this would require making a sign error, which is very unlikely in this regime). It is instead that we wouldn’t know, with any reasonable precision, by how much the winner won. (The other big problem is that, with underpowered tests, we would miss out on many winners, since we’d incur type II errors.)

Kohavi writes,

> When the power is low, the probability of detecting a true effect is small, but another consequence of low power, which is often unrecognized, is that a statistically significant finding with low power is likely to highly exaggerate the size of the effect. The winner’s curse says that the “lucky” experimenter who finds an effect in a low power setting, or through repeated tests, is cursed by finding an inflated effect
> 

In other words, learning a precise estimate from an experiment requires a much higher standard of evidence than making a decision. Decision-making can tolerate type M errors, but not type S errors, and there is a regime in which the former is likely but the latter is not. However, learning precise effects requires that neither type of error occurs.

## Proxy metrics

Decisions in A/B testing are often made on the basis of proxy metrics. As Netflix [explains](https://netflixtechblog.com/improve-your-next-experiment-by-learning-better-proxy-metrics-from-past-experiments-64c786c2a3ac),

> While every team ultimately cares about the same north star metrics (e.g., long-term revenue), it is highly impractical for most teams to measure these in short-term A/B tests. Therefore, each has also developed proxies that are more sensitive and directly relevant to their work (e.g., user engagement or latency).
> 

In other words, instead of trying to move a north-star metric like “long-term revenue” in an A/B test, you instead try to move a more sensitive metric, like “user engagement”, under the assumption that increased user engagement will also increase long-term revenue (in a way that is too weak to be measured in a short A/B test, but still matters to the company long-term). This assumption can be verified in a separate experiment or investigation, but is often just, well, assumed.

We need proxies to make experiments tractable. (Moving north star metrics is very difficult at mature companies.) But it’s worth noting that while proxy metrics can be estimated precisely in short-term A/B tests, provided they are adequately powered, the corresponding north-star metrics cannot. In other words, we can say that the winning variant in a short A/B test increased user engagement by [0.6%, 1.4%], but it is much more difficult, or even impossible, to come up with a reasonably narrow confidence interval for long-term revenue. We can state a decision-related claim (”we shipped a winner that improved long-term revenue”) without being able to state the corresponding learning-related claim (”the winner improved long-term revenue by $1 million”).

Once again, learning “impact” requires a higher standard of evidence: one that measures the intervals of north star metrics themselves, not just those of proxies.

## Interaction effects

The usual advice for dealing with interaction effects in experiments is “don’t worry about it”. That’s not even my being glib: Microsoft has a blog post titled, “[A/B Interactions: A Call to Relax](https://www.microsoft.com/en-us/research/articles/a-b-interactions-a-call-to-relax/)”. The authors argue that “the vast majority of A/B tests either don’t interact or have only relatively weak interactions”, and it is rare that the interaction matters enough to flip the sign of the effect size. But there is an interesting caveat:

> It’s possible that there were other A/B interactions that we just didn’t have the statistical power to detect. If the cross-A/B test treatment effects were, for example, 10% and 11% for two different cross-A/B test assignments, we might not have detected that difference, either because the chi-square test returned a high p-value, or because it returned a low p-value that got “lost” in the sea of other low p-values that occurred by chance when doing hundreds of thousands of statistical tests.
> 
> 
> This is possible, but it raises the question of when we should worry about interaction effects. For most A/B tests at Microsoft, the purpose of the A/B test is to produce a binary decision: whether to ship a feature or not. There are some cases where we’re interested in knowing 
> if a treatment effect is 10% or 11%, but those cases are the minority. Usually, we just want to know if key metrics are improving, degrading, or remaining flat. **From that perspective, the scenario with small cross-A/B test treatment effects is interesting in an academic 
> sense, but not typically a problem for decision-making.**
> 

We see, yet again, the tension between decision-making and generating “accurate” insights (somewhat derisively referred to by Microsoft as a matter of “academic” interest). For making decisions, interactions between concurrent A/B tests rarely matter, since the sign of the estimate hardly ever flips after considering them. But for learning precise effects, interactions can matter a great deal. If you want to report on *how much* your A/B test winner increased user engagement, or *how much* money it will generate, I’d recommend relaxing a little less.

## Discussion and conclusion

The tone of this essay probably sounds a bit defeatist. I’ve basically argued that many of the various tricks we use to speed up A/B testing — sequential testing,  bandits, proxies, and ignoring interactions — run aground on the shoals of “learning something precisely”. Here I will try to be somewhat optimistic, and argue that the situation is not quite as dire as it might appear.

First, for many metrics and experiments, it is true that the only goal is to make a decision. For some guardrail metrics, like app performance, we don’t care by how much we’re moving the metric, only whether we're affecting it negatively. Second, not all tests are money-makers. Sometimes we simply want to replace a legacy implementation (a backend service, an old design) with something newer, and our sole purpose is to assess whether we did harm or not (a “non-inferiority test”). We can likely use sequential testing to speed things up in this case. And, third, if we simply want to “hill-climb”, and already have a good idea of how to prioritize product development, the question of “how much?” might not matter to us.

That being said, learning precise estimates is still useful, and, of the two mandates, it is the one more often neglected. I firmly believe we should try not just to ship winners, but also learn what works, and why. We should also strive to measure our impact. Often this is most important for “political reasons”: to justify further resources devoted to experimentation. Making a statement like “we generated between $5M and $10M in revenue” is much more effective than saying “we had 15 successful experiments”, even if the latter might be an OKR and the former might not be. Moreover, prioritizing insights can increase experiment velocity in the long run, even if it mucks things up in the short run.

All of this usually requires a different track of experimentation: one explicitly devoted to learning, and one that can run over timescales that would be unreasonable for most experiments. These are experiments led by data scientists, not product managers, and such experiments idle in the background while the rest of the company flits busily from one thing to the next. Doing “[long-term](https://drive.google.com/file/d/1ycABCZpD_oWCLwWKL_BixOeMLgB6iOK0/view)/[cumulative impact](https://www.geteppo.com/blog/holdouts-measuring-experiment-impact-accurately)” experimentation can be immensely valuable, but it requires strong buy-in from leadership, since it can be difficult for engineers to implement and tends to create tech debt. Even at the large companies I’ve worked, where resource constraints are less of a problem, long-term experimentation is often eschewed in favor of the dopamine-boosting, experimentation-as-usual mindset.

To close: running a lot of experiments is easy. Even picking winners is not too difficult. But learning — what worked, what didn’t, by how much, and why — remains a big challenge in experimentation, and there aren’t the same set of “tricks” to speed up the process. We need time, sample size, and money: preferably lots of each.
