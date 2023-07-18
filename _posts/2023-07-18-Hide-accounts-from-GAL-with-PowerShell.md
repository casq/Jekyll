---
layout: post
title:  "Hide accounts from GAL with PowerShell"
date:   2023-07-18 18:00:00 +0100
categories: PowerShell ActiveDirectory
---

I have a number of test accounts which were assigned O365 licenses and subsequently appeared in the GAL. I didn't want them cluttering up the GAL but didnt want to go through each account manually.

Import the ActiveDirectory PS module:

	Import-Module activedirectory

To find the status of an account's msExchHideFromAddressLists, you can use:

	Get-ADUser -Identity "ado-01-test" -property msExchHideFromAddressLists

but if the account has not had a value set, nothing will be returned for "msExchHideFromAddressLists".

If you have a number of accounts to amend, you can pattern match for account attributes:

	Get-ADUser -Filter ("DisplayName -like '*-test*'") | `  
    Select Name, SamAccountName, UserPrincipalName | `  
    Export-csv C:\Users\andy\Desktop\userlist.csv -NoTypeInformation

This will create a list of all users who have "-test" in their display name and export it to a csv file which you can manually check.

To import the checked csv:

    $users = Import-csv -Path C:\Users\andy\Desktop\userlist.csv

To set multiple accounts' msExchHideFromAddressLists to true:

    foreach ($user in $users){
    Get-ADuser -Identity $user.SamAccountName -property msExchHideFromAddressLists   
    Set-ADObject -Replace @{msExchHideFromAddressLists=$true}
    }

To check the amended values, run:

	foreach ($user in $users){
		Get-ADuser -Identity $user.SamAccountName -property msExchHideFromAddressLists
	}

and you will see something like:

DistinguishedName          : CN=ado-01-test,OU=Testing,DC=MyDomain,DC=Test  
Enabled                    : True  
msExchHideFromAddressLists : True  
Name                       : ado-01-test  
ObjectClass                : user  
SamAccountName             : ado-01-test  
UserPrincipalName          : ado-01-test@mydomain.test  
