---
layout: post-2
title: "The Right Way to Remove SFMC Data Streams from Salesforce Data 360"
date: 2026-04-01
authors: ["Lesley Higgins"]
categories: ["Data 360", "Marketing Cloud"]
description: A field guide to tearing down your Data 360 infrastructure without losing your mind or your mappings. Learn the proper dependency chain, when to use temporary streams, and how to avoid costly rebuild scenarios.
image: "/assets/images/gen/blog/data-streams.png" 
---

# The Right Way to Remove SFMC Data Streams from Salesforce Data 360

*A field guide to tearing down your Data 360 infrastructure without losing your mind or your mappings.*

---

## The Problem Nobody Warned You About

When Salesforce Marketing Cloud Engagement data is connected to Data 360 via the managed package, it creates a web of data streams, DMO mappings, relationships, segments, enrichments, and Lightning page component bindings that are deeply intertwined. 

When the time comes to restructure (whether you're cleaning up a messy initial implementation, upgrading to a new data model, or simply starting fresh), you will hit a wall of dependency errors.

The error message `Could not delete MktDataModelObject by developerName [DMO_Name] as DMO/DMO fields are in use` is your initiation. It is the platform's way of telling you that you skipped a step. Several steps, probably. And some of those steps involve things you haven't documented yet.

This post is about doing it right: documenting before you delete, understanding what needs to go first, and making a strategic decision about whether to fully tear down your data streams or use a placeholder technique to preserve your DMO integrity.

---

## Document Everything Before You Touch Anything

This cannot be overstated. Before you delete a single data stream or field mapping, take screenshots of every stream's field list and every mapping view. Do this for each stream while it still exists, because once it's gone, the mapping metadata goes with it.

For each data stream, you need to capture:

- Stream name, Object API Name, Object Category, and Data Lake Object Name
- All fields: Field Label, Field API Name, Data Type, Field Used As (Primary Key, Event Time Field), and Formula Field status
- The full mapping view: which source fields map to which DMO fields, on which DMOs, with 0 unmapped fields confirmed or noted
- The DMO's Relationships tab: every relationship, its field, key qualifier, cardinality, related object, related field, and related key qualifier
- Any Related List Enrichments (RLEs) that reference the DMO
- Any Copy Field Enrichments (CFEs) that write from the DMO back to a CRM object
- Any segments that filter on fields from this DMO, including their full criteria, activations, activation targets, and publish schedules
- Any Calculated Insights that use the DMOs
- Any Data Transforms that use the Data Streams or DMOs
- Data Graphs or Semantic Models built off of these Data Streams

### Understanding Stream Dependencies Before You Delete

When you go to remove mappings from Data 360, pay attention to the deletion options presented in the UI: they tell you exactly how many streams are connected to that DMO:

- **If "Delete Object" is the only option:** Your current stream is the only one mapped to that DMO.
- **If "Remove Mappings" is available as an option:** Multiple streams are mapped to that DMO. You can remove your stream's mappings while leaving the DMO intact for other streams.

This distinction is critical for planning your deletion strategy. If you're the only stream feeding a DMO, you need to plan for recreating not just the mappings, but the entire DMO structure, relationships, and any downstream dependencies like segments or enrichments.

Organize this documentation before you start deleting. A Slack Canvas works well for this: it supports tables, checklists, and is accessible to the whole team. The cost of undocumented deletion is recreating mappings from scratch, which is painful and error-prone, especially for streams with 20+ fields mapping to multiple DMOs.

---

## The Dependency Chain: What Must Go First

Data 360 enforces a strict dependency hierarchy. Attempting to delete in the wrong order will produce errors. The correct order is:

**1. Copy Field Enrichments**: These push DMO fields back into Salesforce CRM objects (e.g., syncing `Subscriber_Status__c` from Unified Individual into a Contact field). Delete these first because they hold a reference to the DMO field, not just the DMO itself.

**2. Related List Enrichments**: RLEs surface child DMO records in the Salesforce record detail view. They reference both the DMO and the Unified Object (typically Unified Individual) and must be removed before the DMO can be deleted.

**3. Dynamic Related Lists on Lightning Record Pages**: Similar to RLEs, these components display related DMO data on Lightning pages and must be removed from the page layouts before DMO deletion.

**4. Segments and Activations**: Any segment that uses a DMO field in its Include or Exclude criteria blocks deletion. You must delete the activation first, then the segment. This applies to segments in Draft, Inactive, and Error state as well as Active (the platform does not discriminate).

**5. Calculated Insights**: Any calculated insights that reference fields from the DMO must be deleted before the DMO can be removed. These are particularly painful to lose since they often represent complex business logic that took significant time to configure.

**6. Data Graphs**: Any data graphs that include the DMO or its fields must be deleted before the DMO can be removed.

**7. Semantic Models**: Any semantic models built on top of the data streams or DMOs must be deleted before you can proceed with DMO deletion.

**8. DMO Record Pages and Relationships**: The Relationships tab on the DMO must be cleared of every relationship before the DMO itself can be deleted. This is a common blocker because the error message from the platform is generic (it tells you the DMO is in use but does not tell you which relationship is the culprit). 

Deleting each relationship one by one and waiting for metadata to propagate (typically 5–15 minutes) is the safest approach. Additionally, if any custom Lightning pages have been created for the DMO records themselves, these may need to be updated or removed.

**9. Lightning Page Component Bindings**: This one is easy to miss. If any Data 360 component has been placed on a Lightning record page (whether it's in Salesforce Core or on a Data 360 DMO record page) and that component has been configured to display fields from a specific DMO, the platform will block field-level deletion with a `ComponentInstanceProperty` error referencing a `0AJ`-prefixed ID.

Go to Setup → Lightning App Builder, find all console app pages, and remove any Data 360 enrichment components completely before attempting deletions. Also go to Data Explorer, search for the DMO that is causing problems, view a record for the DMO and select Edit Page if any fields besides the Record Id is displayed.

**10. Data Stream Field Mappings**: Once all dependencies above are cleared, you can remove the field mappings from the data stream itself.

**11. The DMO**: Once unmapped, the DMO can be deleted. However, there is no need to delete the DMO if you are planning on using it in the future.

**12. The Data Stream**: Finally, delete the data stream.

---

## The SFMC Managed Package vs. Custom DE Streams: Two Different Strategies

Here is the key strategic insight that makes this whole process much cleaner: **not all data streams should be treated the same way.**

### Managed Package Streams: Full Teardown Is Correct

Data streams created by the SFMC managed package (things like SFMC Email Engagement Open [BU number] and SFMC Contact Point Email [BU number]) can be fully unmapped and deleted as part of a proper infrastructure overhaul. These streams are recreated automatically when you reinstall or reconfigure the connector. 

Their DMO mappings are standardized and well-documented by Salesforce. There is no risk of losing institutional knowledge by tearing them down completely, because you are not the one who designed them. Just keep in mind that when you reingest those data streams, Marketing Cloud will only send 90 days worth of data. 

### Custom SFMC DE Streams: Use a Placeholder Temp Stream Instead

For data streams sourced from custom Marketing Cloud Data Extensions (especially those with complex multi-DMO mappings, custom field configurations, and extensive downstream dependencies), a full teardown becomes a massive pain when you've already invested significant time in building segments, relationships, calculated insights, enrichments, and other DMO-dependent configurations. 

The problem is this: if you've spent weeks or months carefully mapping custom DE data across multiple DMOs and building segments, calculated insights, and relationships on top of those mappings, tearing down all those dependencies just to rebuild an identical structure with a new data stream is an enormous waste of time. 

You're forced to work through the entire dependency chain in reverse order, document every configuration, delete everything systematically, then rebuild it all from scratch. In many cases, you may spend hours trying to figure out where that last dependency is.

**The solution: export the DE from Marketing Cloud, import it into Data 360 as a temporary file-based ingestion stream, replicate the existing field mappings on the temp stream, then remove the mappings from the original SFMC stream and delete it.**

This approach is particularly worthwhile when you've already invested significant time building segments, relationships, calculated insights, and enrichments. Instead of tearing down all that work, you create a temporary placeholder that keeps everything intact while you rebuild the proper SFMC connection.

### Step-by-Step Process for Creating a Temp Stream

1. **Export the DE data** from Marketing Cloud as a CSV file
2. **In Data 360, go to Data Streams → New → File Upload**
3. **Before uploading**: Note the original stream's Object Category (Profile, Engagement, or Other), you'll need to match this exactly
4. **Create any formula fields** in the temp stream definition while building the stream. You cannot add formula fields after the fact
5. **Pay special attention to date and number fields**: for example, the file upload often detects date and datetime fields as text, but they must be properly type Date or DateTime to map correctly to DMO fields
6. **Replicate the field mappings** from the original SFMC stream to your temp stream
7. **Verify the mappings work** by checking that data flows to the DMO
8. **Remove mappings** from the original SFMC stream (now that the temp stream is feeding the DMO)
9. **Delete the original SFMC stream**

This technique works because:
- The DMO mappings transfer cleanly to the temp stream, preserving the DMO's integrity
- The DMO can remain in place, keeping all downstream dependencies (segments, relationships, RLEs) intact
- The SFMC stream can be deleted without triggering DMO deletion errors
- Once the new SFMC stream is ready, you swap the temp stream out and remap properly

---

## The Recommended Framework

Combining both strategies gives you a clean decision tree:

**If the stream was created by the SFMC managed package connector:** Document it, clear its dependencies in order, and tear it down completely. Recreate from scratch with the new connector configuration.

**If the stream was created from a custom Marketing Cloud Data Extension:** Export the DE from SFMC, import it as a temporary file-based stream in Data 360, replicate the mappings on the temp stream, then remove mappings from the original SFMC stream and delete it. 

The temp stream acts as a placeholder that keeps the DMO alive and all downstream dependencies intact while you rebuild the proper connection.

In both cases, the golden rule is the same: **document before you delete**. Screenshots of every mapping view, every relationship table, every segment criteria screen, every RLE detail page. All of it. You will not regret the ten extra minutes it takes. You will absolutely regret skipping it.

---

## What to Document and Where to Keep It

A minimal documentation entry for each data stream should include the Object API Name, total records, object category, primary key, event time field if applicable, a complete field table with API names and data types, a complete mapping table for each target DMO, and a remapping checklist.

For dependencies, document each one in a dedicated section: segments with their full criteria and activation details, RLEs with their child relationship name and unified object, DMO relationships with all key qualifiers, and CFEs with their source and target field API names.

Keep all of this somewhere your team can access during the rebuild: a Slack Canvas, a Notion page, or a GitHub markdown file all work well. The documentation serves two purposes: it guides the deletion process in the right order, and it is the blueprint for recreation once the new infrastructure is ready.

---

## Final Takeaway

Salesforce Data 360's dependency enforcement is strict by design: it prevents you from leaving orphaned DMOs and broken references in your data model. Working with that enforcement rather than against it means doing the documentation upfront, following the dependency chain in order, and making smart decisions about when to tear down vs. when to use a placeholder to preserve DMO continuity.

The temp stream technique in particular is underutilized. It turns what would otherwise be a high-risk, all-or-nothing deletion into a controlled swap, and it is available to anyone with access to their source DE data and the ability to export a CSV.

---

*Written from real-world experience rebuilding a production Weruva Data 360 implementation. All errors encountered were real. All workarounds were validated.*
