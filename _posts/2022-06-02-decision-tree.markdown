---
layout: posts
title:  "Optimising decision making"
date:   2022-07-10 09:00:00 +0100
categories: python
author: Olayinka Ola
---
Decision trees a way to figure out and clarify complex problems before committing considerable money, time and resources to a project. Below is an application of a decision tree to a decision on whether to conduct an app UX/UI redesign.

### **Problem Statement**

A company app has not had UX/UI resources for a long time and is underperforming.

### **Task summary**

- Need to decide whether to redesign the mobile app through an external resource.
- Boss has asked for an MVP first.
  - The budget for the MVP is negotiable but should be kept to a minimum.
- Doing the MVP helps identify whether the full redesign will be successful. We expect MVPs to identify worthwhile projects 80% of the time.
- The MVP will also identify ineffective projects 90% of the time.

### **Expected Output**

- Define the optimal strategy?
- Define the associated expected monetary value (EMV).

For this analysis, I am using the Decision tree from the Radiant Add-in for R.

[Github Repo][Github Repo]

### Initial assumptions for building my decision tree

The computation of many numbers involves other numbers, so I prefer to define numerical values as variables.

- The fixed cost of redesigning the app is £150,000
- £15,000 will be used as the baseline cost of the MVP
- The probability of the redesign being successful is 60%. The probability of failure is 40%
- Given the MVP is successful the probability of a subsequent full redesign succeeding is 80%. The probability of a redesign failing given a successful MVP is 20%
- Given the MVP fails a full redesign would almost certainly fail (90%) resulting in a slim 10% chance of succeeding.
- The benefit of using variables means the remaining 6 formulas concerning joint and posterior probabilities are easily taken care of.
- The payoffs are defined as 500k if successful and -£100k if the project fails
- There is a £5k cost to roll back changes if we decide not to launch following the MVP.

### Variables

##### Building the tree

With my variables set I am going to build the tree. My first node is a three-option decision node.

- Do redesign
- MVP
- No redesign

## £15k Assumed Trial Cost
```
Decision tree input:
name: invest
variables:
    fixed cost: 150000
    trial cost: 15000
    p(s): 0.6
    p(f): 1 - p(s)
    p(+|s): 0.8
    p(-|s): 1 - p(+|s)
    p(-|f): 0.9
    p(+|f): 1 - p(-|f)
    p(+): p(+|s) * p(s) + p(+|f) * p(f)
    p(-): p(-|f) * p(f) + p(-|s) * p(s)
    p(s|+): p(+|s) * p(s) / p(+)
    p(f|+): 1 - p(s|+)
    p(f|-): p(-|f) * p(f) / p(-)
    p(s|-): 1 - p(f|-)
    payoff_s: 500000
    payoff_f: -100000
    payoff_nl: -5000
type: decision
Do redesign:
    cost: fixed cost
    type: chance
    success:
        p: p(s)
        payoff: payoff_s
    failure:
        p: p(f)
        payoff: payoff_f
MVP:
    cost: trial cost
    type: chance
    MVP positive:
        p: p(+)
        type: decision
        launch:
            cost: fixed cost
            type: chance
            success:
                p: p(s|+)
                payoff: payoff_s
            failure:
                p: p(f|+)
                payoff: payoff_f
        no launch:
            payoff: payoff_nl
    MVP negative:
        p: p(-)
        type: decision
        launch:
            cost: fixed cost
            type: chance
            success:
                p: p(s|-)
                payoff: payoff_s
            failure:
                p: p(f|-)
                payoff: payoff_f
        no launch:
            payoff: payoff_nl
No redesign:
    payoff: 0
```

### App redesign

There is a £150,000 fixed cost of doing a complete redesign and there is a defined chance of the redesign being a success or failure

- Probability of a successful redesign is 60% with a payoff of £500k
- Probability of an unsuccessful redesign is 40% with a payoff of -£100k

### MVP trial

For the trial, I enter a chance node for either a positive or a negative outcome. I enter the probability of reaching this positive outcome followed by a success and failure decision nodes. The success scenario is the same as the app redesign above.  

- Probability of successful redesign given a positive trial is 92.31% with a payoff of £500k
- Probability of unsuccessful redesign given a positive trial  is 7.69% with a payoff of -£100k

The failure sub-tree for the trial is very similar to the the positive. Its probabilities are as follows.

- Probability of successful redesign given negative MVP is 25% with a payoff of £500k
- Probability of an unsuccessful redesign given negative trial is 75% with a payoff of -£100k

### Output

Below is my calculation of the tree.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/decision_tree.png" alt="full model">

The thicker lines represent the optimal strategy which is as follows

- Run the trial
- If the trial is positive then launch a full app redesign project
- If the trial is negative then do not launch a full app redesign project.
- The expected Monetary value of this strategy is £140,600k


## Sensitivity Analysis

Next, I'd like to conduct three types of sensitivity analysis.

- How sensitive is the decision to the changing costs in running the MVP
- How sensitive is the decision to the changing probabilities of success
- How sensitive is the decision to the changing probabilities of success given a positive MVP.

### Sensitivity: MVP cost.

In the sensitivity menu, I see how viable an MVP is as the cost of running it changes.

- The lowest end of my sensitivity is £10k, the max is £60k and I'd like the sensitivity to be done at £5k increments

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/sensitivity_trial cost.png" alt="full model">

This graph tells me that as the MVP trial cost doesn't exceed £45k then an MVP trial is worth doing. £45k is where the MVP trial cost intersects with the redesign cost line.

## Sensitivity: How sensitive is the decision to the probability of success
In this sensitivity I see how viable the redesign work is as the likelihood of its success changes.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/sensitivity_sucess.png" alt="full model">

This graph tells me

- If we feel the probability of success will be very unlikely (say around 12.5%) then we shouldn't redesign the app full stop.
- If we feel the range of success is uncertain and sits somewhere in the 15 - 65% range then conducting an MVP first makes sense.
- If we are very confident (> 70%) that a redesign will be a success then it's a no brainer. We should head straight to redesign and collect our £200k or more payoff.

## Sensitivity: Probability of success given a positive MVP  
In this sensitivity, I see how sensitive the decision is to changes in the probability of success given a positive result for the MVP.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/sensitivity_sucess_positive.png" alt="full model">

- If past MVPs have been a poor predictor of project success or failure, then they are not worth pursuing. In the example above the MVP's success, predictive power would need to meet a threshold of at least 83%. If after some research and internal discussions into past MVPs we realise that MVPs have a poor record of predicting success - quantified as any calibrated estimate that falls below the 83% threshold. Then the decision is to skip the MVP and risk a full redesign with a £110k potential payoff or scrap the idea altogether and go back to square one.

## Useful Links
[Radiant Tutorial Series](https://youtube.com/playlist?list=PLNhtaetb48EdKRIY7MewCyvb_1x7dV3xw)

[Github Repo]: https://github.com/hightowerr/data_analytics
