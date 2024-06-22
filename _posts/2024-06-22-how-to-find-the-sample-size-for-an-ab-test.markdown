## Introduction

This post could be one sentence. I could tell you to use a sample size calculator (e.g., [Evan Miller’s Sample Size Calculator](https://www.evanmiller.org/ab-testing/sample-size.html)), plug in the relevant values (conversion rate, minimum detectable effect, alpha, power), copy the result, and paste it into your A/B test brief. Borrowing from quantum mechanics, we might call this approach “[shut up and calculate](https://en.wikiquote.org/wiki/Shut_up_and_calculate)”. I have no real objections to it, but it doesn’t make for terribly interesting content.

Instead, I want to interrogate the sample size formula itself. What does the equation *mean*, and what can it tell us about running A/B tests? Fortunately, it’s much simpler than the Schrödinger equation, and we can make a lot of progress in a short amount of time.

Let’s limit the discussion to “binary metrics”. These are metrics that are always zero or one. For example: a user clicks on an advertisement (1) or they don’t (0). A voter intends to vote for Biden (1) or they don’t (0). A team upgrades from a free plan to a paid plan (1) or they don’t (0). In all such cases, we end up with a proportion. In order, we have: the proportion of users who click on the ad (usually called ”clickthrough rate”, or “CTR”); the proportion of voters who intend to vote for Biden; and the proportion of teams who upgrade (sometimes called ”take rate” or “conversion rate”). Binary metrics aren’t necessarily always appropriate, but they are very common, and many important business metrics (churn, conversion, active users, etc.) are binary.

In a typical A/B test, we seek to estimate the difference in proportions between the control cell (the status quo) and the treatment cell (our attempt to improve the status quo). So, for example, we might introduce a change to the checkout flow, and we hope that the conversion rate is larger for teams on the new checkout flow than for teams on the old one. In an A/B test, we’d collect two samples, one from treatment and one from control, and run statistical calculations to estimate the difference in conversion rates and to determine whether that difference is “statistically significant”.

## Measuring a single proportion

Let’s start with something simpler, though. Suppose we just want to estimate the current conversion rate. (For simplicity, let’s assume this doesn’t vary with time.) This task requires one sample, not two, since we’re not running an A/B test. This is the formula for the “standard error” (more on that later):

$$\text{SE} = \sqrt{\frac{p (1-p)}{n}}$$

Here, $p$ is the proportion (the conversion rate), and $n$ is the sample size.

(I’ve taken this formula, and the ones that follow, from Gelman and Hill’s excellent book, Data Analysis Using Regression and Multilevel/Hierarchical Models, specifically [Chapter 20](http://www.stat.columbia.edu/~gelman/stuff_for_blog/chap20.pdf))

There are two important limits/bounds.

First, \\(p (1-p)\\) is maximized when \\(p = 0.5\\), and the formula reduces to \\(\text{SE} = 0.5/\sqrt{n}\\), or, inverting,

$$n = \left(\frac{0.5}{\text{SE}} \right)^2$$

This is a conservative estimate, or an upper bound on the sample size. The resulting formula is simple, but profound. It gives the sample size required to estimate a proportion to a desired accuracy, \\(\text{SE}\\). To estimate a proportion to within 5% (absolute), we need a sample size of roughly 100. If we need a more accurate estimate, say to within 0.5%, the sample size required is 10000. Because of this “inverse quadratic” scaling, small changes in the desired accuracy can increase the required sample size by a lot. Gelman and Hill write, “Sample size is never large enough”, and, as a data scientist, truer words have never been spoken.

Another important limit is when \\(p\\) is small. This is a common occurrence in industry, especially when it comes to proportions related to revenue. Users don’t like clicking on ads. Teams don’t want to spend money to upgrade from a free plan to a paid plan. And so on. In this limit, \\(p (1-p)\\) is well-approximated by \\(p\\), and the sample size becomes \\(n = p/\text{SE}^2\\). It appears that the smaller the proportion, the easier it is to measure, since \\(n\\) scales with \\(p\\). Unfortunately, as we’ll show, this trend is largely an illusion.

## Measuring a difference in proportions

In the case of an A/B test, we seek to estimate the difference in proportions, not a single proportion. The formula for the sample size in this case is slightly more complicated:

$$n = 2 \left[p_C(1−p_C) + p_T(1−p_T) \right] \left(\frac{2.8}{p_C - p_T} \right)^2 $$

where the subscripts \\(C\\) and \\(T\\) denote the control and treatment cells, respectively.

(The factor of “2.8” comes from the critical values of \\(z\\) for \\(\alpha = 0.05\\) and power = 0.8, and for different values of \\(\alpha\\) and power this factor will change, but the remainder of the formula will be unchanged. See Gelman and Hill’s book for more details.)

Again, let’s suppose \\(p_C\\) and \\(p_T\\) are small. Then, this formula reduces to:

$$n = 31.36 \left(\frac{p_{avg}}{p_C − p_T} \right)^2$$

where \\(p_{avg} \\) is the average of \\(p_C\\) and \\(p_T\\).  Structurally this formula is very similar to the one we found above. The sample size scales linearly with the proportion we’re measuring, and inverse quadratically with the difference in proportions between treatment and control.

If we want to measure small differences between treatment and control — small “effect sizes” — we need large sample sizes. Again, this is not surprising. To provide a sense of scale, if we want to measure a 0.5% (absolute) increase on a base conversion rate of 5%, we need a sample size of roughly 62000. In relative terms, though, this is 10% increase (= 0.5%/5%), which is a lot. (Imagine making 10% more users click on ads, or 10% more teams convert — you’d make a lot of money for your company’s investors or shareholders!). If we care about a 1% relative increase (0.05% absolute), we’d need a sample size of 6.2 million, which is likely a nonstarter, unless we’re at a large tech company.

Let’s follow the logic of “relative effect sizes” further. Call the minimum effect size we want to detect the “MDE”, and express it in relative terms. Then the sample size formula becomes

$$n = \frac{31.36 p_{avg}^2}{\text{MDE} \cdot p_{avg}} = \frac{31.36}{\text{MDE}^2 \cdot p_{avg}}$$

Multiplying on both sides by pavg, we find:

$$ n \cdot p_{avg} = \frac{31.36}{\text{MDE}^2} $$

The quantity on the left hand side is the sample size times the proportion: the number of “successes”. If the proportion is the conversion rate, this is just the number of conversions during the experiment. If the proportion is the clickthrough rate, this is the number of clickthroughs. I’ll call this equation the **“law of successes”**: measuring a particular relative increase in your desired proportion, for small proportions, **requires collecting at least a certain number of “successes”**. Measuring a 5\% increase in upgraded teams requires collecting 12500 upgraded teams. Powering an A/B test to measure a 1\% increase in clickthrough rate requires 313000 clickthroughs.

A sample size estimate, in A/B testing, usually informs the experiment runtime. If we want more sample size, usually the only option is to wait longer: for more users to become active, more teams to go through the checkout funnel, etc. So really what we are saying in the statement above is that we have to run the experiment **long enough to collect a certain number of successes**. Specifically, we need \\(31.36/\text{MDE}^2\\) units to be successful. If we want to measure a 5\% increase in upgrading teams, and we only have 1000 teams that upgrade per month, we’ll have to wait roughly a year (!) to collect enough sample size for our experiment to be powered (we need \~12500 upgrading teams).

Consider the implication for experimentation at B2B companies vs D2C companies. B2B companies make money in a much “lumpier” way than D2C companies. YouTube accumulates a small amount of money from every user watching an ad. Figma accumulates a large amount of money from every organization that signs a contract. But, because of the math above, it is much easier to run experiments at YouTube than at Figma. Powering a test to increase a YouTube revenue metric by 5% requires 12500 users to do something that makes YouTube money. That probably happens at least every hour at Youtube. Powering a test to increase a Figma revenue metric by 5\% requires 12500 *organizations* to do something that makes Figma money. That could take weeks or even months. It is perhaps not surprising that the companies with very strong experimentation cultures (Uber, Google, Facebook, Spotify, etc.) are all consumer-facing. The reason is not that these business models inherently generate more revenue. It is that these business models generate revenue in smaller increments: lots of “successes” happen every day. 

## Relaxing the small p constraint

It turns out that our previous derivation did not rely too strongly on the assumption of small p. For other values, we get:

$$ n \cdot p_{avg} = n_{\text{successes}} = \frac{31.36 (1 - p_{avg}) }{\text{MDE}^2}, \text{for } 0 \le p_{avg} \le 0.5 $$

Note that \\(1 - p_{avg} \\) is always between 0.5 and 1. So, we have a structurally similar result. We need to run an experiment long enough to collect a certain number of successes, somewhere between 0.5x and 1x the small \\(p\\) limit, depending on the value of \\(p\\). The upper bound is the previous formula. The lower bound, or best case, is where \\(p = 0.5 \\). In this case, we need “only” half as many successes. Depending on the quantity being measured, this can still be a lot of successes, though!

## A detour: overexposure/dilution

Suppose we are designing an experiment in which we make changes to our checkout flow to increase purchases. We know that roughly 0.1\% of visitors to our website end up buying something. The engineer responsible for the A/B test adopts a simple approach. Every visitor is assigned a persistent id, based on their device, and these ids are hashed so that half go to treatment, and the other half to control. Using the formula above, to measure a 5\% relative increase in “take rate” (the % of users who buy something), we need to run the experiment long enough that we have 12500 purchasers, or, equivalently, 12.5 million visitors. This seems like a lot.

A product manager contemplates the situation and has a bright idea. The issue, she says, is that we’re “diluting” our experiment with a bunch of visitors are totally irrelevant. We only care about visitors that start the checkout flow, since the user experience differs only within the checkout flow itself. This adopts the principle that “exposure should happen as late as possible”. We should delay exposing users to our experiment until the very last moment before they encounter a difference in their experience.

Suppose we ameliorate the dilution problem by a factor of 100. So, 10% of visitors who start the checkout flow end up buying something (as opposed to the 0.1\% of all visitors mentioned previously). How much does this help us reduce the required sample size? 

Using the Gelman/Hill formula above, we find that, in the first case (all visitors exposed), we need a sample size of \~12.8 million to measure a 5\% relative effect. In the second case (only visitors starting checkout flow exposed), we need a sample size of \~115000. But, since in the first case we have 100 times as many visitors, the reduction in sample size is only roughly \~(128-115)/128 → 11%. Despite fixing the dilution problem by 100x, our gain in required sample size is relatively modest.

The reason is the “law of successes”. We have the same number of successes whether we’re diluted by 100x or not at all. The only thing that changes is the number of failures: the visitors who never enter the checkout flow, and therefore never purchase. The original law of successes would tell us that the required sample size shouldn’t change under dilution; the modified law (derived above), would tell us that it should change by \\(1 - (1 - p_{avg}) = p_{avg} = 10%\\), very close to what we found above.

Recall that, even in the best case scenario, where \\(p_{avg} = 0.5\\), the required sample size is only half as much as in the small \\(p\\) limit. In other words, even if we “overexpose” by a factor of a thousand, or a million, or a billion, we only lengthen the experiment runtime by a factor of 2 at worst (assuming exposures are linear with time). (As a side note, in Chapter 20 of “Trustworthy Online Controlled Experiments”, Kohavi et al. use a numerical example to arrive at a similar result.)

Of course, overexposure/dilution should still be avoided if possible. For example, non-binary (continuous) metrics tend to behave worse under dilution than binary metrics. The point of this digression was simply to show that overexposure isn’t the enormous problem it’s sometimes made out to be. Also, there are sometimes countervailing considerations that favor “earlier” exposure. In particular, it’s often easier for engineers to implement exposure earlier rather than having to check that each of the conditions required for “late” exposure are met, which might risk introducing other bugs.

## Guardrail metrics

You might argue that companies like Figma don’t just care about revenue-related metrics, or, in general, binary metrics with infrequent successes. They run experiments trying to get users to retain better, to increase their usage frequency, to take more actions in the app, etc. For these experiments, they might be able to employ “normal” experimentation, with much shorter runtimes.

Not so fast! Experiments have both “success” and “guardrail” metrics. We might want to increase “user actions” (a success metric) by 5\%, but not at the cost of harming revenue by 1\% (a guardrail metric). **Guardrail metrics should be included in the sample size calculation too**. Strictly speaking, the required sample size is the *largest* of the required sample sizes for each of the metrics under consideration. It is often the case that a success metric requires a small sample size to power, but a guardrail metric requires a much larger sample size (for the reasons explained above). Typically the “solution” is to relax the MDE for the guardrail metric. But this is highly unsatisfactory. If we truly want to guard against our revenue falling by 1%, we should power our revenue guardrail metric using an MDE of 1\%, not 5\% or 10\% or whatever makes the sample size calculation “work”. Otherwise, we are simply fooling ourselves. We might argue that our experiment should have no effect on revenue, but our intuition is often wrong, and experiments can have unintended effects on seemingly “irrelevant” metrics.

All of this is to say that the law of successes is more far-reaching that you might initially think. Even in the case of “non-inferiority”, where we are simply trying to avoid harm to a “rare” binary metric (as opposed to “superiority”, where we are explicitly trying to increase it), we need to collect a possibly prohibitive number of “successes” to ensure we do no harm. As I mentioned in [a previous blog post](https://varunprajan.github.io/blog/the-bayesian-euro-step/), this is one reason why even “big tech companies” often struggle to adequately power many experiments. 

## Conclusion

Sample size calculations are usually treated as a “shut up and calculate” exercise. The formula and its derivation are opaque, and we often outsource the work to a convenient online calculator. In this essay, I tried to demystify sample sizing for the case of binary metrics, focusing specifically on the “small p” limit, which is of importance for rare events like conversions and clickthroughs. (I later relaxed the “small p” condition.)

I derived the “law of successes” — not an original contribution, but one that I have rarely seen discussed in books and essays. It says that, to measure a particular MDE (in relative terms), we need to run the experiment long enough to collect a certain number of successes. The number of failures, or irrelevant users (as in the overexposure/dilution scenario), doesn’t matter much: only a factor of 2, at most. The required number of successes can often be prohibitive, since it scales poorly with the MDE. For example, getting 1000 teams to upgrade per month is challenging at most startups, and, even with that rate of success, it requires a **year-long** experiment to adequately power a 5\% relative MDE! This fact explains why experimentation is hard at companies where success happens in large “lumpy” increments rather than small frequent ones. Even when these companies are not trying to increase a revenue-related metric, they still (ideally) need adequate sample size to guard against harming it.
