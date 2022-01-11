---
weight: 1
title: "Application and Package Deployments Reporting by Collection with MEMCM/SCCM"
date: 2018-05-23T16:51:00.786Z
lastmod: 2019-08-23T12:36:41+03:00
draft: false
author: "Ioan Popovici"
author_link: "https://sccm.zone/"

#subtitle: "Deployments reports for all collection members available get it while it’s hot…"
description: "Lists the application or package deployments for a MEMCM/SCCM device or user collection."
featuredImage: "cover-package_delivery_street_sign.webp"

tags: ["SQL", "SSRS", "MEMCM", "SCCM", "Applications", "Packages", "Deployments", "Security"]
categories: ["Reports"]
aliases: ["/package-and-application-deployments-reporting-by-collection-with-sccm-64199bbcdc6c"]
lightgallery: true

toc:
  auto: false
---

## Summary
No default reports exists that can display results for each collection member. There is a report that lists deployments for a specific collection but that’s it. If a collection member has a deployment on a different collection you are out of luck. Well not anymore…

I used the [`All package and program deployments`](https://docs.microsoft.com/en-us/sccm/core/servers/manage/list-of-reports#software-distribution---package-and-program-deployment) to a specified user/device as reference. One of the problems was that the `User` report has a bug and returns `NULL` for most packages. Since the views are so well documented, it took a while to get it right.

## Release History

{{< admonition type=info title="Changelog" open=false >}}
{{< include user="Ioan-Popovici" repo="SCCM-Zone" path="Reporting/Software/SW Installed Software by Computer or Publisher/CHANGELOG.md" >}}
{{< /admonition >}}

## Prerequisites

* Report (Follow link → Copy/Download)
  * [DE Software Deployments by Device or User.rdl](https://snippets.cacher.io/snippet/a8c54490242f96c2f43a)

## Installation

### Upload Report to SSRS

* Start Internet Explorer and navigate to [`http://<YOUR_REPORT_SERVER_FQDN>/Reports`](http://en.wikipedia.org/wiki/Fully_qualified_domain_name)
* Choose a path and upload the previously downloaded report file.

### Configure Imported Report

* Replace the [`DataSource`](https://joshheffner.com/how-to-import-additional-software-update-reports-in-sccm/) in the report.

## Code

### SQL Queries

For reference only, reports already include these queries.

#### Application Deployments

```sql
/*
.SYNOPSIS
    Lists the Application Deployments for a Collection.
.DESCRIPTION
    Lists the Application Deployments for a Device or User Collection.
.NOTES
    Created by Ioan Popovici
    Part of a report should not be run separately.
.LINK
    https://SCCM.Zone/DE-Deployments-by-Device-or-User
.LINK
    https://SCCM.Zone/DE-Deployments-by-Device-or-User-CHANGELOG
.LINK
    https://SCCM.Zone/DE-Deployments-by-Device-or-User-GIT
.LINK
    https://SCCM.Zone/Issues
*/

/*##=============================================*/
/*## QUERY BODY                                  */
/*##=============================================*/

/* Testing variables !! Need to be commented for Production !! */

/*
DECLARE @UserSIDs VARCHAR(16)= 'Disabled';
DECLARE @CollectionID VARCHAR(16)= 'HUB005A6';
--DECLARE @CollectionID VARCHAR(16)= 'HUB00744';
DECLARE @SelectBy VARCHAR(16);
DECLARE @CollectionType VARCHAR(16);
SELECT @SelectBy = ResourceID
FROM fn_rbac_FullCollectionMembership(@UserSIDs) AS CollectionMembers
WHERE CollectionMembers.CollectionID = @CollectionID
    AND CollectionMembers.ResourceType = 5; --Device collection
IF @SelectBy > 0
    SET @CollectionType = 2;
ELSE
    SET @CollectionType = 1;
*/

/* Initialize CollectionMembers table */
DECLARE @CollectionMembers TABLE (
    ResourceID     INT
    , ResourceType INT
    , SMSID        NVARCHAR(100)
)

/* Populate CollectionMembers table */
INSERT INTO @CollectionMembers (ResourceID, ResourceType, SMSID)
SELECT ResourceID, ResourceType, SMSID
FROM fn_rbac_FullCollectionMembership(@UserSIDs) AS CollectionMembers
WHERE CollectionMembers.CollectionID = @CollectionID
    AND CollectionMembers.ResourceType IN (4, 5); --Only Users or Devices

/* User collection query */
IF @CollectionType = 1
    BEGIN
        SELECT DISTINCT
            UserName           = Users.Unique_User_Name0
            , PrimaryUser      = Devices.PrimaryUser
            , TopConsoleUser   = Console.TopConsoleUser0
            , SoftwareName     = Deployments.SoftwareName
            , CollectionName   = Deployments.CollectionName
            , Device           = AssetData.MachineName
            , Manufacturer     = Enclosure.Manufacturer0
            , DeviceType       = (
                CASE
                    WHEN Enclosure.ChassisTypes0 IN (8, 9, 10, 11, 12, 14, 18, 21, 31, 32) THEN 'Laptop'
                    WHEN Enclosure.ChassisTypes0 IN (3, 4, 5, 6, 7, 15, 16) THEN 'Desktop'
                    WHEN Enclosure.ChassisTypes0 IN (17, 23, 28, 29) THEN 'Servers'
                    WHEN Enclosure.ChassisTypes0 = '30' THEN 'Tablet'
                    ELSE 'Unknown'
                END
            )
            , ChassisType      = (
                CASE Enclosure.ChassisTypes0
                    WHEN '1' THEN 'Other'
                    WHEN '2' THEN 'Unknown'
                    WHEN '3' THEN 'Desktop'
                    WHEN '4' THEN 'Low Profile Desktop'
                    WHEN '5' THEN 'Pizza Box'
                    WHEN '6' THEN 'Mini Tower'
                    WHEN '7' THEN 'Tower'
                    WHEN '8' THEN 'Portable'
                    WHEN '9' THEN 'Laptop'
                    WHEN '10' THEN 'Notebook'
                    WHEN '11' THEN 'Hand Held'
                    WHEN '12' THEN 'Docking Station'
                    WHEN '13' THEN 'All in One'
                    WHEN '14' THEN 'Sub Notebook'
                    WHEN '15' THEN 'Space-Saving'
                    WHEN '16' THEN 'Lunch Box'
                    WHEN '17' THEN 'Main System Chassis'
                    WHEN '18' THEN 'Expansion Chassis'
                    WHEN '19' THEN 'SubChassis'
                    WHEN '20' THEN 'Bus Expansion Chassis'
                    WHEN '21' THEN 'Peripheral Chassis'
                    WHEN '22' THEN 'Storage Chassis'
                    WHEN '23' THEN 'Rack Mount Chassis'
                    WHEN '24' THEN 'Sealed-Case PC'
                    WHEN '25' THEN 'Multi-system chassis'
                    WHEN '26' THEN 'Compact PCI'
                    WHEN '27' THEN 'Advanced TCA'
                    WHEN '28' THEN 'Blade'
                    WHEN '29' THEN 'Blade Enclosure'
                    WHEN '30' THEN 'Tablet'
                    WHEN '31' THEN 'Convertible'
                    WHEN '32' THEN 'Detachable'
                    ELSE 'Undefinded'
                END
            )
            , SerialNumber     = Enclosure.SerialNumber0
            , Purpose          = (
                CASE
                    WHEN Assignments.DesiredConfigType = 1
                    THEN 'Install'
                    ELSE 'Remove'
                END
            )
            , InstalledBy      = Users.Unique_User_Name0
            , EnforcementState = (
                dbo.fn_GetAppState(AssetData.ComplianceState, AssetData.EnforcementState, Assignments.OfferTypeID, 1, AssetData.DesiredState, AssetData.IsApplicable)
            )
        FROM fn_rbac_CollectionExpandedUserMembers(@UserSIDs) AS CollectionMembers
            INNER JOIN v_R_User AS Users ON Users.ResourceID = CollectionMembers.UserItemKey
            INNER JOIN v_DeploymentSummary AS Deployments ON Deployments.CollectionID = CollectionMembers.SiteID
            LEFT JOIN v_AppIntentAssetData AS AssetData ON AssetData.UserName = Users.Unique_User_Name0
                AND AssetData.AssignmentID = Deployments.AssignmentID
            INNER JOIN v_CIAssignment AS Assignments ON Assignments.AssignmentID = Deployments.AssignmentID
			LEFT JOIN v_GS_SYSTEM_ENCLOSURE AS Enclosure ON Enclosure.ResourceID = AssetData.MachineID
            LEFT JOIN v_GS_SYSTEM_CONSOLE_USAGE AS Console ON Console.ResourceID = AssetData.MachineID
			LEFT JOIN  v_CombinedDeviceResources AS Devices ON Devices.MachineID = AssetData.MachineID
        WHERE Deployments.FeatureType = 1
            AND Users.Unique_User_Name0 IN (
                SELECT SMSID
                FROM @CollectionMembers
                WHERE ResourceType = 4 --Ony Users
            )
    END;

/* Device collection query */
IF @CollectionType = 2
    BEGIN
        SELECT DISTINCT
            Device             = Devices.Name
			, PrimaryUser      = Devices.PrimaryUser
            , TopConsoleUser   = Console.TopConsoleUser0
            , Manufacturer     = Enclosure.Manufacturer0
            , DeviceType       = (
                CASE
                    WHEN Enclosure.ChassisTypes0 IN (8 , 9, 10, 11, 12, 14, 18, 21, 31, 32) THEN 'Laptop'
                    WHEN Enclosure.ChassisTypes0 IN (3, 4, 5, 6, 7, 15, 16) THEN 'Desktop'
                    WHEN Enclosure.ChassisTypes0 IN (17, 23, 28, 29) THEN 'Servers'
                    WHEN Enclosure.ChassisTypes0 = '30' THEN 'Tablet'
                    ELSE 'Unknown'
                END
            )
            , ChassisType      = (
                CASE Enclosure.ChassisTypes0
                    WHEN '1' THEN 'Other'
                    WHEN '2' THEN 'Unknown'
                    WHEN '3' THEN 'Desktop'
                    WHEN '4' THEN 'Low Profile Desktop'
                    WHEN '5' THEN 'Pizza Box'
                    WHEN '6' THEN 'Mini Tower'
                    WHEN '7' THEN 'Tower'
                    WHEN '8' THEN 'Portable'
                    WHEN '9' THEN 'Laptop'
                    WHEN '10' THEN 'Notebook'
                    WHEN '11' THEN 'Hand Held'
                    WHEN '12' THEN 'Docking Station'
                    WHEN '13' THEN 'All in One'
                    WHEN '14' THEN 'Sub Notebook'
                    WHEN '15' THEN 'Space-Saving'
                    WHEN '16' THEN 'Lunch Box'
                    WHEN '17' THEN 'Main System Chassis'
                    WHEN '18' THEN 'Expansion Chassis'
                    WHEN '19' THEN 'SubChassis'
                    WHEN '20' THEN 'Bus Expansion Chassis'
                    WHEN '21' THEN 'Peripheral Chassis'
                    WHEN '22' THEN 'Storage Chassis'
                    WHEN '23' THEN 'Rack Mount Chassis'
                    WHEN '24' THEN 'Sealed-Case PC'
                    WHEN '25' THEN 'Multi-system chassis'
                    WHEN '26' THEN 'Compact PCI'
                    WHEN '27' THEN 'Advanced TCA'
                    WHEN '28' THEN 'Blade'
                    WHEN '29' THEN 'Blade Enclosure'
                    WHEN '30' THEN 'Tablet'
                    WHEN '31' THEN 'Convertible'
                    WHEN '32' THEN 'Detachable'
                    ELSE 'Undefinded'
                END
            )
			, SerialNumber     = Enclosure.SerialNumber0
            , SoftwareName     = Deployments.SoftwareName
            , CollectionName   = Deployments.CollectionName
            , Purpose          = (
                CASE
                    WHEN Assignments.DesiredConfigType = 1
                    THEN 'Install'
                    ELSE 'Remove'
                END
            )
            , InstalledBy      = AssetData.UserName
            , EnforcementState = Dbo.fn_GetAppState(AssetData.ComplianceState, AssetData.EnforcementState, Assignments.OfferTypeID, 1, AssetData.DesiredState, AssetData.IsApplicable)
        FROM v_CombinedDeviceResources AS Devices
            INNER JOIN fn_rbac_FullCollectionMembership(@UserSIDs) AS CollectionMembers ON CollectionMembers.ResourceID = Devices.MachineID
                AND CollectionMembers.ResourceType = 5 --Only Devices
            INNER JOIN v_DeploymentSummary AS Deployments ON Deployments.CollectionID = CollectionMembers.CollectionID
                AND Deployments.FeatureType = 1
            LEFT JOIN v_AppIntentAssetData AS AssetData ON AssetData.MachineID = CollectionMembers.ResourceID
                AND AssetData.AssignmentID = Deployments.AssignmentID
            INNER JOIN v_CIAssignment AS Assignments ON Assignments.AssignmentID = Deployments.AssignmentID
			LEFT JOIN v_GS_SYSTEM_ENCLOSURE AS Enclosure ON Enclosure.ResourceID = Devices.MachineID
            LEFT JOIN v_GS_SYSTEM_CONSOLE_USAGE AS Console ON Console.ResourceID = Devices.MachineID
        WHERE Devices.isClient = 1
            AND Devices.MachineID IN (
                SELECT ResourceID
                FROM @CollectionMembers
                WHERE ResourceType = 5 --Only Devices
            )
    END;

/*##=============================================*/
/*## END QUERY BODY                              */
/*##=============================================*/
```

{{< gist Ioan-Popovici cd400114066f58bf1f931fb0523e63ea >}}

{{< details "[Click to expand]" >}}

<script src="https://embed.cacher.io/815539d65961ad15a9ae47c50d7913a67f5aad10.js?a=05f6022fa311628003703128e475bcfb&t=github_gist"></script>

{{< /details >}}

#### Package Deployments

{{< details "[Click to expand]" >}}
<script src="https://embed.cacher.io/d7046c89083aac12a0fb13c55f2e1bf67909fd48.js?a=887e02ddfdf124481a69303f24b3a93c&t=github_gist"></script>
{{< /details >}}

## Preview

{{< image src="preview-de-software-deployments-by-device-or-user_a.webp" caption="User application deployments">}}

{{< image src="preview-de-software-deployments-by-device-or-user_b.webp" caption="Device application deployments">}}

> **Notes**
> I did not post screenshots with package deployments because of severe laziness but you get the picture.

***

{{< image src="meme-hold-my-beer-i-got-this.webp" caption="Hold my beer meme :)" align="Center" >}}

***

## Support

**Please use [Github](http://MEM.Zone/GIT) for 🐛 reporting, or 🌈 and 🦄 requests**
