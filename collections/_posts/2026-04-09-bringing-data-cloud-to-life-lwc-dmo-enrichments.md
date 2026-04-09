---
layout: post-2
title: "Bringing Data 360 to Life: Custom LWCs on DMO Record Pages and Bridging to Salesforce CRM"
date: 2026-04-09
authors: ["Lesley Higgins"]
categories: ["Data 360", "Salesforce"]
description: Make Data 360 actually useful where your teams work. Learn to build custom Lightning Web Components for DMO record pages and surface DMO data in Sales or Service Cloud with Related List Enrichments — without breaking your consumption credit budget.
thumbnail: "/assets/images/gen/blog/pet-profile-dmo.png"
---

# Bringing Data 360 to Life on the Core side: Custom LWCs on DMO Record Pages and Bridging to Sales and Service Cloud

---

Data 360 is powerful. It ingests, unifies, and harmonizes data from every corner of your tech stack. However, your service reps and sales teams will likely never open a DMO record page. They live in Service Cloud or Sales Cloud (or Revenue Cloud, or Non-Profit Cloud, or Financial Services Cloud, etc. etc, etc). They don't know what a Data Model Object is, and they shouldn't have to.

So once you build your data model, the challenge becomes making the data *useful* beyond building segments and sending emails.

This post covers two strategies for closing that gap:

1. **Enhancing DMO record pages with custom Lightning Web Components**
2. **Using Related List Enrichments to surface DMO data on CRM record pages**

And then we'll dive into the boring side of things: **consumption credits**, and how both of these approaches affect your bill.

---

## Part 1: Custom LWCs on DMO Record Pages

### The Problem With Vanilla DMO Pages

Out of the box, DMO record pages support several standard Lightning components including Action Launcher, Dynamic Related Lists, Tableau dashboards, CRM Analytics components, and workflow tools like Orchestration Work Guide and Decision Explainer Log History. However, you can't convert the main record detail to a dynamic form, and the standard components don't always solve the core problem: making your data truly actionable.

For example, if your DMO has a field like `Image_URL__c` storing a link to a pet photo, product image, or headshot, the standard record detail only shows the raw URL string — not the actual image. This is a solvable problem with a simple Lightning Web Component that renders the URL as an HTML image.

Displaying an image is the need I encountered in my most recent build, but you will likely come up with many use cases as you build out Data 360. 

### Building an Image Display Component

The approach is straightforward: create a Lightning Web Component that reads a URL field from the current record and renders it as an actual image. The component uses `getRecord` from `lightning/uiRecordApi` — no Apex required.

Here's the key consideration for DMO objects: **you can't use standard `@salesforce/schema` imports** for DMO field references. DMOs aren't SObjects in the traditional sense, so schema imports can be unreliable. Instead, reference your fields as strings:

```javascript
const IMAGE_FIELD = 'Pet_Profile__dlm.Image_URL__c';
```

This works because the Lightning UI API *does* support DMO record pages. When you're on a DMO record page, `recordId` gets passed to the component through the `@api` decorator just like any other record page.

The metadata XML is where you lock the component to the correct object:

```xml
<targetConfig targets="lightning__RecordPage">
    <objects>
        <object>Pet_Profile__dlm</object>
    </objects>
</targetConfig>
```

Once deployed, you drag it onto the record page like any other component. However, unlike regular sObjects, you can't access DMO record page layouts through Setup → Object Manager. Instead, navigate to a DMO record in Data Explorer, open it, and select **Edit Page** from the gear menu.

### What Else Can You Build?

Once you've deployed one LWC to a DMO page, you start seeing opportunities everywhere:

- **Calculated insight visualizations** — pull aggregated metrics from related CIOs and render them as progress bars or charts
- **Engagement timeline components** — query related engagement DMOs and display them as a chronological feed
- **Data quality indicators** — highlight records with missing or stale data using conditional formatting
- **Quick-action buttons** — trigger Data 360-specific actions (like a segment refresh or an activation) from the record page itself

The DMO record page is a canvas. The standard layout just doesn't show it.

---

## Part 2: Bridging to Service Cloud and Sales Cloud With Related List Enrichments

### Why Your Reps Won't Visit Data 360

Data 360 is a back-end tool. It's where architects and data engineers build the plumbing. Your service agents work in the console. Your sales reps work in opportunity views. No service rep is going to natively navigate to the DMO page that I just added the LWC to.

The bridge between Data 360 and the CRM apps is <a href="https://trailhead.salesforce.com/content/learn/modules/data-cloud-enrichments" target="_blank">**Data 360 Enrichments**</a>, and specifically, **Related List Enrichments**.

**Note**: If your sales or service reps have problems viewing Data 360 Enrichments even after following the instructions in the linked Trailhead, there might be additional undocumented permissions you need to configure. I cover those <a href="https://www.devlesley.com/blog/2026-03-17-fixing-data-cloud-related-list-enrichment-permissions/" target="_blank">here</a>.

### How Related List Enrichments Work

A Related List Enrichment creates a lookup relationship from a DMO to a CRM object (like Contact, Lead, or Account). Once that relationship exists, you can add a Related List of DMO records directly onto a Contact or Account record page — right there in Service Cloud or Sales Cloud, alongside Cases, Opportunities, and Activities.

The setup follows a dependency chain:

