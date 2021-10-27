---
layout: posts
title:  "Forecasting a feature group"
categories: product
author: Olayinka Ola
---

As a product manager, I am usually provided with a list of features and tasked with forecasting what can be done when.

My first step step is to work with the product & development team to document all proposed epics. The following structure of headings works very well.

- Epic - a large user story that cannot be delivered as defined within a single sprint
- Benefit (per month) - The value - typically in monetary terms - that can be unlocked by the feature group.
- Feature - Functionality or experience enhancement that satisfies a customer need.
- Story Point - The estimated size of the feature.


With a good understanding of the work to be completed the next step is to estimate the size of the project. To do this efficiently I extrapolate the size of all features in this project by sampling seven stories at random.

Here I use the random number generator provided by [omnicalculator](https://www.omnicalculator.com/statistics/random-number)  to generator 7 non-repeat numbers.

With seven stories identified at random (2, 4, 7, 10, 12, 13, 20). It's time as a product team to sit down with the development team to settle on story estimates for each feature. Typically the best way to reach a satisfactory estimate is to leverage the development teams' experience by simply referencing similar completed features.

Simply put we - the product team and dev ops - discuss where this new feature should sit in comparison to similar epics/stories.


“It all comes down to your Development Team as they are doing the estimation and the work. The less stable the Development Team, the less precision, resulting in a higher variance in the estimates and velocity.”
    - Excerpt From: Don McGreal. “Professional Product Owner, The: Leveraging Scrum as a Competitive Advantage, First Edition.

This approach works best when the development team is largely unchanged. In scenarios where you are working with a new team or when the requirement is completely new, then a rough guess is the way to go.

Here are my seven features. Each has a story estimate and short rationale where appropriate.


Now I enter my teams estimations into the [story forecaster worksheet](https://github.com/FocusedObjective/FocusedObjective.Resources/tree/master/Spreadsheets).

## Estimated Feature List

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Estimation1.png" alt="">

## Story Forecaster

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/estimation results.png" alt="">

## Results

For my **21 feature list,** assuming there are no inefficiencies during the sprint that would make story splitting a problem. I can report to the client or sponsor with a 95% certainty that the identified enhancements will cost around 135 story points.

If the average velocity of the current development team is well established then this forecast can be conveyed to the relevant stakeholders in sprint content.

For example

IF Average velocity = 25

THEN 135 / 25 = 5.44

SO this feature list should take around 6 sprints to deliver.

# Summary
In this post, I have demonstrated how to forecast story points. This technique helps answer questions about what we can / should do.

## References

Professional Product Owner, The: Leveraging Scrum as a Competitive Advantage, First Edition Don McGreal

McDonald, K., 2021. *Themes vs Epics vs Features vs User Stories*. [online] KBP Media. Available at: <https://www.kbp.media/themes-epics-features-user-stories/> [Accessed 26 October 2021].

[response-variable]: https://github.com/hightowerr/notebooks
[here]: https://github.com/hightowerr/notebooks
