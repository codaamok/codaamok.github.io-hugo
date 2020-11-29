---
title: "Hosting and Protecting Your Own NuGet Feed with ProGet"
date: 2020-11-29T00:00:00+00:00
draft: true
image: images/cover.jpg
categories:
    - PowerShell
---

In this post I will share with you how to install [Inedo's ProGet](https://inedo.com/proget) to host your own NuGet feed. This will let you share your own PowerShell modules and scripts via cmdlets from the PowerShellGet module like `Install-Module`, `Install-Script`, `Find-Module`, `Find-Script` etc.

There are already several posts which detail how to do this, but I could not find one which explained how to configure username/password authentication for the feeds. I wanted to add an authentication later using the `-Credential` parameter for each of the aforementioned cmdlets when pulling modules.

## What is NuGet?

Here I'll quickly breakdown why "NuGet" is a thing. It should give you an insight when trying to understand how or why it is relevant to PowerShell, especially when you're trying to effectively host your own PowerShell Gallery for internal/private consumption.

NuGet is a package management protocol developed by Microsoft. It's a protocol primarily for .NET packages on [Nuget.Org](https://nuget.org). .NET developers use NuGet with NuGet.org much like how PowerShell users use `Install-Module` from the PowerShell Gallery. However Nuget is also the binary that is behind the scenes to make commands like `[Install/Find]-[Module/Script]` pull content from the PowerShell Gallery, or other NuGet feeds e.g. your self-hosted one with products lik ProGet.

Microsoft leveraged the NuGet protocol for PowerShell's package/script/module management so they did not have to reinvent the wheel and produce a different package management system for another product. That's why you see references of NuGet when it comes to PowerShell module/script package management.

I do not yet know any .NET, but I found the below 5 video YouTube series very insightful as it explains how/why it is used for .NET developers. All 5 videos will take about ~30 mins of your time. [Here is the playlist link](https://www.youtube.com/watch?v=WW3bO1lNDmo&list=PLdo4fOcmZ0oVLvfkFk8O9h6v2Dcdh2bh_).

{{< youtube WW3bO1lNDmo >}}

Microsoft also have good documentation laying out what NuGet is and what it does:

- [What is NuGet and what does it do?](https://docs.microsoft.com/en-us/nuget/what-is-nuget)