---
title: "Blank InstalledOn Property for Get-HotFix and Win32_QuickFixEngineering"
date: 2019-02-18T00:00:00+01:00
draft: false
image: images/cover.jpg
categories:
    - PowerShell
---

Want to get Windows Update status on a bunch of machines, only to discover the InstalledOn property is blank for lots of updates when you're using Get-HotFix or querying Win32_QuickFixEngineering class with Get-WmiObject?

Turns out it's related to PowerShell parsing the date as string to datetime object incorrectly, depending what your system locale is.

```powershell
$Updates = Get-HotFix | Select-Object description,hotfixid,installedby,@{l="InstalledOn";e={[DateTime]::Parse($_.psbase.properties["installedon"].value,$([System.Globalization.CultureInfo]::GetCultureInfo("en-US")))}}
$Updates | Sort InstalledOn -Descending | Select -First 10
```

I like the above approach, [as posted by user pyro3113](https://community.idera.com/database-tools/powershell/ask_the_experts/f/learn_powershell_from_don_jones-24/12409/get-hotfix-and-get-wmiobject-win32_quickfixengineering-missing-installedon-property/21923#21923), because the select expression lets you easily query remote systems using Get-HotFix or Get-WmiObject.

These two solutions use the Microsoft.Update.Session COM object. The Windows Update PowerShell Module creates the COM object on a remote host whereas the post by britv8 shows just enough to get you going executing on local host:

- [Windows Update PowerShell Module](https://gallery.technet.microsoft.com/scriptcenter/2d191bcd-3308-4edd-9de2-88dff796b0bc)
- [Getting Windows Updates installation history](https://community.spiceworks.com/topic/1461956-get-hotfix-installedon-date-missing) shared by Spiceworks member britv8

Interestingly, using the below command in cmd doesn't suffer from this data type parsing issue. It's just spat out as is with no interpretation.

```
wmic qfe
```
