---
title: "Hosting and Protecting Your Own NuGet Feed with ProGet"
date: 2020-11-29T00:00:00+00:00
draft: true
image: images/cover.jpg
categories:
    - PowerShell
---

In this post I will share with you how to install [Inedo's ProGet](https://inedo.com/proget) to host your own NuGet feed. This will let you share your own PowerShell modules and scripts via cmdlets from the PowerShellGet module like `Install-Module`, `Install-Script`, `Find-Module`, `Find-Script` etc.

There are already several posts which detail how to do this, but I could not find one which explained how to add an authentication layer when trying to pull packages. I wanted to authenticate using the `-Credential` parameter for each of the aforementioned cmdlets when pulling modules.

## What is NuGet?

Here I'll quickly breakdown why "NuGet" is a thing. It should give you an insight when trying to understand how or why it is relevant to PowerShell, especially when you're trying to effectively host your own PowerShell Gallery for internal/private consumption.

NuGet is a package management protocol developed by Microsoft. It was a protocol primarily intended for .NET packages on [Nuget.org](https://nuget.org). Developers for .NET used NuGet to pull their project's package dependencies from NuGet.org, much like how PowerShell users use `Install-Module` from the PowerShell Gallery for their scripts. 

NuGet is the binary which is behind the scenes to make commands like `[Install/Find]-[Module/Script]` pull content from the PowerShell Gallery, or other NuGet feeds e.g. your self-hosted one with products lik ProGet.

Microsoft leveraged the NuGet protocol for PowerShell's package/script/module management so they did not have to reinvent the wheel by producing and maintaining another package management system. 

I hope that has given you a little more insight and understanding as to why we see references to NuGet when it comes to PowerShell module/script package management.

I do not yet know any .NET, but I found the below 5 video YouTube series very insightful as it explains how/why it is used for .NET developers. All 5 videos will take about ~30 mins of your time. [Here is the playlist link](https://www.youtube.com/watch?v=WW3bO1lNDmo&list=PLdo4fOcmZ0oVLvfkFk8O9h6v2Dcdh2bh_).

{{< youtube WW3bO1lNDmo >}}

Microsoft also have good documentation laying out what NuGet is and what it does:

- [What is NuGet and what does it do?](https://docs.microsoft.com/en-us/nuget/what-is-nuget)

## Endpoint Dependencies

- PackageManagement and PowerShellGet
- If not already, get your environment using PowerShell 5.1 as it will make your life _alot_ easier. One of the biggest annoyances with older versions of PowerShell, at least this in context, is that they do not let you store multiple versions of a module.

```ps
PS C:\> Register-PackageSource -Name "MyPackageSource" -Location "https://urlhere" -SkipValidate
PS C:\> Register-PSRepository ... -Credential
```
