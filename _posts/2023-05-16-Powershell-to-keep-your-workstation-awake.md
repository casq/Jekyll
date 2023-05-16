---
layout: post
title:  "PowerShell command to keep your workstation awake"
date:   2023-05-16 18:55:00 +0100
categories: Powershell
---
A one-liner you can run in a non-admin PowerShell window to keep your workstation awake:

    $limit = (Get-Date).AddMinutes(180); $wsh = New-Object -ComObject WScript.Shell; while ((Get-Date) -le $limit) {write-host "End time is $limit"; write-host "Time now is " (Get-Date); $wsh.SendKeys('+{F15}'); Start-Sleep -seconds 360};