---
layout: post
title:  "How to evaluate a quantitative claim"
date:   2024-10-14 05:07:59 -0500
categories: jekyll update
---

## Introduction

The world abounds with quantitative claims. Here are three headlines + sub-headlines, each from *The New York Times,* and all from the last 3 months.

- "[Older Adults](https://www.nytimes.com/2024/08/12/health/alcohol-cancer-heart-disease.html) Do Not Benefit From Moderate Drinking, Large Study Finds. Virtually any amount increased the risk for cancer, and there were no heart benefits, the researchers reported.”
- "[Black Voters](https://www.nytimes.com/2024/10/12/us/politics/poll-black-voters-harris-trump.html) Drift From Democrats, Imperiling Harris’s Bid, Poll Shows. Vice President Kamala Harris is on track to win a majority of Black voters, and has brought many back to her party since taking over for President Biden. Still, a significant gap in support persists.”
- “[Executives and Research](https://www.nytimes.com/2024/10/12/business/dealbook/executives-and-research-disagree-about-hybrid-work.html) Disagree About Hybrid Work. Why? Companies like Amazon have required a return to the office five days a week despite findings showing benefits to employers that allow some remote days.”

I call all of these “quantitative claims” despite the fact that, upon first glance, they might not appear “quantitative” at all. Neither the headlines nor the sub-headlines include any numbers, percentages, or p-values.

## Reformulating the claim as quantitative

Let’s try to improve upon the headline writers at the Times, and rewrite the central claim of each of these stories in a quantitative way:

- Older adults who engage in moderate drinking are X% more likely than older adults who drink only occasionally to develop cancer over the next Y years. Presumably, therefore, their lifespan is shorter by Z years.
- Black voters are X% less likely to vote for Kamala Harris in 2024 than for Joe Biden in 2020
- Hybrid (or remote) workers are X% happier than in-office workers, Y% more productive, Z% less likely to quit, etc.

In some cases, “X”, “Y”, and “Z” are easy to fill in. The article about black voters claims that black support for Democrats has dipped from 90% in 2020 to 78% in 2024. So, X = 12 pp (i.e., in absolute terms).

In other cases, filling in the blanks is more taxing. For the hybrid work article, one researcher explained, “The academic studies that have been done, and there are not that many, show a range of outcomes — and they generally show a kind of neutral to slightly positive.” Another, Adam Grant, pointed to a [“meta-analysis”](https://onlinelibrary.wiley.com/doi/abs/10.1111/peps.12641?casa_token=4EUYwv7UhWIAAAAA%3A4riTpGzPUjsFVYpDMWhTD7xnqYHL1z1FroYZ0jwhmIogWyZhhVU7tEliJa9rY-VN8Uos8xPCXBeLdqo) of hybrid work studies. Even the abstract of that article is missing explicit statements of the effect sizes, simply stating that “[remote work intensity] had overall small but beneficial effects on multiple consequential employee outcomes including job satisfaction, organizational commitment, perceived organizational support, supervisor-rated performance, and turnover intentions”. (I wasn’t willing to fork over the $20 required to read the full text.).

And for the article about the effects of drinking on health, the journalist similarly eschews quantitative claims. I dug into [the study](https://jamanetwork.com/journals/jamanetworkopen/fullarticle/2822215) upon which the article is based, and the abstract states:

> In the total analytical sample, compared with occasional drinking, high-risk drinking was associated with higher all-cause (hazard ratio [HR], 1.33; 95% CI, 1.24-1.42), cancer (HR, 1.39; 95% CI, 1.26-1.53), and cardiovascular (HR, 1.21; 95% CI, 1.04-1.41) mortality; moderate-risk drinking was associated with higher all-cause (HR, 1.10; 95% CI, 1.03-1.18) and cancer (HR, 1.15; 95% CI, 1.05-1.27) mortality, and low-risk drinking was associated with higher cancer mortality (HR, 1.11; 95% CI, 1.01-1.22).”
>

(Don’t worry if you found that sentence impenetrable. We’ll return to it later.)

Even the process of reformulating claims as quantitative can be revealing.

For example, explicitly quantitative claims typically involve a **comparison** between two groups. Black voters in 2024 vs black voters in 2020; hybrid workers vs in-office workers; moderate drinkers vs occasional drinkers, etc. Often the headline doesn’t mention the comparison group. When I initially read the headline about alcohol use, I thought the comparison was to non-drinkers, not to occasional drinkers. That fact was stated only in the third paragraph. So, in fact, the health benefits of teetotalling might be even greater than this article implies.

Explicitly quantitative claims should also distinguish between actuals and counterfactuals, and between descriptive claims and causal ones. The claim about black voters is a **descriptive** one. It states that the Democratic share of the black vote has dropped between 2020 and 2024. It does not imply, for example, that the preferences of 2020 black voters have changed in the intervening 4 years. Some of these voters are now dead; others might not vote this cycle out of apathy; and previous non-voters might become voters, either because they are now newly eligible (via citizenship or because they turned 18), or because they have been persuaded that this election is particularly significant. Decomposing the effect size referenced in the quantitative claim into these smaller, individual effects is quite challenging, and likely requires extra data and analysis. It is possible, for example, that much of the effect is being driven by first-time voters (and, in fact, 17% of the NYT poll sample is of voters who didn’t vote in 2020).

Because this claim is purely descriptive, we should not ascribe to it **causal** import. It is not necessarily true, for example, that changes in the black vote between 2020 and 2024 are *caused* by inflation, immigration, the Democratic candidate, or any number of other factors that have changed between 2020 and 2024. The causal graph of vote preference is very tangled, and the problem is “overdetermined”: there are too many causes, and too few data points to tease them apart. (That’s what makes discussing politics both interesting and frustrating: it is easy to tell a causal story, and very difficult to disprove it.)

But other claims, like the ones about hybrid workers vs in-office workers, are likely intended to be **counterfactual** and **causal**. We don’t simply want to assert that hybrid workers are more productive than in-office workers. We instead want to claim that “similar” hybrid workers are more productive than their in-office counterparts, or that changing the working arrangement for some set of workers will change their productivity. The trouble with such counterfactual statements — e.g., taking a hybrid worker, constructing their in-office doppelganger, and predicting that counterfactual worker’s productivity — is that they require stronger tools than mere descriptive analysis. This is the province of “causal inference” or “econometrics”.

To sum up:

1. Trying to reformulate *implicitly* quantitative claims into *explicitly* quantitative ones is a useful exercise. It often reveals subtleties, like the exact groups being compared, whether these comparisons are descriptive or counterfactual, and the type of claim being made (causal or descriptive).
2. Doing this reformulation isn’t trivial, especially for casual readers of the news. It requires digging into the text and, often, the references as well. We should not expect everyone to “do their own research”, and I would hope journalists would communicate more clearly.

Readers of this blog presumably have higher standards than casual NYT readers. We would like to know not just *what* the quantitative claims are, but also whether they are **meaningful**. How might we do that?

## A framework

This is a deep subject, and I am certainly not an expert in it. But I find three sets of questions useful. 

1. “**Practical significance**”: If the claimed effect size is true, what does it mean or imply, in practical terms?
2. “**Statistical uncertainty**”: If the process by which the claim was generated were *repeated*, how much would the effect size change?
3. “**Other uncertainty**”: What are other sources of uncertainty in the effect size that are not accounted for in the previous step?

Let’s try to apply this framework to the quantitative claims mentioned above.

### Practical significance

Practical significance is the “so what?” question. In other words: suppose it’s true that Democrats perform 12 pp worse among Black voters in 2024 than in 2020. Or suppose moderate drinking increases mortality by X%. **So what?** We grant the premise, and then interrogate what follows.

Let’s take, as an example, the claim about the effect of drinking on mortality. [The study](https://jamanetwork.com/journals/jamanetworkopen/fullarticle/2822215) analyzed ~135k participants, with median age of 64. Over the next 12.4 years (on average), 15.8k participants died, or almost 12%. For moderate drinkers, the “hazard ratio” for all-cause mortality was estimated as 1.1. In absolute terms, though, this translates into something like 1-2 pp: switching from moderate drinking to occasional drinking might reduce your risk of dying from 12% to 10%, or, in terms of lifespan, add less than a year to your life. (Once again, it would be nice if journalists used these kinds of numbers in their articles!)

Practical significance depends on your perspective. From an individual perspective, 1-2 pp of mortality risk might not sound like a lot. But, from a social or governmental perspective, that could translate into many more years of life for citizens in aggregate, significant increases to GDP, and perhaps also significant reductions in healthcare expenditures.

In hypothesis testing, statistical significance does not imply practical significance. Practically insignificant effects can be statistically significant if sample sizes are enormous or, roughly equivalently, if large sample sizes are generated for small subpopulations. The poll of black voters shows how this might happen. The NYT found an effect size of -12pp (for the Democratic vote) and +6pp (for the Republican vote). In terms of election margin, this translates into -2.2pp, assuming that, [as in 2020](https://catalist.us/wh-national/), 12% of the electorate is black. This number is meaningful, given how close the election is, but it isn’t necessarily “large” (e.g., compared to the margin of error). In “normal” polls, various subgroups are sampled, or at least weighted, in proportion to their actual population. Large changes in small subgroups don’t influence the topline result, and we are discouraged from squinting too much at the “crosstabs”. But, in the NYT poll, black voters are “oversampled” and viewed in isolation. We lose the sense of how these changes affect the overall election because we are no longer looking at the overall election.

Finally: effect sizes are almost always presented as means or medians. But some effects are highly heterogeneous. Remote work might provide “overall small but beneficial effects on multiple consequential employee outcomes including job satisfaction” for the average worker, but the average worker is a fiction. Benefits of remote work are likely larger for particular kinds of workers (those with children or care obligations, those with excessively long commutes, etc.) and much smaller, or negative, for other workers. For example, if we were interested in predicting how a change in remote work policy might affect employee retention, we should probably examine individual changes in job satisfaction, not average ones.

### Statistical uncertainty

Statistical uncertainty can be challenging to understand. I find the following thought exercise useful: suppose we repeated the procedure used to generate the claim thousands of times. How much would the effect size change? What range would encompass most of the observed values?

The subtleties largely lie in the phrase “the procedure used to generate the claim”. In polling, we imagine that we repeat the polling procedure thousands of times. Because of random sampling, even if the true (”population”) parameter remains the same for each iteration, we will get slightly different results each time. This is known as “**sampling error**”.

In modern polling, matters are even more complicated. The NYT, like most reputable pollsters, reweights their sample to correct for “response bias” and for the likelihood a respondent will vote. This further increases the uncertainty of the effect. As the NYT writes in fine print: “The margin of error accounts for the survey’s design effect, a measure of the loss of statistical power due to survey design and weighting. The design effect [is]…1.89 for the Black likely electorate”. In other words, the true confidence interval is almost twice as wide as a naïve estimate would yield.

The NYT reports the margin of sampling error as “plus or minus 6.3 points among Black voters”, meaning that values of Democratic support (among 2024 black voters) between 72% and 84% are all reasonably plausible, as are values of Republican support between 9% and 21%. In other words, the effect size of -12 pp we previously quoted could be anywhere between -6 pp and -18 pp (with 95% confidence).

For claims derived using procedures more complicated than random sampling, estimating the statistical uncertainty or “standard error” becomes correspondingly more difficult. Typically we start with the “usual” standard error formulas, but inflate or make them “robust” according to some procedure. For cases where a closed form procedure is not available, we can rely on methods like bootstrapping, in which we generate many hundreds of “realizations” from the actual dataset, and repeat the modeling procedure on these realizations to accumulate a distribution of effect sizes. However, even bootstrapping isn’t a panacea, at least [with some causal methods](https://onlinelibrary.wiley.com/doi/10.3982/ECTA6474).

### Other uncertainty

Often, though, the statistical uncertainty isn’t even the largest contributor to the overall uncertainty. Let’s return to the polling example. The NYT writes, again in fine print:

> Real-world error includes sources of error beyond sampling error, such as nonresponse bias, coverage error, late shifts among undecided voters and error in estimating the composition of the electorate.
>

In a study by Shirani-Mehr and collaborators, “[Disentangling Bias and Variance in Election Polls](https://www.tandfonline.com/doi/full/10.1080/01621459.2018.1448823)”, the authors empirically analyzed “4221 polls for 608 state-level presidential, senatorial, and gubernatorial elections between 1998 and 2014”. They found

> Comparing to the actual election outcomes, we find that average survey error as measured by root mean square error is approximately 3.5 percentage points, about twice as large as that implied by most reported margins of error.
>

In other words, we ignore non-statistical uncertainty at our peril. It is likely that the [-18 pp, -6 pp] confidence interval I mentioned earlier is itself too narrow.

(It also seems suspicious to me that the Times compares a polling result, in 2024, to a post-election analysis, in 2020 and 2016. These types of estimates are not directly comparable, which introduces additional errors/uncertainty. All in all, the evidence is compatible with a much weaker version of the claim that the NYT initially presented, although I realize this would not have made for as compelling a headline.)

The situation can be even worse for causal methods. They suffer not just from non-representativeness of the dataset (as in polling), but possibly also invalidity or misspecification of the *model*.

(Even polling isn’t based purely on simple random sampling anymore, as explained above. It relies on models, for non-response bias and voter propensity. So the “data” vs. “model” distinction may not always be a clear one.)

In the alcohol-health study, the authors fit a regression model with many covariates, designed to adjust for selection bias: the possibility that moderate drinkers differ from occasional drinkers in ways other than just their drinking. Their reported confidence intervals (e.g., a hazard ratio of “1.03-1.18”) are generated using the “usual” standard errors. But if the regression model itself is “misspecified” — if it omits an important covariate — the errors will be larger than those stated. The authors of the alcohol-health study note both problems. They write,

> Second, as in any observational study, we cannot entirely rule out residual confounding, despite adjusting for many potential confounders. And third, this study was conducted in older adults in the UK with a high proportion of White participants, so our results may not be generalizable to other racial ethnic groups or populations with different lifestyles, drinking patterns, or socioeconomic development.
>

Both the non-representativeness of the data (”high proportion of White participants”) and possible model misspecification (“residual confounding”) increase the uncertainty of the effect size estimates.

The big problem with the “other uncertainty” category is that it is difficult to estimate by how much the uncertainty increases, and by how much our confidence should deteriorate. We typically struggle to estimate the effects of unmeasurable or unknown confounders.

## Wrap-up

It takes a lot of work to evaluate what we read (or, more accurately, skim) in the news. A headline like “Black Voters Drift from Democrats” hides a great deal of nuance, and, in this blog post, I’ve undertaken the (surprisingly) arduous task of unpacking the subtleties. (Even after nearly 3000 words, there are aspects I’ve glossed over or omitted.) I hope this exercise has conveyed the numerous caveats associated with these claims, and made you think twice before forwarding them to your social media followers, at least without additional context.

But more important than dissecting a _particular_ claim is understanding the _general_ process of evaluating a claim. Once again, there are four key steps:

1. **Rewrite** the claim as a quantitative one, and identify what **effect** is being computed, what **groups** are being compared, what **type** of claim (causal/counterfactual, or descriptive) is being posited, and what kind of **evidence** is being marshalled.
2. Take the claim at face value. What does it imply in **practical terms**? Should we **care** about the effect, even if it is statistically significant?
3. Now revisit the assumption in the previous step. How much **uncertainty** in the estimated effect size is a result of the **statistical process** by which the claim was generated? This is typically related to “standard errors” of some sort. Is the resulting confidence interval narrow enough to take the claim seriously?
4. Finally, think about **other** sources of uncertainty. For model-based estimates, this can involve model misspecification. The data might be unrepresentative as well. "Other uncertainty" is particularly tricky because “[unknown unknowns](https://en.wikipedia.org/wiki/There_are_unknown_unknowns)”, unlike “known unknowns”, are difficult or impossible to estimate quantitatively. The best course of action is to rely on domain knowledge (like the [paper](https://www.tandfonline.com/doi/full/10.1080/01621459.2018.1448823) I referenced above, about non-sampling polling errors), if possible.
