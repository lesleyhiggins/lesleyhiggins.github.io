---
layout: post
title: "Agent Builder to Marketing Cloud Engagement"
date: 2025-10-18
authors: ["Lesley Higgins"]
categories: ["Marketing Cloud", "Aalesforce", "AI"]
description: "A recap of my 2025 TDX Hackathon project: using Salesforce's Agent Builder to automate Marketing Cloud email creation through Apex, AI, and Slack integration."
thumbnail: /assets/images/gen/blog/2025-hackathon-project.png
---

<p align="center">
  <img src="/assets/images/gen/blog/2025-hackathon-project.png" alt="2025 Hackathon Project illustration showing camp counselor, AI, and Marketing Cloud email creation" width="800">
</p>

Earlier this year, I participated in the Hackathon at Salesforce’s TDX conference. I have a love/hate relationship with hackathons. They can either be an amazing opportunity to learn, but they are also exhausting, as you are essentially working non-stop for a couple of days. 

The theme of this particular hackathon was centered around the newly released **Agent Builder**, a tool designed to let developers create intelligent agents that combine AI with your own company's data and automation.

I decided to work solo on this challenge because I wanted to force myself to learn the ins and outs of Agent Builder from scratch. Over the course of just two days, I built an integrated prototype that connects **Agentforce**, **Apex**, **Marketing Cloud Engagement**, and **Slack** into a single workflow.

You can watch a quick demo of the final prototype below before diving into the build details.

<div class="video-embed">
<iframe width="800" height="450"
  src="https://www.youtube.com/embed/En-I8jNU8zU"
  title="Agent Builder to Marketing Cloud Engagement Demo"
  frameborder="0"
  allowfullscreen></iframe>
</div>

Here’s what I put together:

1. **An Apex Handler for Marketing Cloud Emails**  
   I started by writing an Apex handler that makes an API call to **Marketing Cloud Engagement** to automatically create an HTML email. This allowed the agent to go beyond simple responses and actually generate content that could be sent to customers.

2. **Custom Objects for Contextual Data**  
   To make the use case realistic, I built a small Salesforce data model to support a fictional youth summer camp organization. I created custom objects for Camps, Camp Sessions, and Camp Counselors, so that the AI could pull relevant context like program details, age groups, campers, their parents, and locations.

3. **An Agentforce Template for AI-Generated Emails**  
   I then built an **Agentforce Template** that would use AI to draft email content based on business-specific data from those custom objects. This is where the “intelligence” comes in by giving the agent awareness of company-specific tone, offerings, and seasonal messaging.

4. **A Working Agent That Connects It All**  
   Using Agent Builder, I created an agent that listens for email requests, interprets them, and calls the Apex handler that performs the Marketing Cloud API call. The result: an AI that can generate a complete, ready-to-send HTML email in seconds.

5. **Streamlined Marketing Cloud Integration**  
   To simplify things (and save time), I leaned on the **default header and footer** in Marketing Cloud. That meant the API call only needed to build the email body. After all, I only had two days to put this all together.

6. **Slack Integration for Real-Time Use**  
   Finally, I connected everything to Slack, so a camp counselor on the go could request an email from within a channel or direct message. The counselor could type something like:  
   _“Draft an email letting parents know that a camper has taken a vow of silence”_  
   …and the Agent would generate a well-written, context-aware email using the camp data from Salesforce and then push the content to Marketing Cloud so a brand-appropriate email could be sent.

---

⚡ **Key Takeaways**  
This was one of the most technically rewarding projects I’ve worked on in a short timeframe. It demonstrated just how quickly Salesforce’s new **Agentforce** tools can bridge AI-driven experiences with existing marketing automation workflows. The entire solution — from Apex to Slack — came together in under 48 hours.

However, Agent Builder in Salesforce is not totally intuitive and there are a lot of moving parts. On the upside, though, it's very easy to view the output and diagnose any errors with the prompts and actions.

🎥 **Watch the video above** to see a condensed demo of the build in action. This is the video I actually submitted. You’ll see how the agent interprets requests, uses business context, and sends API calls to build an email directly inside Marketing Cloud Engagement.