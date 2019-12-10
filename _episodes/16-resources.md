---
title: "Using resources effectively"
teaching: 15
exercises: 10
questions:
- "How do we monitor our jobs?"
- "How can I get my jobs scheduled more easily?" 
objectives:
- "Understand how to look up job statistics and profile code."
- "Understand job size implications."
keypoints:
- "The smaller your job, the faster it will schedule."
---

We now know virtually everything we need to know about getting stuff on a cluster. We can log on,
submit different types of jobs, use preinstalled software, and install and use software of our own.
What we need to do now is use the systems effectively.

## Estimating required resources using the scheduler

Although we covered requesting resources from the scheduler earlier, how do we know how much and
what type of resources we will need in the first place?

Answer: we don't. Not until we've tried it ourselves at least once. We'll need to benchmark our job
and experiment with it before we know how much it needs in the way of resources.

The most effective way of figuring out how much resources a job needs is to submit a test job, and
then ask the scheduler how many resources it used.

A good rule of thumb is to ask the scheduler for more time and memory than you expect your job to
need. This ensures that minor fluctuations in run time or memory use will not result in your job
being canceled by the scheduler. Recommendations for how much extra to ask for vary but 10% is 
probably the minimum, with 20-30% being more typical. Keep in mind that if you ask for too much,
your job may not run even though enough resources are available, because the scheduler will be
waiting to match what you asked for.

## Example: sharpen

We are going to explore this using a simple exercise that runs a program to sharpen an image. The
exercise description can be found at:

   - [sharpen Exercise Description]({{site.url}}{{site.baseurl}}/files/sharpen-cirrus.pdf)

and you can get the files to Cirrus with `wget {{site.url}}{{site.baseurl}}/files/sharpen.tar.gz`

{% include links.md %}
