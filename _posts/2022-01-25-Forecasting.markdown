---
layout: posts
title:  "Forecasting work using the size growth & pace model"
date:   2022-01-23 13:00:00 +0100
categories: Forecasting, product,
author: Olayinka Ola
---

I love this definition of forecasting by Troy Magennis:

<aside>
💡 Forecasting is carefully answering a question about the future, to a transparent degree of certainty, with as little effort as possible
</aside>

In the book ‘Superthinking’, the authors propose a set of characteristics they believe make good forecasters

- **Practice:** It makes perfect.
- **Collaboration**: A group of experts typically outperform an expert.
- **Domain expertise**: The more you / the group know about the area of focus the better.
- **Intelligence**: When you have knowledge gaps work hard to get up to speed quickly.
- **Open-mindedness**: Stay open-minded throughout and avoid confirmation bias
- **Don’t rush**: Take time to calibrate your inputs
- **Look to the past:** Use recent relevant data inputs wherever possible
- **Incorporate New Information**: Revised the model based on new information

In this post, I want to demonstrate how to forecast date and/or duration at different likelihood percentages. To do this I will use the Throughput forecaster spreadsheet developed by Troy Magennis which you can download here for free: [https://github.com/FocusedObjective/FocusedObjective.Resources/raw/master/Spreadsheets/Throughput Forecaster.xlsx](https://github.com/FocusedObjective/FocusedObjective.Resources/raw/master/Spreadsheets/Throughput%20Forecaster.xlsx)

The Throughput Forecaster spreadsheet is a simple Monte Carlo simulation forecasting tool. Given my inputs the spreadsheet hypothetically completes all work 500 times giving me a range of completion date outcomes at varying confidence levels.

The model is based on the growth [Size-Growth-Pace Model](https://medium.com/forecasting-using-data/size-growth-pace-model-b6bbf73249c8) laid out by Troy Magennis in his draft book. With it, I will compute the duration a use case will take to complete and how much work my team can complete in a given period.

For inspiration I will use the fictitious Pegasus bike trainer used in Agile Model-Based Systems Engineering Cookbook: Improve System Development by Applying Proven Recipes for Effective Agile Systems Engineering a book by Bruce Powell Douglas. The product is modelled on the [Wahoo Kickr Bike](https://uk.wahoofitness.com/devices/indoor-cycling/smart-bikes).

## The Full Model
<img src="{{ site.url }}{{ site.baseurl }}/assets/images/The Full Model.png" alt="full model">

The model facilitates using historic or estimate inputs for throughput. I will use estimates.

1. **Start Date** - This optional input is the planned start date for the forecast.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Start Date.png" alt="Start Date">

2. **Stories left to complete (High - Low bound)** - This is the size portion of the model input, it is the range estimate of the work needed to deliver a fitness bike to the customer. Estimating simple features is the goal so I need to split the use cases into smaller batches. My unit of measure will be the count of stories because it supports ranges and requires low effort. This is the first time the team is building the product so we opt for a low high range estimate with 90% confidence. Here are my inputs.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Stories left to complete.png" alt="Stories left to complete">

As an additional input, I specify that at this stage there is still some uncertainty over the size of the work therefore a multiplier is added to my estimates.

3. Next, we have the growth part of the model. For us the more we complete the more we learn about what we will need to deliver. Also inevitably defects happen before deployment so they too must be factored in. After brainstorming for items most likely to cause new work we consolidated and agreed on the following inputs.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/split rate.png" alt="split rate">

4. Onto the pace portion which models the sustainable completion pace of the team. To calculate appropriate ranges for this I put the following points on the agenda for the team meeting.

- How many people do we have who can work at the constraint? (A)
- Set a low & high item per sprint guess (B)
- Define low & high pace estimates for the team (C,D)
- Discuss how long ramp-up & ramp-down will take and pace the impact (E,F,G,H)

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Pace inputs.png" alt="Pace inputs">

Once the most likely constraints are identified we specify the resource available to work on any constraints and the capacity per person. For the fitness bike, we discuss the expected velocity of the team, as this is a new product line the development pace is likely to ramp up & down slowly to facilitate learning and testing.

My workstream delivers value in 2-week sprints as this is a new product and the development team is a mixture of experience and new hires so there is no reliable historical reference to go on. Velocity data ranges are estimated. Using the above pace inputs and the following formulas.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/pace calculations.png" alt="pace calculations">

Here are my pace estimate inputs.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/pace throughput.png" alt="pace throughput">

## The Results
<img src="{{ site.url }}{{ site.baseurl }}/assets/images/results.png" alt="results">

With these inputs, I can confidently advise that the end to end development will take roughly 26 - 31 weeks. If pushed to narrow my range I can settle on a percentage likelihood that I feel comfortable with and so whether I feel like being cautious or confident my feedback can be presented accordingly. This isn’t the one tool to rule them all but through it, I can express my viewpoint on delivery timelines simply and concisely shaped around size, growth and pace.

In this post, I have used the throughput forecaster tool to forecast how long it will take to complete development on a product. using monte carlo simulation and the [Size-Growth-Pace Model](https://medium.com/forecasting-using-data/size-growth-pace-model-b6bbf73249c8) I can achieve this with as little effort as possible. It is a tool that must be maintained through regular feedback loops and its outputs can be valuable inputs for upstream and downstream processes.

Extensions
- Feed actual data into the model to continuously improve the forecast.

Learn More

- Weinberg, G (2019). *Super Thinking: Upgrade Your Reasoning and Make Better Decisions with Mental Models*.
- Magennis, T. (2017). *Sampling, probability and certainty.* Available: https://medium.com/forecasting-using-data/sampling-probability-and-certainty-3e105e065138. Last accessed 25th Jan 2022.
