---
layout: post
title: "Build Dynamic Form Checkboxes with JavaScript + SSJS"
date: 2022-04-13
authors: ["Lesley Higgins"]
categories: ["Marketing Cloud"]
description: "Learn how to create dynamic form checkboxes that pull data from Salesforce Marketing Cloud Data Extensions using Code Resources, SSJS, and JavaScript. Perfect for external websites that need real-time product data."
thumbnail: "/assets/images/gen/blog/code-resource-to-form.jpg"
---

I previously wrote about how to build a Customer Service form on a CloudPage, which included a lookup to current active products so that customers could select their product from a dropdown menu. Being able to build product selection dynamically leads to **better case creation and makes support teams very happy**. I am still proud of this form, but it relied heavily on AMPScript and therefore had to be iFramed onto their website, which is not always an option web developers are pleased to implement.

## The Challenge

The challenge I faced this time around, however, was to build an **HTML order form that could be placed on a website directly**, but was still able to reference internal Salesforce data to build product checkboxes. The ability to use AMPScript or Server Side JavaScript (SSJS) on the form itself was eliminated by the requirement that the HTML be rendered on a website outside of Marketing Cloud, as those programming languages can only be processed inside of Marketing Cloud.

However, I was able to leverage a **Code Resource** to accomplish this feature.

## Code Resources: The Solution

Code Resources are powerful, utilitarian tools that can be leveraged for all sorts of things in Marketing Cloud. I recommend taking a look at the <a href="https://mateuszdabrowski.pl/docs/config/sfmc-code-resources/#fun-things-you-can-do-with-code-resource" target="_blank">"Fun Things" you can do with Code Resources</a>, courtesy of Mateusz Dąbrowski.

By using a Code Resource, we can **leverage SSJS and/or AMPScript and return JavaScript** that the website can read.

<p align="center">
  <img src="/assets/images/gen/blog/code-resource-to-form.jpg" alt="Code Resource data flow from Marketing Cloud to external website" width="400">
</p>

## HTML Form Requirements

### 1. Create the Container

First, we need to create a container in the form where the checkboxes will be added:

```html
<div id="container"></div>
```

### 2. Reference the Code Resource

Second, in order to call a script from the HTML form, we just need one line of code that references the Code Resource's URL:

```html
<script type="text/javascript" src="https://cloud.<yourpage>.js"></script>
```

This will effectively **treat the response from your Code Resource as if it's part of the page** that the form is on.

## Code Resource Requirements

### Initial Setup

The Code Resource type we are leveraging here is, not surprisingly, a **JavaScript resource**. When you create a new JavaScript resource, the page will be pre-populated with the below code. Don't delete this, your code will have to live in between the brackets.

```javascript
(function(/* root, doc */){

}(window, document));
```

### Adding SSJS for Data Retrieval

The first thing we are going to do is leverage **SSJS to pull data in from data extensions**. You could also leverage AMPScript to pull data directly from Salesforce, but for most forms the **15 minute refresh rate of Synchronized Data Extensions** gives accurate enough information. Retrieving data from Data Extensions is faster than doing lookups to Salesforce and should be used whenever possible.

If we were just writing JavaScript in this Code Resource there would be no reason to also use a `<script>` tag. However, since we are leveraging SSJS, we need to let the page know what we are up to. This should be placed within the existing pre-populated brackets:

```html
<script runat="server">
Platform.Load("core","1.1.5");

</script>
```

### Initialize Data Extension and Create Filter

Within the script tags, the first thing we are going to do is **initialize a Data Extension and create a lookup filter**:

```javascript
var de = DataExtension.Init("Data-extension-external-key");

var filter = { Property: "Pricebook2Id", SimpleOperator: "equals", Value: "18DIGSFID" };
```

### Retrieve and Process Results

We can then retrieve the results of the lookup filter on the initialized data extension:

```javascript
var results = de.Rows.Retrieve(filter);
```

The results are returned as an **Array in JSON format**, like this:

```json
[
    {
        "Pricebook2Id":"XXXXXXXXXXXXXXXXX",
        "Web_Name":"Gloss",
        "ProductCode":"XXXXX",
        "Family":"Symphony"
    },
    {
        "Pricebook2Id":"XXXXXXXXXXXXXXXXX",
        "Web_Name":"Semi-Gloss",
        "ProductCode":"XXXXX",
        "Family":"Cascade"
    },
    {
        "Pricebook2Id":"XXXXXXXXXXXXXXXXX",
        "Web_Name":"Matte",
        "ProductCode":"XXXXX",
        "Family":"Cascade"
    },
    {
        "Pricebook2Id":"XXXXXXXXXXXXXXXXX",
        "Web_Name":"Satin",
        "ProductCode":"XXXXX",
        "Family":"Symphony"
    }
]
```

