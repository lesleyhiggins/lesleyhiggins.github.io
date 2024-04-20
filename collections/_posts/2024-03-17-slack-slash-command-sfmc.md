---
layout: post
title: "Streamline Your Workflow: Slack Slash Command for Salesforce Marketing Cloud Data Extension Lookup"
date: 2024-03-17T09:49:03Z
authors: ["Lesley Higgins"]
categories: ["Slack", "MarTech Hacks", "Marketing Cloud"]
description: Dive into the simplicity of integrating a Slack Slash Command with Salesforce Marketing Cloud for quick Data Extension lookups.
thumbnail: "/assets/images/gen/blog/blog-18-thumbnail.webp"
image: "/assets/images/gen/blog/blog-18.webp"
comments: true
---

## Introduction

I know I am not the only one who has, despite best efforts to maintain a logical folder structure, not been able to find a Data Extension (DE). In fact, one of the most common frustrations in Marketing Cloud is the inability to search for a DE using out-of-the-box search functionality, which only searches the parent level, not child folders.

There are a few solutions to this, none of which are “just leave every DE in the parent folder so it’s easier to search.”

### Solutions to Enhance DE Lookup

- **Chrome Extension by DESelect**: This tool has worked for me on and off throughout the years. 
- **Custom Search Form on a CloudPage**: Alternatively, you can craft your own search form on a CloudPage.

### Slack Integration

But, what if you're practically living in Slack? Integrating a solution into your Slack workspace could significantly streamline your workflow. Enter the Slack Slash Command, a nifty tool for creating custom interactive commands. Here’s how to set it up:

#### Step 1: Create a Slack App

Begin by creating a Slack App. This app will house your custom slash command.

#### Step 2: Craft Your Slash Command

Navigate to your App Management dashboard, select your app, and find Slash Commands under Features. Click `Create New Command` and fill in the details:

- **Command**: Choose a simple yet unique name to avoid clashes with existing commands. For instance, `/delookup`.
- **Request URL**: Your command's endpoint that Slack will hit when the command is invoked. It should be HTTPS. This will be a Code Resource in Marketing Cloud.
- **Short Description**: Summarize what your command does.
- **Usage Hint**: Offer a hint or example usage to guide users.

#### The Importance of Naming

Slash commands aren't namespaced, meaning commands can overlap. If conflicts arise, Slack defaults to the most recently installed command. Pick a name that's both memorable and unique to your needs.

#### Step 3: Prepare for Incoming Commands

When someone uses your slash command, Slack sends a detailed HTTP POST request to your specified URL. For a command like `/delookup pets`, expect a payload similar to:

```text
token=gIkuvaNzQIHg97ATvDxqgjtO
&team_id=T0001
&team_domain=example
&channel_id=C2147483705
&channel_name=lesley-slack-dev
&user_id=U2147483697
&user_name=lhiggins
&command=%2Fdelookup
&text=pets
&response_url=https%3A%2F%2Fhooks.slack.com%2Fcommands%2FTDJ8J3TDM…
&trigger_id=13345224609.738474920.8088930838d88f008e0
&api_app_id=A123456
```


## History

John Gruber created the [Markdown](#) language in 2004 in collaboration with Aaron Swartz on the syntax, with the goal of enabling people "to write using an easy-to-read and easy-to-write plain text format". Its key design goal is readability. That the language be readable as-is.

> "Markdown is a lightweight markup language with plain-text-formatting syntax"

To this end, its main inspiration is the existing conventions for marking up plain text in email, though it also draws from earlier markup languages, notably setext, Textile, and reStructuredText.

## Markdown Flavours

From 2012, a group of people including Jeff Atwood and John MacFarlane launched what Atwood characterized as a standardization effort. A community website now aims to "document various tools and resources available to document authors and developers, as well as implementors of the various markdown implementations".

{% include framework/shortcodes/figure.html src="/assets/images/gen/content/content-1.webp" title="There are many popular text editors for Markdown" caption="VSCode Editor" alt="Photo of designing a website in Figma" link="https://figma.com" target="_blank" %}

### GitHub Flavored Markdown (GFM)

In 2017, GitHub released a formal specification of their GitHub Flavored Markdown (GFM) that is based on CommonMark. It follows the CommonMark specification exactly except for tables, strikethrough, autolinks and task lists, which the GitHub spec has added as extensions. GitHub also changed the parser used on their sites accordingly, which required that some documents be changed. For instance, GFM now requires that the hash symbol that creates a heading be separated from the heading text by a space character.he user to create their own.

{% include framework/shortcodes/figure.html src="/assets/images/gen/content/content-2.webp" title="There are many popular text editors for Markdown" caption="VSCode Editor" alt="Photo of designing a website in Figma" link="https://figma.com" target="_blank" %}

### Markdown Extra

Markdown Extra is a lightweight markup language based on Markdown implemented in PHP (originally), [Python](#) and [Ruby](#). It adds features not available with plain Markdown syntax. Markdown Extra is supported in some content management systems such as, for example, Drupal.

### MDX

At the same time, a number of ambiguities in the informal specification had attracted attention.These issues spurred the creation of tools such as Babelmark to compare the output of various implementations, and an effort by some developers of Markdown parsers for standardisation. However, Gruber has argued that complete standardization would be a mistake:

```js
$(window).scroll(function () {
  // this will work when your window scrolled.
  var scroll = $(window).scrollTop();
  if (scroll > 100) {
    $(".header").addClass("header-scrolled");
  } else {
    $(".header").removeClass("header-scrolled");
  }
});
```

Gruber avoided using curly braces in Markdown to unofficially reserve them for implementation-specific extensions. Markdown Extra adds the following features to Markdown:

- markdown markup inside HTML blocks
- elements with id/class attribute
- fenced code blocks that span multiple lines of code
- tables
- definition lists
- footnotes
- abbreviations
