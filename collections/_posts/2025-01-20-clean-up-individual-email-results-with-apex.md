---
layout: post
title: "Clean up Individual Email Results with Apex"
date: 2025-01-20
categories: ["marketing-cloud"]
description: "Apex code and strategies to bulk delete or deduplicate IndividualEmailResult records safely in Salesforce."
thumbnail: /assets/images/gen/blog/clean-up-individual-email-results-with-apex-hero.png
---


<p align="center">
  <img src="/assets/images/gen/blog/clean-up-individual-email-results-with-apex-hero.png" alt="Clean up Individual Email Results with Apex illustration" width="800">
</p>


Marketing Cloud Connect (MCC) is not going anywhere anytime soon, despite Salesforce’s push towards Data Cloud. The primary reason is that the refresh schedule with Data Cloud just cannot keep up with MCC at this time.

So, it’s time to tackle the biggest issue with MCC: Individual Email Results (IERs). 


IERs are an optional feature within the Marketing Cloud Connected App in Salesforce that most users turn on. It allows for the pass-back of email sends and link data, connected to a Contact or Lead in Salesforce. They can be very useful as a related list on the Contact or Lead. They can also be useful for reporting. They also take up a tremendous amount of storage. 


If storage is becoming a problem, you need to find the line in which you want to purge old IERs based on your own business and reporting needs, as well as the limitations of your storage space. For this example we will be purging records older than 180 days.


IERs are only created if you send emails through Journey Builder or through Salesforce Sends, which are a method of a guided send that requires the use of Data Extensions in the Salesforce Data Extensions folder, or sending directly to a Salesforce Campaign or Report. 


When it comes to storage, the first culprit will be Individual Email Results themselves. After that, Individual Link Level Details will likely take up the most amount of storage. I stopped my record purging after that because nothing else was taking up enough storage space to risk losing data. For example, the next object you’ll likely see in the order of storage space will be Triggered Sends. These are created for any Journey Builder send and are potentially being referenced by newly created IERs. Ultimately, removing the one-off Journey Builder runs that are no longer being used was too much effort for the amount of storage it was going to reduce. 


Depending on the volume of your email sends, it’s highly likely that a Flow will not be able to handle this record delete. I tried, but if you are dealing with more than 10,000 records on a daily basis, it’s just too complicated for Flows. In other words, if you ever have a single day where you send more than 10,000 records, then a Flow is not for you. 


This means you are going to have to leverage Apex for a bulk delete. 


These Apex classes are very simple for a seasoned Salesforce developer, but a new concept for Marketing Cloud specialists. If you have never done anything with Apex before, this is a good place to start. If you have no interest in Apex, passing this code over to your Salesforce developer will make their lives a little easier.


Steps:

1. Access a Partial or Dev Sandbox

It’s preferable to use a Partial sandbox, which comes with some data from your production org. If you can do this, that is what I would recommend because you are going to want some existing Send Definitions for testing. Creating a Send Definition requires a Marketing Cloud Business Unit record in Salesforce, which requires a MCC connection.


There are workarounds for a dev Sandbox, though. Personally, I used a dev Sandbox, but I was able to connect it to a Marketing Cloud business unit in a multi-business unit org where I sent myself an email, which in turn created a Send Definition for me, which I later used for my test class. IERs are not created immediately, though, so I had to return to this project a day after doing the send. 


To Access a Sandbox:

Log in to your Salesforce org.

Navigate to Setup → Sandboxes.

Select and log into a Partial Copy Sandbox (or create a new one).


Hot tip for new to sandbox Marketing Cloud peeps: your sandbox password is the same as your production password at the time the sandbox was created.


2. Build Batch Apex for IER Cleanup

Steps:

Open up the Developer Console. It’s located in the Setup Menu.

Create a new Apex Class. 

Paste in the following code. This will retrieve IERs created prior to 180 days ago. The code we use to call this class will specify that it deletes in batches of 200.



Apex Class: IERCleanupBatch

{% gist lesleyhiggins/75da689f72a4941b7bf154c78ed49228 %}


3. Build Batch Apex for Individual Link Level Details Cleanup

Steps:

Staying within the Developer Console, create a new Apex Class. 

Paste in the following code. This will retrieve IERs created prior to 180 days ago. The code we use to call this class will specify that it deletes in batches of 200.



Apex Class: IndividualLinkCleanupBatch

{% gist lesleyhiggins/c6da3971b921f3355b693cb7fc565ee0 %}


4. Evaluate Storage Usage in Production

Go to Setup → Storage Usage.

Identify any other large objects consuming storage.

Examples:

Email Sends, Aggregate Link Level Details, or Triggered Sends.

Avoid over-cleaning-up critical objects (e.g., Triggered Sends, which are essential for Journeys).

If any other IER related objects are taking up too much space, repeat the steps above for those objects.


5. Create a Batch Scheduler
Steps:

Staying within the Developer Console, create a new Apex Class. 

Paste in the following code. This will execute the Apex classes you created above in batches of 200.



Apex Class: MorningCleanupScheduler

{% gist lesleyhiggins/c4278b7339167561020d300935ef1ad6 %}


6. Create a Test Class

You cannot deploy Apex from a sandbox to production without a test class. A test class will travel to production with your executable Apex and, as the name suggests, test it. Essentially, it creates test records and then deletes test records for each of the processes. Getting test classes to work actually took me more time than the original Apex because of required fields and dependencies that come with creating these records. This is why I recommend using a partial sandbox, because you are going to need a send definition (or triggered send definition) in order to create a test IER. 


Steps:

Staying within the Developer Console, create a new Apex Class. 

Paste in the following code. This will execute the Amex classes you created above in batches of 200.

There are some quirks with the required field of et4ae5__TriggeredSendDefinition__c OR et4ae5__SendDefinition__c. Salesforce requires one of these fields to be populated when creating an IER and you have to make sure you populate the correct field with the correct type. 

Technically you should write a lookup to the Send Definition object to retrieve an id for the test so that this test can be re-run in perpetuity. However, I was not able to get a lookup to work and I ended up just hardcoding in a SendDefinition Id for the purpose of getting the test to work. Since I also advised you not to delete Send Definitions, this will probably be fine. To retrieve a hard coded Id, open an existing IER in production, click on the Send Definition lookup field and copy the id of that record from the URL.



Test Class: MorningCleanupSchedulerTest

{% gist lesleyhiggins/a03972525dde2a8a04388316d85a6d60 %}


7. Push to Production

Create a Change Set in your sandbox.

Add the following to the change set:

IERCleanupBatch

IndividualLinkCleanupBatch

MorningCleanupScheduler

MorningCleanupSchedulerTest

Upload the Change Set to production.


8. Validate and Deploy

In production, go to Inbound Change Sets in Setup and find the change set you just pushed up. 

Click validate.

Assuming the test is validated, choose Quick Deploy. Hopefully you do not need to fix anything, but if you do, the error log will give you pretty good instructions.


9. Schedule the Job

Now in Production, open up the Developer Console. 

Choose Debug → Open Execute Anonymous Window. 

Paste in the following code and adjust the time if needed.

Click Execute.

```
String cronExpression = '0 0 6  ?'; // Every day at 6:00 AM
System.schedule('Morning Cleanup Jobs', cronExpression, new MorningCleanupScheduler());
```

Summary

By following these steps, you will:

Remove old and unused et4ae5__IndividualEmailResult__c and et4ae5__IndividualLink__c records.

Keep storage usage in check.

Automate daily cleanup using a robust, scheduled Apex solution.