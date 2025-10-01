---
type: page
title: Getting Started with Spindle
listed: true
slug: getting-started-with-spindle
description: 
index_title: Getting Started with Spindle
hidden: 
keywords: 
tags: 
---

{% callout type="warning" title="Wait a second, Partner!" %}
This is Spindle's developer documentation. It should only be read by people who want an idea of how Spindle works under the hood. This documentation assumes knowledge in how to write and execute Spindle code.  If you'd like help with how to write code in Spindle, please visit our user documentation here: [https://spdl.netlify.app/docs/user/](https://spdl.netlify.app/docs/user/)
{% /callout %}

Spindle is a program written in Python that executes AP Computer Science puesdo-code, allowing students, learners, and teachers to see its execution in action - better preparing them for what to expect on test day. However, the process to execute it is more complicated of a task then it might seem. Here is a tour of that process. 

{% callout type="info" title="Info" %}
For the sake of convince, lets assume the user has inserted the following snippet of code
{% /callout %}

{% code %}
{% tab language="javascript" title="Spindle" showLineNumbers="true" %}
# Write your Spindle code here
            
PROCEDURE calculate_sum(numbers) {
    sum <-- 0
    REPEAT LENGTH(numbers) TIMES {
        sum <-- sum + numbers[i]
    }
    DISPLAY(sum)
}
calculate_sum([1,2,3,4])
{% /tab %}
{% /code %}

The above function takes displays the sum of a given array of numbers. 

When a user executes Spindle code, whether that is by pressing the "run code" button on the website IDE or by running it on their terminal, the code is first handed over to the Pre-Parser. Depending on the method the user uses to execute Spindle code, this process differs. 

{% table widths="383" %}
| On the Website's IDE | On the Terminal | 
| ---- | ---- | 
| PyScript, the library that runs python projects like Spindle in the browser, runs the `semi_parse_string()` function, giving it the code snippet as an argument. | The code snippet is interpreted as a text string and is given to the `semi_parse_string()` function as its only argument. | 
{% /table %}

Once the function is ran, the code is interpreted by the Pre-processor as a string. 

[auto$](/spindle/the-pre-processer)