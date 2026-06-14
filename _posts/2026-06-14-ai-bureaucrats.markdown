---
layout: post
title:  "AI and the bureaucrats"
date:   2026-06-14 05:07:59 -0500
categories: jekyll update
---

## Introduction

I was applying for an apartment in Brooklyn in 2023, and the owner insisted I submit personal and professional letters of reference. He wanted to know what my friends and coworkers thought of me. I found it an unusual request, based on my 10 years of apartment hunting in NYC, but I liked the place, so I indulged him.

In the “Before ChatGPT/Claude” ([BC](https://bsky.app/profile/ethanrosenthal.com/post/3mmplkhnyys2s)) era, this kind of request would have (collectively) wasted several hours. In the “AC” era, on the other hand, it took just a few minutes. My reference writers prompted ChatGPT with a few facts about me, and, in short order, they obtained the kind of breezy boilerplate that LLMs excel at.

Here’s my thesis: AI undermines bureaucracy. Everywhere you look, there are interactions where one party wants another party to go through a series of steps: a bureaucratic process. The former cares that the process is followed, whereas the latter doesn’t really care at all. They simply want to speed through, and get to the end. AI makes that, in many cases, trivial.

## Some examples

For example: your manager wants weekly status updates posted in a Slack channel (an “async standup”). Before Claude, you might have spent 5 minutes trying to remember all the things you did in the last 7 days, and another 5 minutes writing them down in a digestible way. Now, you can have Claude connect to your Jira and Slack via MCP, comb through your tickets and messages, and create a neatly bulleted list of your weekly accomplishments, with emojis for a little extra flair.

Or: you want to get a PR merged, but the repository enforces a test coverage requirement and requires a codeowner to approve. Before Claude, you might have spent hours handcrafting tests and going back and forth over nits and code quality in a Github conversation. Now, you can have Claude create the tests, push up the code, write the PR description, and argue with the codeowner over a few rounds of review.

Or: there is a requirement at your workplace that an experiment design be approved by a panel of data scientists. The design document must adhere to a template, standardized for all experiments. Before Claude, you’d might have devoted an hour or two to filling out this template, and another hour or two to responding to feedback and adjusting the experiment design accordingly. Now, once again, AI can handle essentially the entire workflow. Someone might devise an Agent Skill to help the agent fill out the template using the information in the project’s Slack channel. The agent, armed with this skill, can probably even respond to DS comments.

Or: you want to launch a new feature but your data science colleague insists that it be instrumented/logged thoroughly. You don’t care about logging much — after all, you don’t consume the data — but this request becomes launch-blocking. Before Claude, you might have spent a few hours (or even days!) writing a “logging spec”, receiving feedback, rewriting it, implementing the logs in code, getting those changes deployed, emitting the logs in dev, seeing if they populate in the dev data warehouse, verifying with DS that they look correct, etc. Now, once again, Claude can do a decent job of handling each of these steps.

Or: you’re a PM who wants to get your pet idea prioritized for the Q1 roadmap. You need data to convince your boss, the PM director, that this effort is worthwhile. Before Claude, you might have conferred with your data science partner. They might have told you that, actually, you’re wrong, and the expected $ impact is minimal. Now, you have Claude do its own research, and write a ten page data analysis that looks legitimate but no one has time to read or scrutinize.

Or: you want a job as a data scientist. Before Claude, you’d have invested dozens of hours to clean up your resume, practice Leetcode, and master case studies and metrics exercises. Now, you can fake your way through many interviews using AI.

Or (finally): you’re building a new feature, but your PM tells you they want user research to confirm you’re building something users want to use. Before Claude, you might have bothered a user researcher, who’d take weeks to set up a study, interview users, collect data, and make a shiny presentation, full of nuanced findings. Now, you can simulate those interviews by using skills to create different Claude personas, each of which respond to the feature differently. You can also use Claude to synthesize the results from these synthetic interviews, and to make the shiny presentation yourself (no nuance required).

(As you might have guessed, these are all real examples (with some literary license) from my workplace in the last 6 months.)

## On bureaucracy

Why does bureaucratic process exist? I’m certainly no expert, but there seem to be a few (overlapping) reasons.

The party who cares (the “bureaucrat”) ostensibly gets **value** from the process. The manager wants visibility into her team’s work. The landlord wants to ensure he’s renting to a decent person. The codeowner wants to establish a minimum quality bar. The PM director wants to appropriately prioritize scarce resources. Etc.

The disparity in caring is largely driven by a disparity in value. The bureaucrat cares because they derive value from the bureaucratic process. The non-bureaucrat (you) doesn’t because they are given little incentive to, at least in the short-term.

In some cases, the lack of value and incentives for “good behavior” is stark. What reason would a candidate have to care that they are undermining the hiring process?

In other cases, caring requires long-term thinking and putting the collective interest first. Many non-bureaucrats have these traits, but many don’t. Ultimately, poor experiment designs and poor logging will slow down product development. Each experiment will be more likely to be improperly set up, and each experiment will yield less insight about user behavior. But those problems might take a while to uncover, and, of course, we never know what the counterfactual would be. In the meantime it looks like you’re moving much faster, cranking out more commits and code and PRs and prototypes and features.

Bureaucracy is also about establishing **commitment**. A renter who goes through the effort of collecting several references is probably a much more serious candidate than one who doesn’t. The bureaucratic process creates **friction**, and this is sometimes a desirable outcome, not an undesirable one.

My belief with experiment design reviews, for example, is that much of the value is actually in filling out the template, not in the review process. Filling out the template forces you to think about the audience, the targeting, the randomization unit, the exposure point, the metrics, and the decision criteria. Those are probably things you weren’t thinking about before, or at least not carefully. The process forces you to pay **attention**. By farming out this work to Claude, you don’t have to pay attention, and consequently get little of the value.

There was a [nice skeet](https://bsky.app/profile/eugenevinitsky.bsky.social/post/3mnrulpyhyk2j) from Eugene Vinitsky on this topic: “The one thing I do know is that reading the solutions feels like learning and it is never is.” The reason we have homework is not that the instructor gets value from grading it. It’s that by doing homework, you learn the material much more deeply than you would have by asking Claude for the answers.

I don’t mean to imply that all bureaucracy is good (it’s not!). But seeing Claude used, and abused, at my workplace has clarified something I never fully grasped before: **I am a bureaucrat**, both temperamentally and organizationally.

Temperamentally, I want people to follow the rules, to think about what they’re doing, and to maintain high standards. I shudder every time I see an AI-generated message addressed to me, and, [like Eugene Vinitsky](https://bsky.app/profile/eugenevinitsky.bsky.social/post/3mnsoqboyqs2m), I believe this is also an implicit breaking of the “social contract”. 

Organizationally, I’m usually in the position of relying on others to do a good job, of reviewing their work, and of being a blocker for them to ship. I’m the bureaucrat who insists on experiment design reviews, who analyzes whether a feature is worth pursuing, who depends on good instrumentation, who has to say no when a p-value is 0.06, and who leaves long reviews on shoddy PRs. I’m sure a lot of my colleagues think I’m annoying, and they’re probably right! Using AI to circumvent or undermine my bureaucracy is one way of making me less annoying, but I’m skeptical it makes the organization more effective, even if certain individuals appear more productive.

Silicon Valley thought leaders [emphasize](https://www.nytimes.com/2026/04/01/opinion/high-agency-silicon-valley.html) “agency”, specifically “high agency”, these days. “You can just do things”, they say. The best way to “just do things” is to ignore the barriers and rules that stand in the way. There are the bureaucrats, on one hand, and the builders, on the other.

Perhaps this is why some of the worst people in the world have thrilled to AI: it helps circumvent and undermine all the people who have been (apparently) holding them back. (I don’t mean this as a criticism of AI itself, which is a tool with legitimate applications. It’s rather a rebuke of the people who think that the problem with tech, and the world, is that we’re not moving fast enough and breaking enough things.)

## What to do?

Here are a few suggestions:

**Bad processes should be reviewed**. In areas where Claude undermines bureaucracy, we should ask ourselves whether the process needs to exist, or if there are better ways to structure the process. For example: does a manager need a weekly async update from their team? (e.g., if they already have 1:1s with each team member). We have to consider the costs/benefits to both the bureaucrats and the non-bureaucrats/builders.

**Better tooling can help**. Some of the areas where we’d like to move faster — self-serve data analysis, better instrumentation — would genuinely benefit from better tooling: canonical datasets, easy ability to verify instrumentation is implemented correctly, etc. These are all capabilities that “data tooling” teams can help build. These kinds of investments can help us all move faster, without destroying the value that bureaucratic process is intended to create. See more [here](https://varunprajan.github.io/blog/vision-ai-data-work/).

**Bureaucrats should fight back**. We should move back to in-person interviews. (Seriously.) If the quality of async updates becomes low, we should reintroduce live standups. Experimentation reviews should also be conducted live, instead of in Slack threads. There should be organizational norms (shame, or worse) for trying to fake data analysis or user research, or for wasting people’s time with bad AI writing or code. We’re already seeing this in open source, where, for example, many repository owners been overwhelmed with AI slop and have [shut down](https://www.techtimes.com/articles/317463/20260531/flathub-ai-ban-blocks-every-submission-type-flatpak-repos-enforcement-hits-open-source-hardest.htm) unsolicited AI submissions.

**People need to re-learn being comfortable thinking for themselves**. Even I’ve succumbed to the temptation of asking Claude for the answer when I don’t feel like thinking. But this is often a bad habit, and one that we ought to unlearn. The point of bureaucratic process is in part to force us to think, to pay attention. If something genuinely requires your attention, you're only doing yourself a disservice by substituting Claude’s “thinking” for your own.