1. **Profile data ingestion** — Contacts/Leads must be ingested into Data 360 and mapped to the Individual DMO (or Accounts into the Account DMO)
2. **Identity Resolution** — Unified Individual (or Unified Account) profiles must be active. The enrichment queries through the unified profile to resolve which DMO records belong to which CRM record.
3. **Engagement/custom data ingestion** — The DMO you want to surface (like Pet Profile, Email Engagement, or Transaction History) must be ingested and mapped
4. **DMO relationships** — The target DMO needs an N:1 or 1:1 relationship to the Individual or Account DMO
5. **Create the enrichment** — In Object Manager, navigate to the CRM object → Data 360 Related List → New → follow the wizard

Once activated, the related list appears on the CRM record page. A service agent looking at a Contact can now see every Pet Profile, every email engagement, every transaction — without leaving the console. In Service Cloud, you can also add the Related List Enrichment to Case layouts for quick customer context access.

**Person Accounts**: If your CRM uses Person Accounts, create the Related List Enrichment on the Contact object and then apply it to the Person Account layout - it works seamlessly.

### Dynamic Related Lists vs. Standard Related Lists

When you add the enrichment to a page layout, you have two options: a standard related list or a Dynamic Related List. **Use Dynamic Related Lists.** They support display-time filtering, which means you can filter out inactive records, limit the date range, or scope to specific record types — all without physically deleting data from the DMO. This is especially useful for engagement data where you might have years of history but only want to show the last 90 days. Or, in my case, I only wanted to display pets that are marked as Active.

---

## Part 3: Consumption Credits

### How Data 360 Credits Work

Almost every action in Data Cloud consumes credits (ingestion of CRM and Marketing Cloud Engagement data streams is free). Credits are calculated based on the volume of data processed (rows) multiplied by a rate-card multiplier that varies by operation type. The multipliers range dramatically: from 2 credits per million rows for basic data queries up to 100,000 credits per million rows for Identity Resolution.

As of the March 2026 pricing overhaul, credits are fungible across all features, and the old segmented credit types (sandbox credits, segmentation credits, etc.) have been consolidated into a single Data Service Credit pool. Credits are purchased in bundles of 100,000, at approximately $500 per bundle.

### How LWCs on DMO Pages Affect Credits

Here's the good news: **a custom LWC using `getRecord` on a DMO record page has minimal credit impact.** The `getRecord` call through the UI API performs a single-record query. At the Data Query multiplier of 2 credits per million rows, loading one record is essentially free. Even if your team opens hundreds of DMO records per day, the credit impact rounds to zero.

This is one of the rare areas where you can add real value to Data 360 without worrying about your Digital Wallet.

### How Related List Enrichments Affect Credits

This is where it gets more nuanced — and more expensive.

Related List Enrichments query Data 360 every time a CRM user opens a record page that contains the enrichment. Each page load triggers a query against the related DMO(s), and the credit consumption scales with:

- **The number of DMO records related to that individual** — more records = more rows processed per query
- **The number of DMOs being traversed** — if your enrichment requires joining through multiple DMOs to resolve the relationship, every intermediate DMO's rows count toward the total
- **The frequency of page loads** — a high-traffic Service Cloud console where agents are clicking through dozens of contact records per hour means dozens of Data 360 queries per hour, per agent

The Profile Engagement component is a concrete example: displaying Unified Individual engagement data on a Contact record page consumes credits, and extending the date range from 7 days to 30 or 90 days increases the query scope and therefore the credit cost.

### Practical Credit Management Strategies

**For LWCs on DMO pages:**

- These are essentially credit-neutral. Build freely.
- If you're building components that query *related* DMOs (not just the current record), be mindful of the query scope — use `LIMIT` and filter by date range where possible.

**For Related List Enrichments:**

- **Use Dynamic Related Lists with tight filters.** A 30-day lookback on engagement data costs significantly less than the default 2-year lookback.
- **Limit the number of enrichments per page.** Each enrichment on a record page is a separate query. Three Related List Enrichments on a Contact page means three Data 360 queries per page load.
- **Consider Copy Field Enrichments for single-value data.** If you only need one metric from Data 360 (like a Lifetime Value score or a pet count), use a Copy Field Enrichment instead. It syncs the value once and stores it on the CRM record — no per-page-load query cost.
- **Monitor with Digital Wallet.** Set up consumption threshold alerts so you know before you blow through your credit pool. The `TenantDailyEntitlementConsumption` DMO can even be queried in a Flow to automate alerts when daily consumption spikes.
- **Minimize DMO traversal depth.** If your most common Related List Enrichment has to traverse more than 2-3 DMOs to resolve the relationship, your data model may need redesign. Flatter models with related fields co-located on fewer objects reduce join complexity and credit cost.

---

## Putting It All Together

The combination of custom LWCs on DMO pages and Related List Enrichments on CRM pages creates a two-tier strategy:

| Layer | Who uses it | Credit impact | Best for |
|---|---|---|---|
| LWCs on DMO pages | Architects, data engineers, power users | Minimal | Rich data visualization, image rendering, data quality monitoring |
| Related List Enrichments on CRM pages | Service agents, sales reps | Moderate to high (per query) | Surfacing engagement history, calculated insights, cross-system data |
| Copy Field Enrichments on CRM pages | Everyone | One-time sync cost | Single metrics like LTV, scores, flags |

The technical team gets a better Data 360 experience. The front-line teams get Data 360 insights in the tools they already use. And with careful attention to credit consumption — tight filters, minimal DMO traversal, and Copy Fields where possible — you can deliver both without burning through your credit pool.

Data 360 is only as valuable as the people who can access its insights. Build the bridges.