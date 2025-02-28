---
layout: post
title:  "A/B testing and time"
date:   2025-02-28 00:07:59 -0500
categories: jekyll update
---

## Tl;dr

- I discuss how many aspects of A/B testing -- sample size calculation, treatment effects, etc. are implicitly dependent on time
- I make a case for re-running experiments, particularly after large changes to the product or userbase

## Introduction

Most of us know the key assumptions underpinning A/B testing: things like “SUTVA” (stable unit treatment value assumption), [finite variance](https://mverbakel.github.io/2021-04-03/metric-validation) of metrics, randomization, [“adequate” sample size](https://varunprajan.github.io/blog/how-to-find-the-sample-size-for-an-ab-test/), and so on.

In this post, I want to talk about one lesser-known, or at least less well-appreciated, assumption. This has to do with “time”. I’ll admit this sounds ponderous, but fear not: unlike most of my posts, there will be no math or code. The goal is to simply get you thinking about this topic, and how it applies to tests you’re responsible for — ones in which this assumption seems automatic, even though it isn’t.

## Sample sizing

Picture this common situation. You run a sample size calculation for an A/B test. The calculation is based on historical data. To be specific, you use historical data to estimate two things: the mean and variance of the metric, and the expected number of exposures over time. You then plug these into a sample size calculator, calculate the runtime, and launch the experiment.

After a week or two of data collection, you find that you erred. The sample size and runtime are incorrect. In the happy case, your test can be stopped earlier than you expected. In the unhappy case, which, in my experience, is more likely, it means your test will take longer, possibly imperiling the overall experimentation timeline that you and your PM painstakingly constructed.

What happened? Barring a calculation mistake, the most likely source of error is the assumption that historical data can be straightforwardly applied to the future: **that time doesn’t matter**. For example, consider exposures.

Historical exposures (say, users) are not guaranteed to equal future exposures. For a new feature, or a feature that was recently launched to a new country or market segment, we may not have historical data. Even if we have historical data, the time period that the experiment is run in, and the time period that the historical data was pulled from, may differ in important ways. There may be holiday effects or other seasonal effects. The userbase might have grown rapidly (or the opposite). We might have had an important geopolitical event, or an unforeseen catastrophe, or a pandemic. Or, more prosaically, we might have just revamped our product, or our pricing, and are waiting for our business metrics to settle. Or we might have altered our total marketing spend, or its mix across channels. Or Facebook or Google might have tweaked their algorithm, yet again, and our acquisition funnel will suffer until we adjust in turn.

What makes this situation challenging is that *these things happen all the time*. Unlike pandemics, which, I hope, are once in a lifetime events, the more quotidian examples are simply normal, day-ending-in-y sort of stuff at tech companies. (The only constant is change, or something.) In other words, if you have a time series for which ARIMA or [prophet](https://facebook.github.io/prophet/) works reasonably well, it’s likely the product you work on hasn’t experienced much innovation.

The same applies, only more so, to the other source of historical data: the mean and variance of the metric. These are, once again, highly sensitive to seasonality, holidays, feature rollouts, changes to *other* parts of the product (e.g., through cannibalization), pricing, user acquisition, macroeconomic conditions, acts of god, and so on. Of course, I don’t mean to imply that sample size estimation and power calculations are useless. We can often get reasonably close to the right answer. But there are situations, particularly after a highly disruptive change, where accurate sample size estimates are simply impossible to develop.

## Treatment effects and time

There’s actually a deeper issue here, one that extends well beyond getting the sample size and runtime wrong. Critical aspects of an A/B test are implicitly related to time. Obviously, the audience that gets exposed to an experiment depends on when the experiment is run. And, as we discussed, the behavior of that audience during the experiment (the mean and variance of the metric(s)) also depends on time. As such, the estimated “treatment effect” — how much the treatment helps, or hurts, the primary metric — can also be highly sensitive to the time period over which the test is run, and everything else going on at the company around this time. **The treatment effect is not just a function of the treatment**. It’s also a function of, well, everything else: the userbase and the environment and so on.

Let’s take, as an example, a product change that increases the frequency of ads we show to users (say, at a company like YouTube). We’re worried that showing users more ads might hurt our metrics, so we’ll run this as an A/B test, on some portion of users who use the service. Consider these “time points”, through our company’s history, during which we could have run this test:

1. After the product was first launched (to a limited set of countries)
2. After the product reached worldwide adoption
3. After the introduction of a premium, ads-free tier in a limited set of countries
4. After a pricing change for the ads-free tier in a limited set of countries
5. After the introduction of a content recommendation engine
6. After the introduction of an ads recommendation engine
7. After the pandemic
8. After the acquisition of a rival and integration of their content

In all cases, we’re experimenting on “all users”. But who those users are, and how they behave, varies greatly. We should expect to see **substantially different treatment effects** in many of these experiments (at least in magnitude, although perhaps not sign).

The problem isn’t just “heterogeneous treatment effects” (HTEs). In other words, you might say: particular countries might differ in their pricing structure, userbase, presence of ads-free tier, etc. So, we can decompose “all users” into “users in various countries”, estimate the treatment effects country-by-country, and compare these across time points to better understand the phenomena at play. The overall treatment effect is just a weighted average of the treatment effects in individual countries.

It’s not that simple, though. First, it’s not obvious how to extend this logic to some of the other scenarios discussed above, like “after the pandemic” or “after a pricing change” or “after an acquisition”. Second, even if we can run an HTE analysis, some of the important covariates might also be unmeasurable. We can’t really measure “willingness to purchase” or “willingness to endure ads”, and we only have crude proxies (like country) that are imperfectly correlated with what we’d like to know.

Here’s a particular devilish (and instructive!) example, from my time at Spotify. My team worked on a particular set of algorithmic playlists. Users would complain about the “echo chamber” problem: that, on average, their playlists would recommend too many songs they had listened to and saved, and too few new discoveries. Users felt trapped in an echo chamber of their favorite music.

We ran a series of experiments to reduce the “familiarity index” of these playlists, but the results were always negative (drops in consumption time, active users, etc.). One explanation, of course, is that users were deceiving us (or themselves). Another is that the users who were complaining were unrepresentative of the overall population. There were probably also time effects, like the fact that echo chambers become annoying over a timescale longer than we were willing to run our experiments for.

No doubt these explanations are partly true. But another problem is that the “overall population” is constantly changing, and therefore the treatment effect of “making playlists less familiar” probably is as well. The treatment effect depends on the userbase and its behavior, but the userbase and its behavior **themselves depend on time**.

If you make a product that is, intentionally or unintentionally, geared towards a particular audience, like users who enjoy familiar music, you will cause other types of users to consume less, churn, or not adopt in the first place. (Particularly when there’s so much choice in content to consume, and so little time to consume it.). My team’s algorithmic playlists had likely self-selected into a population dominated by familiarity-heavy users. Unsurprisingly, the treatment effect we found, *on that population*, was negative. But if we could have, somehow, forced all music listeners on Spotify to listen to our playlists, we might have found rather different results.

You experiment on the users you have, not the one you would like to have. **But those experiments themselves change the users you have**, often in ways that are unmeasurable.

When we talk about “[hill-climbing](https://chris-said.io/2016/02/28/four-pitfalls-of-hill-climbing/)”, or “optimization”, we should keep these facts in mind. We are optimizing on a particular population, and that population itself (and the way it behaves) is constantly changing, in some cases because of the changes we make to the product, but in other cases because of exogenous factors. We should not expect the same experiment with the same treatment, repeated at different points in time, to return the same results — whether that is the treatment effect, the mean and variance of the metric for the control cell, the number of exposures, or anything else. This is not just because of “seasonality” or holiday effects. It is because the population being experimented on, and their behaviors, are constantly changing, both in ways we’ve influenced and in ways we haven’t.

## The importance of re-running experiments

I remember being interviewed a few years ago and asked, “what would you do in your first week on the job?” I responded with something like “read through all the past experiments you’ve run, and past analyses you’ve conducted.” I believe in learning from the successes of the past, and not repeating its failures.

Now, I’m not so sure. Some forgetfulness might not be a bad thing. The constant churn of organizational structure and personnel in tech might have the happy consequence of making us forget what we tried in the past, and therefore be willing to try it again. If the experiment’s audience and/or its behavior have changed in substantial ways since our last attempt, we might reverse our previous failures.

I’m being slightly facetious. But only slightly! There are strong reasons to re-run experiments again and again, especially after the product or the userbase has undergone massive changes. And re-running experiments can make sense whether the initial experiment was successful or not!

In the former case, we estimate a new treatment effect, and can do further analysis (like HTEs) to try to decompose that effect into its drivers. We might learn something about the importance of various changes (product, userbase, macroeconomic) that transpired between our last attempt and the current one. In some cases, we might even learn that what we had previously shipped is actually neutral, or even negative! Either our previous result was a false positive, or the test we’re running now is fundamentally different than the previous one, for the reasons described above.

In the latter case, of a previously failed experiment, we might be newly successful. Typically, though, re-running a previously failed experiment requires a strong rationale. This could be informed by the HTE-type analysis discussed above. (For example: “we expect users will respond differently to these in-product notifications, given the pricing and bundling changes we launched 2 months ago have changed the userbase in X and Y ways”)

## Concluding remarks

What I’ve discussed might strike you as unsatisfactory. Is the knowledge gained from A/B testing largely unstable and contingent? Do we need to run, and re-run, tests constantly? Can’t we learn something once and be done with it?

There is obviously a compromise to be made between trying out new ideas and re-assessing old ones. But, at least at the companies I’ve worked at, that balance is tipped too strongly towards the former and away from the latter.

Of course, re-running past experiments and analyzing them takes time, particularly if advanced techniques, like HTEs, are not part of the experimentation platform. Data scientists’ time is precious, and we already have too much to do. But the risk of not re-running experiments is also great: the danger is that you’ve climbed a hill that no longer exists.
