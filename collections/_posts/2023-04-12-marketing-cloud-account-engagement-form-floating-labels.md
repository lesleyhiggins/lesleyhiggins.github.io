---
layout: post
title: "Marketing Cloud Account Engagement Form Floating Labels"
date: 2023-04-14
authors: ["Lesley Higgins"]
categories: ["Pardot", "MarTech Hacks", "Marketing Cloud Account Engagement"]
description: "Explore the integration of Bootstrap 5's floating labels feature with Marketing Cloud Account Engagement forms. This post details how to implement floating labels to improve form aesthetics and functionality, despite the native limitations of Marketing Cloud. It includes practical code examples and customization tips for making your forms more dynamic and user-friendly."
thumbnail: "/assets/images/gen/blog/blog-17-thumbnail.webp"
image: "/assets/images/gen/blog/blog-17.webp"
---

When working with custom Marketing Cloud Account Engagement forms, you may have encountered requests to use placeholder text instead of labels. Marketing Cloud Account Engagement doesn't natively support placeholders, and shifting labels to the placeholder position can be challenging.

Expanding on my [previous post about creating multi-column forms](https://www.lesleyhiggins.com/post/updated-pardot-multi-column-forms) using Bootstrap 5, this version introduces Bootstrap 5's new floating labels feature. Floating labels in Bootstrap 5 are a form design enhancement where the input field labels "float" above the field once the user starts typing, instead of staying static. This space-saving feature gives forms a modern, clean appearance. Though hidden by Bootstrap CSS, the labels remain accessible to screen-readers. This is what floating labels look like in action:

<video src="/assets/images/gen/content/file.mp4" controls></video>



Bootstrap 5 simplifies this with built-in CSS and JavaScript that manage this process—almost. We still need to add our own JavaScript to apply the appropriate classes to input fields and move checkbox and radio labels above their inputs.

The good news is that the following solution works for any fields, allowing you to use the template to build out any form once the layouts have been updated.

In MCAE, the following code will go into the Layout tab in the Layout Template that you plan to use for the form. Make sure you uncheck the “Include default CSS stylesheet” while you are there.

```html
<!DOCTYPE html>
<html>
<head>
    <base href="" >
    <meta charset="utf-8"/>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="%%description%%"/>
    <title>%%title%%</title>
    <script src="https://code.jquery.com/jquery-3.6.1.min.js"></script>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        .container {
            padding: 100px;
        }
        span.value {
            display: block;
            padding-left: 25px;
        }
        span input {
            display: block;
        }
        .form-floating > label {
            padding-left: 15px; /* Adjust this value to your preference */
        }
        .form-check {
            display: block;
            clear: both;
        }
    </style>
</head>
<body>
    <div class="container">
        %%content%%
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>

