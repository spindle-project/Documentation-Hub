---
type: page
title: Text Sanitization: The Pre-Parser
listed: true
slug: the-pre-processer
description: 
index_title: Text Sanitization: The Pre-Parser
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

please expand this and turn it into md. give me the raw md format. No HTML:

            tips_and_updates            

**Pro Tip:** You're on a the documentation for Spindle's develepers, if you'd like to know how Spindle works, read on! If you'd like to know how to write Spindle code, go to [here](https://spdl.glitch.me/docs/user)!            

{% callout type="info" title="Info" %}
This document details how the project itself works, not how to write code in it.       If you would like to know how to write code in Spindle, please look at the user documentation!        
      With that out of the way, this document will lay out how Spindle works and will help you contribute to it.
{% /callout %}

## All about the Semi-Parser

      When you write code on using the spindle, you are told to run it though the       `shell.py` file. This file exists to provide a front end for       the language and to feed your code into the interpreter (in the       `spindle.py` file) but is not necessary.      

### Running a file with the RUN("") command.

      All Spindle code starts out as a string of text. The very first thing       Spindle does is get it ready to run. This step of the process is called       Semi-parsing and does three very critical things.
      The first thing the semi-parser checks for is a RUN("") command. This is       **not**       apart of the college board standards but is necessary for the desktop       experience. This is called a "command" because it is technically       _outside_       of the spindle language but included in the file. RUN() takes one       parameter, which is the full name of the file that you would like to run       (including all extensions) as a string. When you execute a RUN command,       Spindle will look for a file with that name, and feed that code into the       semi-parser. If there is no RUN command, this process is skipped and the       code written directly in the terminal is ran instead.

---

### Standardizing IF statements

      The second thing the semi-parser does is look at your program code and add       "ELSE{}" to any of your if statements that do not have an else statement.       It does this to fix a critical bug where an if statement without an else       causes the program to       _nullrun_, which is where spindle stops running code completely, or       return a       _fauxerror_, which is where spindle returns an error that simply       doesn't make sense.

---

### Divide and Conquer Functions

      The last, and most major thing the semi parser does is check whether your       program has a function in it. If and only if your program has a function       in it, the semi parser will chop your code into pieces in such a way that       function definitions are isolated. Each piece will then be fed into       Spindle.

---

      In a sense:    

{% code %}
{% tab language="none" %}
PROCEDURE add(a,b) {

        IF (a == b) {

        RETURN

        }

        RETURN a + b

        }

        DISPLAY("hi")

        add(10,40)

        PROCEDURE sub(a,b) {

        RETURN a - b

        }

        sub(40,10)

       
{% /tab %}
{% /code %}

Becomes (each code block is put into Spindle:

{% code %}
{% tab language="none" %}
PROCEDURE add(a,b) {

        IF (a == b) {

        RETURN

        } ELSE{}

        RETURN a + b

        }

       
{% /tab %}
{% /code %}

{% code %}
{% tab language="none" %}
DISPLAY("hi")

        add(10,40)

       
{% /tab %}
{% /code %}

{% code %}
{% tab language="none" %}
PROCEDURE sub(a,b) {

        RETURN a - b

        }

       
{% /tab %}
{% /code %}

{% code %}
{% tab language="none" %}
sub(40,10)

       
{% /tab %}
{% /code %}

##