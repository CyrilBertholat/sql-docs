---
title: "Grant permissions on report server items on a SharePoint site"
description: "Grant permissions on report server items on a SharePoint site"
author: maggiesMSFT
ms.author: maggies
ms.date: 09/25/2024
ms.service: reporting-services
ms.subservice: security
ms.topic: conceptual
ms.custom:
  - updatefrequency5
helpviewer_keywords:
  - "permissions [Reporting Services], SharePoint integrated mode"
  - "SharePoint integration [Reporting Services], permissions"
  - "permissions [Reporting Services], native mode"
  - "security [Reporting Services], SharePoint integrated mode"
---
# Grant permissions on report server items on a SharePoint site
  [!INCLUDE[msCoName](../../includes/msconame-md.md)] [!INCLUDE[SPF2010](../../includes/spf2010-md.md)] provides built-in security features that you can use to grant access to report server items that you access from SharePoint sites and libraries. If you already assigned permissions to users, those same users will have access to report server items and operations immediately after you configure the integration settings between [!INCLUDE[SPF2010](../../includes/spf2010-md.md)] and a report server. You can use existing permissions to upload report definitions and other documents, view reports, create subscriptions, and manage items.  
  
 If you didn't assign permissions or if you aren't familiar with the security features in [!INCLUDE[SPF2010](../../includes/spf2010-md.md)], follow these guidelines:  
  
1.  In the product documentation for [!INCLUDE[SPF2010](../../includes/spf2010-md.md)], read about the default security settings for the standard SharePoint groups so that you know how to manage permissions and user access.  
  
2.  Review the list of permissions that specifically affect access to report server items and operations. For more information, see [Use built-in security in Windows SharePoint services for report server items](../../reporting-services/security/use-built-in-security-in-windows-sharepoint-services-for-report-server-items.md).  
  
3.  Assign user and group accounts to predefined SharePoint groups.  
  
4.  Optionally, create new permission levels and groups, or modify existing ones to vary server access permissions as specific needs arise.  
  
 To use [!INCLUDE[SPF2010](../../includes/spf2010-md.md)] security features with report server items, you must have a report server that runs in SharePoint integrated mode.  
  
## About permissions, permission levels and SharePoint groups  
 The following list provides a brief introduction to the security features in [!INCLUDE[SPF2010](../../includes/spf2010-md.md)]. For more information, see "Windows SharePoint Help and How-to" on your SharePoint site.  
  
-   Securable objects include sites, lists, libraries, folders, and documents.  
  
-   A permission is an authorization to perform a specific task. [!INCLUDE[SPF2010](../../includes/spf2010-md.md)] provides 33 predefined permissions that you can combine into a permission level.  
  
-   A permission level is a set of permissions that can be granted to users or SharePoint groups on a securable object such as a site, library, list, folder, item, or document. It's equivalent to a role definition in [!INCLUDE[ssRSnoversion](../../includes/ssrsnoversion-md.md)]. There are five predefined permission levels. You can customize them or create new ones if needed.  
  
-   A SharePoint group is a group of users that you can create on a SharePoint site to manage permissions to the site and to provide an e-mail distribution list for site members. A SharePoint group consists of Windows user and group accounts, or user logins if you're using Forms authentication. [!INCLUDE[SPF2010](../../includes/spf2010-md.md)] provides three groups. You can customize them or create new ones if needed.  
  
-   Permission inheritance allows subsites, lists and libraries, and items to inherit the security settings of the parent site. You can use inherited permissions to access report server items that are stored in a SharePoint library. Use permission inheritance and the predefined SharePoint groups to help simplify your deployment and provide immediate access to most report server operations.  
  
## Who sets permissions  
 The administrator who installs [!INCLUDE[SPF2010](../../includes/spf2010-md.md)], runs the SharePoint Configuration Wizard, and creates the portal site becomes the default portal site owner. The site owner can set permissions in Central Administration for a farm or a stand-alone SharePoint Web application, and can set permission at the top-level site for each SharePoint Web application. This person can also designate other site owners.  
  
 At the top-level site of a SharePoint Web application, site collection administrators can set permissions for multiple sites throughout the site hierarchy. Individual site owners can perform the same tasks relative to a subsite.  
  
 A server administrator or a site collection administrator can set options that determine whether other site owners can set permissions. Depending on the level of permissions you have, you might not be able to create or customize SharePoint groups or permission levels.  
  
## Use predefined SharePoint groups and permission levels  
 Recommendations in [!INCLUDE[SPF2010](../../includes/spf2010-md.md)] product documentation suggest that you use standard SharePoint groups (which are *Site name* **Owners**, *Site name* **Members**, and *Site name* **Visitors**) and assign permissions at the site level. Most users that you assign permissions to should be members of the *Site name* **Visitors** or *Site name* **Members** groups. Permissions on the parent site are inherited throughout the site hierarchy. You can break permission inheritance for specific items that require other restrictions.  
  
 The following SharePoint groups have the following predefined permission levels:  
  
-   The **Owners** group has Full Control permissions, which enable group members to change the site content, pages, or functionality. Full Control access should be limited to site administrators only.  
  
-   The **Members** group has Contribute level permissions, which allow group members to view pages, edit items, submit changes for approval, add, and delete items from a list.  
  
-   The **Visitors** group has Read level permission, which enables group members to view pages, list items, and documents.  
  
 The SharePoint groups have permission levels that provide immediate access to many report server operations. You can create custom groups or permissions levels if you find that the built-in security settings don't provide the level of access that you need.  
  
 For more information about which report server operations are supported through the default security features, see [Use Built-in Security in Windows SharePoint Services for Report Server Items](../../reporting-services/security/use-built-in-security-in-windows-sharepoint-services-for-report-server-items.md).  
  
 To use the built-in security features, you must assign Windows user or group accounts to the SharePoint groups. Most users must be granted permissions to access the server. The only exceptions are the server administrator and portal site owner who have automatic access to [!INCLUDE[SPF2010](../../includes/spf2010-md.md)] when the software is installed.  
  
## In this section  
 [Use built-in security in Windows SharePoint services for report server items](../../reporting-services/security/use-built-in-security-in-windows-sharepoint-services-for-report-server-items.md)  
 Explains how the predefined SharePoint groups and permission levels can be used to access report server items.  
  
 [SharePoint site and list permission reference for report server items](../../reporting-services/security/sharepoint-site-and-list-permission-reference-for-report-server-items.md)  
 Provides a reference of all SharePoint product permissions that can be used to access report server operations.  
  
 [Set permissions for report server operations in a SharePoint web application](../../reporting-services/security/set-permissions-for-report-server-operations-in-a-sharepoint-web-application.md)  
 Describes permission requirements for unplanned reporting and suggests approaches for making features available.  
  
 [Compare roles and tasks in Reporting Services to SharePoint groups and permissions](../../reporting-services/security/reporting-services-roles-tasks-vs-sharepoint-groups-permissions.md)  
 Provides a brief summary of how the SharePoint groups compare with predefined role definitions in [!INCLUDE[ssRSnoversion](../../includes/ssrsnoversion-md.md)].  
  
 [Set permissions for report server items on a SharePoint site &#40;Reporting Services in SharePoint integrated mode&#41;](../../reporting-services/security/set-permissions-for-report-server-items-on-a-sharepoint-site.md)  
 Provides instructions for creating new SharePoint groups that have permission to start Report Builder and set model item security. This article also contains general guidelines about setting custom permissions for any report server item or operation.  
  
## Related content

- [Reporting Services security and protection](../../reporting-services/security/reporting-services-security-and-protection.md)
