---
layout: post
title: "Find the Nearest Store/Office to an Ad Studio Lead"
date: 2020-04-11
authors: ["Lesley Higgins"]
categories: ["Marketing Cloud"]
description: Learn how to build an automation that captures leads from Facebook Ad Studio and automatically determines their nearest office location using Google's Geocode API, SQL, AMPScript, and SSJS.
image: "/assets/images/gen/blog/geo-map.png"
---

<p align="center">
  <video width="800" autoplay loop muted playsinline style="border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1);">
    <source src="/assets/images/gen/blog/geolocate-map.mp4" type="video/mp4">
    <img src="/assets/images/gen/blog/geolocate-map-thumb.png" alt="Geo-locate subscribers to nearest office illustration" width="800">
  </video>
</p>

## Requirements

- **Platform Tools:** Marketing Cloud, Ad Studio, Content Builder, and Automation Studio
- **Bonus:** CloudPages and Email Studio
- **Code Skills:** SQL, AMPScript, SSJS, oh my!

This is an automation that I was very excited to finally piece together. Shout out to [Traversy Media](https://www.youtube.com/channel/UC29ju8bIPH5as8OGnQzwJyA) and [Adam Spriggs](https://sprignaturemoves.com/geolocation-in-sfmc/) for putting solutions out into the universe that I relied on heavily while piecing together the following solution.

## The Challenge

**The problem I was attempting to solve:** I have a client that has multiple office locations. They are running Facebook ads that use a lead collect form to gather the names, email addresses, and zip codes of potential clients. We have dynamic content blocks built with maps and contact information for each office. We want to be able to send a welcome email to new leads with the location of their nearest office. They also do not have Salesforce, so I could not leverage the Geolocation Custom Field through Marketing Cloud Connect.

## The Complete Automation

### Step 1: Query Campaign Data into Master Leads

Query the campaign-specific data extension into a Master Leads data extension. This is going to be the one manual process in all of this because a new lead capture in Ad Studio creates a new data extension that is then populated by new leads. Whenever you add a new campaign in Ad Studio, you are going to have to add the new data extension to the first step of the automation. You should also take the time to remove old campaigns that are no longer running.

Run this query as an update with Lead Id as the primary key. Additionally, you can keep the Master Leads data extension to a smaller size by only capturing leads from the last day and setting data retention on the destination data extension to a short period, like one or two days.

I am also going to join the new leads with a master list that I am maintaining of all leads in my database. I am doing this because I want to get as much address information as possible for an incoming lead. If you have an audience that you are targeting in Ad Studio, and that audience exists in your marketing cloud universe with a full address, and your lead capture form is only asking for a zip code, you might as well try to incorporate as much address information as you can into the address we are going to target with the API.

**Sample query:**

```sql
SELECT DISTINCT
    n.Lead_Id,
    n.Email,
    n.First_Name,
    n.Last_Name,
    CASE 
        WHEN m.Zip_Code = n.Zip_Code THEN m.Address_Line_1
        ELSE NULL
    END AS Address_Line_1,
    CASE 
        WHEN m.Zip_Code = n.Zip_Code THEN m.City
        ELSE NULL
    END AS City,
    CASE 
        WHEN m.Zip_Code = n.Zip_Code THEN m.State
        ELSE NULL
    END AS State,
    n.Zip_Code,
    n.Created_Date
FROM [New_Leads_DE] n
LEFT JOIN [Master_Contacts_DE] m ON n.Email = m.Email
WHERE n.Created_Date >= DateAdd(day, -1, GETDATE())
```

In addition to the above fields, the destination data extension should also have the following fields:
- **Latitude** (set a default of 0)
- **Longitude** (set a default of 0)
- **XAxis**
- **YAxis**
- **ZAxis**

### Step 2: Clean Address Data

Remove # signs from addresses. While you can manually type an address that includes a # into a call to Google's Geocode API, the AMPScript below has issues concatenating the URL when there is a # in the address. The # might come up when someone enters an apartment, suite or mailbox number, so it's best to just get rid of it now.

The below code is a little overkill - the odds of someone putting a # in the City, State or Zip field is very slim, but there's no reason to not get rid of any potential typos.

```sql
UPDATE [Master_Leads_DE] 
SET 
    Address_Line_1 = REPLACE(Address_Line_1, '#', ' '),
    City = REPLACE(City, '#', ' '),
    State = REPLACE(State, '#', ' '),
    Zip_Code = REPLACE(Zip_Code, '#', ' ')
WHERE 
    Address_Line_1 LIKE '%#%' 
    OR City LIKE '%#%' 
    OR State LIKE '%#%' 
    OR Zip_Code LIKE '%#%'
```

I replaced # signs with just a space. You could also replace it with Apt, Ste, the word "number," or whatever you want. However, the Google Geocode API does not need to be well-formatted, so inserting a space is adequate.

### Step 3: Set Up Google's Geocoding API

Sign up for [Google's API](https://developers.google.com/maps/documentation/javascript/geocoding). You will need to sign up using a google login and start a new project. The "Getting Started" section on that page will walk you through the steps to get up and rolling. Once you have an API key, you are ready for the scripting step.

You probably want to start this step by creating a CloudPage (if you have them enabled in your account). A CloudPage will allow you to test your code much quicker than where the code will ultimately live, which is in an HTML content block.

### Step 4: AMPScript and SSJS for Geocoding

Using AMPScript, you are going to run new leads through a loop. To start, target the new leads that have not yet been through the Geocode API with a LookupRows, using Latitude and Longitude = 0. You could probably just use one of these, but it would be possible for someone to be located at 0 latitude OR 0 longitude. *What about 0 for both?* [Here's a link](https://www.thoughtco.com/prime-meridian-and-the-equator-intersect-4070819) so you can easily go down that rabbit hole...

The reason we are grabbing only leads with a 0 in those fields (which was populated by the default settings on the data extension), is to minimize the volume of leads that are processed on each run. This is so the script doesn't time out and also to keep down the costs of making API calls. Google's Geocode API allows $200 per month free of API calls. If you avoid hitting that number, then you don't have to pay for this automation. From my limited experience, if you are a small business generating social leads you will never get anywhere near $200 per month.

**The complete geocoding script:**

```html
%%[
VAR @rows, @row, @rowCount, @counter, @Lead_Id, @address, @fullAddress, @getRequest
VAR @lat, @lng, @rowsUpdated, @Address_Line_1, @City, @State, @Zip_Code

SET @counter = 1
SET @rows = LookupRows("Master_Leads_DE","Latitude","0")
SET @rowCount = RowCount(@rows)

IF @rowCount > 0 THEN
    FOR @counter = 1 TO @rowCount DO
        SET @row = Row(@rows, @counter)
        SET @Lead_Id = Field(@row,"Lead_Id")
        SET @Address_Line_1 = Field(@row,"Address_Line_1")
        SET @City = Field(@row,"City")
        SET @State = Field(@row,"State")
        SET @Zip_Code = Field(@row,"Zip_Code")
        
        /* Build address string - prioritize form data over matched data */
        IF NOT EMPTY(@Address_Line_1) AND @Zip_Code == Field(@row,"Zip_Code") THEN
            SET @fullAddress = CONCAT(@Address_Line_1, " ", @City, " ", @State, " ", @Zip_Code)
        ELSE
            SET @fullAddress = @Zip_Code
        ENDIF
        
        /* Call Google Geocoding API */
        SET @getRequest = HTTPGet(CONCAT("https://maps.googleapis.com/maps/api/geocode/json?address=", @fullAddress, "&key=YOUR_API_KEY_HERE"))
]%%

<script runat="server">
    Platform.Load("core", "1.1.1");
    
    try {
        var getRequest = Variable.GetValue("@getRequest");
        var response = Platform.Function.ParseJSON(getRequest);
        
        var lat = 0;
        var lng = 0;
        
        if (response.status == "OK" && response.results.length > 0) {
            lat = response.results[0].geometry.location.lat;
            lng = response.results[0].geometry.location.lng;
        }
        
        Variable.SetValue("@lat", lat);
        Variable.SetValue("@lng", lng);
        
    } catch(ex) {
        Variable.SetValue("@lat", 0);
        Variable.SetValue("@lng", 0);
    }
</script>

%%[
        /* Update the data extension with coordinates */
        SET @rowsUpdated = UpdateData(
            "Master_Leads_DE", 
            1,
            "Lead_Id", @Lead_Id,
            "Latitude", @lat,
            "Longitude", @lng
        )
        
    NEXT @counter
ENDIF
]%%
```

### Step 5: Test Your CloudPage

Run this code on a CloudPage by selecting Schedule/Publish. You do not need to actually publish this page, the code will run in the preview screen. You can check the data extension to see if it has been updated, or add a few lines of HTML before the NEXT @counter to print the results. Delete this bit before moving to the HTML block:

```html
Lat: %%=v(@lat)=%%<br>
Long: %%=v(@lng)=%%<br>
rowsUpdated: %%=v(@rowsUpdated)=%%<br><br>
```

If all goes well, copy this code over to an HTML block in Content Builder and note the ID of the content block.

### Step 6: Call HTML Block from Automation Studio

Call the HTML block from Automation Studio using SSJS. You cannot put AMPScript into a Script activity in Automation Studio (huge annoying bummer), so you have to call an HTML content block using SSJS. Drop a Script Activity onto your Automation canvas and add the following script with your content block ID number:

```javascript
<script runat="server">
Platform.Load("core","1.1.1");

var contentBlockID = "YOUR_CONTENT_BLOCK_ID_HERE";
var contentBlock = Platform.Function.ContentBlockByID(contentBlockID);
</script>
```

**Important**: Make sure your Automation Studio sequence includes:
1. SQL Query Activity (Step 1) 
2. SQL Query Activity (Step 2) 
3. Script Activity (Step 6)
4. SQL Query Activity (Step 7)
5. SQL Query Activity (Step 8)

### Step 7: Calculate X, Y, Z Coordinates

This step and the next step could be combined into one, but for the sake of not wanting my head to explode when I look at my SQL code, I am splitting it into two processes.

We are going to calculate the x-axis, y-axis, and z-axis from the latitude and longitude and update the leads data extension. This is going to give us a three-dimensional point in space, rather than two sides of a triangle, which is what latitude and longitude provide.

```sql
UPDATE [Master_Leads_DE]
SET 
    XAxis = COS(RADIANS(Latitude)) * COS(RADIANS(Longitude)),
    YAxis = COS(RADIANS(Latitude)) * SIN(RADIANS(Longitude)),
    ZAxis = SIN(RADIANS(Latitude))
WHERE 
    Latitude != 0 
    AND Longitude != 0
    AND (XAxis IS NULL OR YAxis IS NULL OR ZAxis IS NULL)
```

### Step 8: Find Nearest Office Location

This final step assumes you have already applied the above processes to a data extension that holds all of your offices/stores/etc. The list of locations you are going to compare your leads to should also have x-axis, y-axis, and z-axis coordinates.

The below SQL will calculate the distance between each lead and each office/store/etc and return only the shortest distance, thanks to the rank field:

```sql
SELECT 
    l.Lead_Id,
    l.Email,
    l.First_Name,
    l.Last_Name,
    o.Office_Name,
    o.Office_Address,
    o.Office_City,
    o.Office_State,
    o.Office_Phone,
    SQRT(
        POWER(l.XAxis - o.XAxis, 2) + 
        POWER(l.YAxis - o.YAxis, 2) + 
        POWER(l.ZAxis - o.ZAxis, 2)
    ) AS Distance
FROM [Master_Leads_DE] l
CROSS JOIN [Office_Locations_DE] o
WHERE 
    l.XAxis IS NOT NULL 
    AND l.YAxis IS NOT NULL 
    AND l.ZAxis IS NOT NULL
QUALIFY ROW_NUMBER() OVER (PARTITION BY l.Lead_Id ORDER BY Distance ASC) = 1
```

Populate a new data extension with this information. You can now use this data extension to enter a Welcome Journey that has location-specific information.

## Conclusion

This automation combines the power of Marketing Cloud's automation capabilities with Google's Geocoding API to create a seamless experience for new leads. By automatically determining the nearest office location, you can provide personalized, location-specific content that enhances the customer experience and increases engagement.

The setup requires some initial configuration and testing, but once implemented, it runs automatically and scales with your lead volume. The combination of SQL for data manipulation, AMPScript for API calls, and SSJS for JSON parsing creates a robust solution that bridges the gap between lead capture and personalized communication.