### Stringify the Array

However, the array has to be **stringified to actually read the Array**, otherwise all you get back is this:

```
System.Collections.ArrayList
```

To do that we make a new temporary variable with the stringified version of the array:

```javascript
var resultsString = Stringify(results);
```

### Pass Data from SSJS to JavaScript

Once we have the array in readable format we need to move to the JavaScript, which is ultimately what the HTML page will process. However, **JavaScript code cannot immediately grab the variables** that you have created in the SSJS script. In order to pass a variable from SSJS to JavaScript, we can create AMPScript variables from SSJS:

```javascript
Variable.SetValue("@results", resultsString);
```

Now the JSON formatted version of the results is ready to be used by JavaScript. This is where we close out the SSJS script and move onto writing JavaScript that will build the checkboxes.

## Building the Dynamic Checkboxes

### Convert AMPScript Variable to JavaScript

The first thing we have to do is take that AMPScript variable and turn it right back into a JavaScript variable:

```javascript
var results = %%=v(@results)=%%;
```

### Create Checkboxes Dynamically

Now we are going to **loop through the array and build checkboxes**, appending them one-by-one to the container we created on the HTML form:

```javascript
for (var i = 0; i < results.length; i++) {

    var checkbox = document.createElement('input');
    checkbox.type = 'checkbox';
    checkbox.id = results[i].Web_Name;
    checkbox.name = 'Sample';
    checkbox.value = results[i].ProductCode;

    var label = document.createElement('label')
    label.htmlFor = results[i].Web_Name;
    label.appendChild(document.createTextNode(results[i].Web_Name));

    var br = document.createElement('br');

    var Container = document.getElementById('container');
    Container.appendChild(checkbox);
    Container.appendChild(label);
    Container.appendChild(br);

}
```

## Complete Code Resource

All put together, the **full Code Resource** will look something like this:

```javascript
(function (/* root, doc */) {

<script runat="server">
Platform.Load("core","1.1.5");

var filter = { Property: "Pricebook2Id", SimpleOperator: "equals", Value: "18DIGSFID" };

var de = DataExtension.Init("Data-extension-external-key");

var results = de.Rows.Retrieve(filter);

var resultsString = Stringify(results);

Variable.SetValue("@results", resultsString);

</script>

var results = %%=v(@results)=%%;

for (var i = 0; i < results.length; i++) {

    var checkbox = document.createElement('input');
    checkbox.type = 'checkbox';
    checkbox.id = results[i].Web_Name;
    checkbox.name = 'Sample';
    checkbox.value = results[i].ProductCode;

    var label = document.createElement('label')
    label.htmlFor = results[i].Web_Name;
    label.appendChild(document.createTextNode(results[i].Web_Name));

    var br = document.createElement('br');

    var Container = document.getElementById('container');
    Container.appendChild(checkbox);
    Container.appendChild(label);
    Container.appendChild(br);

}  

}(window, document));
```

## Key Benefits of This Approach

✅ **External Website Integration**: Forms can be placed directly on external websites  
✅ **Real-Time Data**: Pulls current data from Marketing Cloud Data Extensions  
✅ **No iFrame Required**: Eliminates the need for embedded CloudPages  
✅ **Developer-Friendly**: Standard HTML/JavaScript that web developers can work with  
✅ **Dynamic Content**: Product lists stay current without manual updates  

## Use Cases

This technique is perfect for:

- **E-commerce product selection forms**
- **Service request forms with dynamic product lists**
- **Lead capture forms with current offerings**
- **Event registration with real-time availability**
- **Any form requiring up-to-date Salesforce data**

## Technical Considerations

⚠️ **Performance**: Data Extension lookups are faster than direct Salesforce queries  
⚠️ **Refresh Rate**: Synchronized Data Extensions refresh every 15 minutes  
⚠️ **Error Handling**: Consider adding try/catch blocks for production use  
⚠️ **Security**: Validate and sanitize all dynamic content  

This approach bridges the gap between Marketing Cloud's data capabilities and external website requirements, giving you the best of both worlds: dynamic, data-driven forms that can live anywhere on the web.