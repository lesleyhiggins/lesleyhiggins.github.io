---
layout: post
title: "Marketing Cloud Account Engagement Form Floating Labels"
date: 2023-04-14
authors: ["Lesley Higgins"]
categories: ["marketing-cloud"]
description: "Explore the integration of Bootstrap 5's floating labels feature with Marketing Cloud Account Engagement forms. This post details how to implement floating labels to improve form aesthetics and functionality, despite the native limitations of Marketing Cloud. It includes practical code examples and customization tips for making your forms more dynamic and user-friendly."
thumbnail: "/assets/images/gen/blog/MCAE-floating-labels.png"
image: "/assets/images/gen/blog/MCAE-floating-labels.png"
---

When working with custom Marketing Cloud Account Engagement forms, you may have encountered requests to use placeholder text instead of labels. Marketing Cloud Account Engagement doesn't natively support placeholders, and shifting labels to the placeholder position can be challenging.

Expanding on my [previous post about creating multi-column forms](https://www.lesleyhiggins.com/post/updated-pardot-multi-column-forms) using Bootstrap 5, this version introduces Bootstrap 5's new floating labels feature. Floating labels in Bootstrap 5 are a form design enhancement where the input field labels "float" above the field once the user starts typing, instead of staying static. This space-saving feature gives forms a modern, clean appearance. Though hidden by Bootstrap CSS, the labels remain accessible to screen-readers. This is what floating labels look like in action:

<video src="/assets/images/gen/content/file.mp4" controls style="max-width: 500px;"></video>



Bootstrap 5 simplifies this with built-in CSS and JavaScript that manage this process — almost. We still need to add our own JavaScript to apply the appropriate classes to input fields and move checkbox and radio labels above their inputs.

The good news is that the following solution works for any fields, allowing you to use the template to build out any form once the layouts have been updated.

In MCAE, the following code will go into the Layout tab in the Layout Template that you plan to use for the form. Make sure you uncheck the “Include default CSS stylesheet” while you are there.

The style tag here includes several approaches to adjusting the appearance of the checkboxes and radio buttons. Adjust them to fit your styling needs.

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
```

Now onto the "Form" tab, where most adjustments will be made. To make the labels float, specific classes need to be added to a div that surrounds each set of label/input, as well as classes that target particular types of fields.


While it's relatively easy to add the form-floating class to the div surrounding each field, you actually have to remove it for checkboxes and radios to line up correctly.


I am using the "btn-danger" button color in this example. You can choose a [Bootstrap color](https://getbootstrap.com/docs/5.0/components/buttons/){:target="_blank"} by using classes, or you can use CSS to update it as needed.


Here is the full Form tab code:

```html
<form accept-charset="UTF-8" method="post" action="%%form-action-url%%" class="row g-3" id="pardot-form">
%%form-opening-general-content%%

%%form-if-thank-you%%
%%form-javascript-focus%%
%%form-thank-you-content%%
%%form-thank-you-code%%
%%form-end-if-thank-you%%

%%form-if-display-form%%

%%form-before-form-content%%
%%form-if-error%%
<p class="errors">Please correct the errors below:</p>
%%form-end-if-error%%

%%form-start-loop-fields%%
<div class="%%form-field-css-classes%% %%form-field-class-type%% %%form-field-class-required%% %%form-field-class-hidden%% %%form-field-class-no-label%% %%form-field-class-error%% %%form-field-dependency-css%% form-floating mb-3">
%%form-field-input%%
%%form-if-field-label%%
<label class="form-label" for="%%form-field-id%%">%%form-field-label%%</label>
%%form-end-if-field-label%%

%%form-if-field-description%%
<span class="description">%%form-field-description%%</span>
%%form-end-if-field-description%%

<div id="error_for_%%form-field-id%%" style="display:none"></div>
%%form-field-if-error%%
<p class="error no-label">%%form-field-error-message%%</p>
%%form-field-end-if-error%%
</div>
%%form-end-loop-fields%%

%%form-spam-trap-field%%

<!-- forces IE5-8 to correctly submit UTF8 content -->
<input name="_utf8" type="hidden" value="&#9731;" />

<div class="d-grid gap-2">
<button type="submit" class="btn btn-danger" accesskey="s">%%form-submit-button-text%%</button>
</div>
%%form-after-form-content%%

%%form-end-if-display-form%%

%%form-javascript-link-target-top%%
</form>
<script type="text/javascript">
  $(document).ready(function () {
    // Add classes to the various field types
    $('input[type=text], textarea').addClass('form-control').attr("placeholder", "label");
    $('textarea').attr("style", "height: 100px");
    $('select').addClass('form-select');
    $('input[type=checkbox],input[type=radio]').addClass('form-check-input');
    $('.pd-checkbox,.pd-radio').addClass('form-check');
    
    $("div").each(function () {
      // Check if the div contains a label and at least one checkbox or radio button
      if (
        $(this).find("label").length &&
        ($(this).find("input[type='checkbox']").length || $(this).find("input[type='radio']").length)
      ) {
        // Remove the extra classes from the div
        $(this).removeClass("form-floating mb-3");

        // Move the label above the span with the class 'value'
        $(this).find(".form-label").each(function () {
          $(this).insertBefore($(this).siblings(".value"));
        });
      }
    });
    
    $('span').each(function () {
      if ($(this).children('input[type="checkbox"], input[type="radio"]').length > 0) {
        $(this).addClass('form-check');
      }
    });
  });
</script>
```
Now, onto the actual form. This is also a multi-column form, which means that you can have two, three, or up to 12 fields across. Bootstrap operates on a 12-grid system, so just divide everything by 12. They also don’t have to be equal. For example, you could put "Address", "State", and "Zip" in one line and have the fields be 6-2-4. I recommend that if you are going to do this, you set an "md" breakpoint so that on mobile, every field gets its own row. 

This is what that looks like in the "Form Field Advanced" tab:

<img src="/assets/images/gen/blog/MCAE settings.webp" alt="MCAE Settings">

That’s it. That is all you need to do to create multi-column forms with floating labels.


It's important to note that while floating labels can be visually appealing and save space, they can also have negative implications for accessibility. Users who rely on screen readers may have difficulty understanding the context of an input field when the label moves out of view. This can be especially problematic if the label contains important information or instructions for filling out the form. Additionally, there are potential issues with color contrast and font size that can make the labels difficult to read for some users, even when they are in view.


To mitigate these issues, it's important to carefully consider the use of floating labels in your forms. If you do choose to use them, make sure that the labels are still accessible to screen readers and that they are clearly visible to all users. This may require adjusting the color, font size, or placement of the labels to ensure that they are legible and easy to understand. You may also want to consider providing additional instructions or context for users who may have difficulty with the floating labels.


Ultimately, the decision to use floating labels should be based on a careful consideration of the trade-offs between aesthetics and accessibility. While they can be a useful design tool in some cases, they should not be used at the expense of making your forms inaccessible to some users. By keeping accessibility in mind throughout the design process, you can create forms that are both visually appealing and easy to use for all users.
