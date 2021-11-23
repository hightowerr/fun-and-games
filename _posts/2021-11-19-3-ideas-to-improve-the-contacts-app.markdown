---
layout: posts
title:  "Improving Apple's contacts app"
date:   2021-11-19 13:00:00 +0100
categories: cleaning
author: Olayinka Ola
---
Contacts is a computerised address book included with the Apple operating systems iOS, iPadOS and macOS. It includes various cloud synchronisation capabilities and integrates with other Apple applications and features, including iMessage, FaceTime and the iCloud service. The core functionality of Contacts hasn't changed much since its release in 2001

Contacts is exclusive to iOS users. It primary function is to store contact details for its owner. There are a wide range of personas that use the product as it is vital for connecting people and maintaining relationships.

For more information on what it does - Contacts Features & Integrations are mentioned [here][here]

## **Personas**

Here i brainstormed a short list of potential personas that use the product:

- **Self employed tradesperson** - tradespeople have their phones glued to the hips. It’s essential for acquiring new clients and retaining existing ones.
- **Captain of the local football team** - local team captains are constantly contacting their existing pool of players to confirm their availability. At the same time they are in contact with new players to ensure they can field a strong 11 for Saturday.
- **The best man organising a stag** - Is handed a group of contacts by the groom. Tasked with coordinating, organising and managing a potentially large group of people for a short period.

For brevity I will focus on one segment from those listed above.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/beers.jpeg" alt="beers">

### Billy the Best mans needs

| Name               	| Behaviours                                                               	| Needs and Goals            	|
|--------------------	|--------------------------------------------------------------------------	|----------------------------	|
| Billy the Best man 	| Coordinates the group of people                                          	| Add contacts quickly       	|
|                    	| Organises a series of events                                             	| Share content easily       	|
|                    	| Communicates with group largely  made up of strangers on a regular basis 	| Quick clean up of unnecessary contacts 	|


#### **Onboarding Contacts**

As best man, I want to selectively bulk import contacts details so that I have only the information I need to plan activities effectively.

#### Share content easily

As a best man, I want to share content with a specific group of contacts so that I can easily inform the group or selected individuals.

#### Quickly clean up contact list

As a best man, I want to keep my personal contact list tidy so that I don't store useless contact details on my phone.

## Prioritization
Contacts is included with the Apple operating systems iOS, iPadOS and macOS so the value of these potential features cannot be measured through a thoughtful and quantitative approach.

My alternative approach is to base my prioritisation criteria on subjective grade (1 - 5). The focus will be on customer impact as the goal for delivering value quickly.

| ID 	| Feature                          	| Dependency 	| Size 	| Impact 	| Weight 	| Preferred Order 	|
|----	|----------------------------------	|------------	|------	|--------	|--------	|-----------------	|
| 1  	| Selective contact details import 	|     --     	|   3  	|    3   	|  3.33  	|        3        	|
| 2  	| Selective contact sharing        	|     --     	|   2  	|    2   	|  2.60  	|        2        	|
| 3  	| Quickly clean up contact list    	|     --     	|   2  	|    4   	|   4.50  	|        1        	|

- **Dependency** -  ensure features are prioritised sequentially
- **Size** -  estimate of the engineering effort necessary to complete a story
- **Impact** - measures the value to the user
- **Weight** - calculated value: weight = impact + (1/size)
- **Preferred Order** = Priority order

With a clear order of priority it is time to get creative. The most important item on my list is # 3 Quickly Clean up contact list.

My first task is to isolate the problem. Here is my problem statement.

*“Customers don't want to store unnecessary contact details once they are no longer required"*

Here are some to ideas to help solve this problem.

1. **Adapt contacts to include expiry date setting** - In the contact profile add an expiry date option. When the user defined date is reached then the contact is removed.
2. **Proactively suggest contacts for removal** - A periodic clean up notification is sent to customer via in-app message suggesting contacts to be removed. Customer can execute the action with one call to action click.
3. **Modify how contacts are added** - New contacts can be are added into 'smart' groups. A clean up workflow can be executed based on group definition.

These ideas aren't perfect and so here are some pros and cons.

## Evaluate Trade offs
<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Trade-offs.png" alt="trade-offs">

My recommendation would be to adapt the contact entry form to include an expiry setting. This setting will allow a user to set an expiry date for a new or existing contact. When the date set by the user is reached a workflow removes that contact from the contacts list. This feature would help users trim their contact lists by automatically removing contact details once they are no longer required.

In my opinion this is the best option because unlike proactively suggesting contacts for removal. Setting and expiry date is less presumptive and will not result in more notifications being sent to customers.

Additionally by requiring the user to mandate this change at the contact level, signals a clear call to action rather than a generalised one size fits all group approach to contact management embedded within a 'smart' feature.

[here]: https://en.wikipedia.org/wiki/Contacts_(Apple)
