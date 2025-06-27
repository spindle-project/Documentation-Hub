---
type: page
title: The Pre-Processer
listed: true
slug: the-pre-processer
description: 
index_title: The Pre-Processer
hidden: 
keywords: 
tags: 
---

{% callout type="info" title="About the Pre-Processer" %}
The Pre-processor is the first step in abstracting user code so that it can be ran, specifically, it serves to standardize code and format it as a string. Both of which are important for the next steps.
{% /callout %}

Diagram of what the Pre-Processer does: 

{% badge type="warning" text="Unformatted User Code" /%}  → {% badge type="success" text="Formatted User code." /%}

{% callout type="success" title="Did you know" %}
Fun fact: Spindle code does not actually require the pre-processer to run, all it does is reformat code. However, please note that you will receive immediate issues and sanity errors with if-statements and functions.
{% /callout %}

That being said, the Pre-Processer has two important tasks:  

1 - Adding an empty else statement to all if statements if they do not have one

2 - Separating function definition and function execution