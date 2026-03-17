---
layout: post
title: "Fixing Data Cloud Related List Enrichment Permissions: Solving the 'DataSpace' Error"
date: 2026-03-17
authors: ["Lesley Higgins"]
categories: ["Salesforce", "Data 360"]
description: "Step-by-step guide to resolve the 'sObject type DataSpace is not supported' error when setting up Related List Enrichment permissions in Salesforce Data Cloud. Learn the missing permission set configuration that Salesforce's documentation doesn't mention."
thumbnail: /assets/images/gen/blog/fixed-errors.png
---

# Do you have users that are not able to see Related List Enrichments in Sales or Service Cloud?

I just spent the last month of my life (in small doses) trying to expose Related List Enrichments to Service Reps without giving them the Data Cloud Architect permission set. No matter how many different ways I tried to follow Salesforce's own help article (<a href="https://help.salesforce.com/s/articleView?id=data.c360_a_create_a_related_list_permset.htm&type=5" target="_blank" rel="noopener noreferrer">Create a Permission Set for a Related List Enrichment</a>), I continued to see this error: 

> INVALID_TYPE: SELECT Id FROM DataSpace WHERE DataSpaceDefinition.DeveloperName ^ ERROR at Row:1:Column:16 sObject type 'DataSpace' is not supported.


Here's what's actually going on and how to solve it.

## The Setup

Data Cloud Related List Enrichments are a great way to surface Data Cloud insights directly on CRM record pages - things like transaction history, engagement scores, calculated insights, and more. Your admins and architects can probably see them just fine because they have the **Data Cloud Architect** permission set, which grants broad access.

But when you try to extend that visibility to regular users — say, a service rep who just needs to *see* the enrichment data — you hit a wall.

Salesforce's own help article (<a href="https://help.salesforce.com/s/articleView?id=data.c360_a_create_a_related_list_permset.htm&type=5" target="_blank" rel="noopener noreferrer">Create a Permission Set for a Related List Enrichment</a>) walks you through creating a custom permission set and configuring Data Cloud Data Space Management. That's all correct and necessary. But the article leaves out a critical step, which is what causes the `DataSpace` SOQL error.

## Why the Error Happens

When a user views a Related List Enrichment on a record page, the platform runs a SOQL query against the `DataSpace` object behind the scenes to resolve which data space the enrichment pulls from. The standard **Data Cloud User** permission set only grants **Read** access to the `DataSpace`, `DataSpaceDefinition`, and `DataSpaceMember` objects. That's not enough.

The query needs **View All Records** on those objects to execute successfully. The **Data Cloud Architect** permission set has this, which is why architects and admins don't encounter the error. But you obviously don't want to hand out Architect-level access to every service rep.

## The Fix

You need three things assigned to the user:

### 1. Data Cloud Permission Set License

Go to the user's record and add the **Data Cloud** permission set license. Not "Customer Data Cloud for Marketing" and not "Remote Data Cloud" — just **Data Cloud**.

### 2. Data Cloud User Permission Set

Assign the standard **Data Cloud User** permission set. This gives the baseline Data Cloud access but isn't enough on its own for Related List Enrichments.

### 3. A Custom Permission Set (This Is the Key)

Create a new permission set. Then configure two things on it:

**Data Space Management:**

- Go to **Apps → Data Cloud Data Space Management**
- Under **Data Spaces**, click **Edit** and enable the data space your enrichment uses (likely "default")
- Click into the data space, and under **User Data Access Level for This Data Space**, check both boxes:
  - Allow users to query only CRM-related data that they have access to based on the related parent object (you can not click the second without this one being checked).
  - Allow users to query all objects in the data space. When deselected, users can query only an object that they have access to based on governance policies (if you want).

**Object Permissions (the missing piece):**

Go back to **Permission Set Overview** and find **Object Settings** and change the following objects from **Read** to **Read, View All Records**:

- **Data Spaces** (`DataSpace`)
- **Data Space Definitions** (`DataSpaceDefinition`)
- **Data Space Members** (`DataSpaceMember`)

This is the step Salesforce's documentation doesn't mention, and it's the one that actually fixes the error. There is a good chance that just one of these fixes the error, but I chose all three to match the Data Cloud Architect permission set.

### 4. Assign Both Permission Sets

Make sure the user has both **Data Cloud User** and your custom permission set. Have them log out and back in, and the Related List Enrichments should now be visible.

## Quick Troubleshooting Guide

| Symptom | Likely Cause |
|---|---|
| `sObject type 'DataSpace' is not supported` error | Missing **View All Records** on DataSpace, DataSpaceDefinition, and DataSpaceMember objects |
| "Looks like you don't have access to the data space" | Missing **Data Cloud User** permission set or data space not enabled in Data Cloud Data Space Management |
| Related List just doesn't appear at all | Check that the Related List Enrichment is added to the page layout and the user's profile has access to the parent object |
| Works with Data Cloud Architect but not Data Cloud User | The gap is almost always the View All Records permissions on the three DataSpace objects described above |

## Wrapping Up

Salesforce's documentation gets you about 90% of the way there. The missing 10% — granting View All Records on the DataSpace-related objects — is what turns a frustrating SOQL error into a working enrichment for your end users.