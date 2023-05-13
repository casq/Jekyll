---
layout: post
title:  "Find status of an Office 365 sub-license"
date:   2023-05-13 15:00:00 +0100
categories: PowerShell
---

You may find you need to check the status of an Office 365 sub-license (“service plan” in MS parlance) for a user account.

Recently I had to do this for the phone system sublicense.

This has the shortcode “MCOEV”. You can find a list of other shortcodes at:

> [https://scripting.up-in-the.cloud/licensing/list-of-o365-license-skuids-and-names.html](https://scripting.up-in-the.cloud/licensing/list-of-o365-license-skuids-and-names.html
){:target="_blank"}
 
You will need to amend the AccountSkuID and the sub-license (serviceplan.servicename).
 
The script is:
 
    $user = user.name@domain.com

    # Create a list of licenses assigned to the user
    $licenses = (Get-MsolUser -UserPrincipalName $user).Licenses

    # Create a list of sub-licenses contained within a particular AccountSkuID
    $servicestatus = $licenses[[array]::IndexOf($licenses.AccountSkuID,'domain:ENTERPRISEPREMIUM')].serviceStatus

    # Return the value of the item with the index matching
    write-host $user ": MCOEV status is" $servicestatus[[array]::IndexOf($Servicestatus.serviceplan.servicename,'MCOEV')].provisioningstatus
 
This works because we first cast the list of licenses (AccountSkuIDs), and then the list of sub-licenses (“service plans”) into arrays and then search the arrays for the indexes of the items which match our requested strings - 'domain:ENTERPRISEPREMIUM' and 'MCOEV'.
 
You can find your domain:AccountSkuID with:
 
    $licenses.AccountSkuID