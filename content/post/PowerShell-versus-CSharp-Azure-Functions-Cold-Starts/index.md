---
title: "PowerShell vs. CSharp Azure Functions Cold Starts"
date: 2021-08-07T18:29:18+01:00
draft: true
image: images/cover.jpg
categories:
    - PowerShell
    - Azure
---

- Introduction / summarise the post
  - What are Azure Functions?
  - What is a "cold start"?
  - Highlight other posts exists on the topic
    - I guess I wanted to see if there were any improvements and I had both working code samples available anyway, figured why not
- Explain the test
  - Share the Function App code
  - Share the testing script
  - Mention it's not a perfect test because I didn't aim for performant and high quality code in either examples
    - Could perhaps maybe some how get the C# code in to PowerShell?
  - Share the results
- Discussion 
  - Mention the shared Function App code includes commented out sections which demonstrates how to add replyTo email addresses to messages
  - Explain I've started learning c#, while learning I struggled with the journey a bit:
    - Completely different style of coding, more OOP
    - A lot of 'scaffolding' work involved to do trivial things that you can otherwise do in PowerShell with minimal effort
    - Like many learning experiences I feel frustrated not being able to be productive, more so with C#
    - My motivation to learn/write more C# drastically changed after seeing performance benefits of this comparison

It has been a while since I've written a post. I've been pretty committed to a new job I started at the start of the year. I need to find more positive outlets and return to doing the things I enjoy in my down-time. I hope to do more tinkering on side projects and giving back to the community.

The cover image for this post if for our pup, Alfie! He's a beagle and we will be bringing him home in a couple of weeks.

# Introduction

In this post I want to show you how PowerShell and C# stack up against each other with their cold start times in Azure Functions.

A quick recap of Azure Functions if you're unsure: 

- Serverless computing ("less servers" &copy; [Jeff Hollan](https://twitter.com/jeffhollan))
- Scalable cloud resources where you can execute given code to perform a task on a schedule or trigger, e.g.:
  - A website backend API that interacts with internal resources or third party APIs
  - Trigger a script to run when a new VM is created in Azure
  - Send an email or Teams notification with a given message fed via HTTP POST request
- If you want to learn with more examples, check out these resources:
  - [Build serverless APIs with Azure Functions | Azure Friday](https://www.youtube.com/watch?v=499iCgNLDDE)
  - [Introduction to Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview)
  - [Intro to Azure Functions - What they are and how to create and deploy them](https://www.youtube.com/watch?v=zIfxkub7CLY)

What is a cold start? A cold start is the process of a function starting up and completing the task. The painful fact here is that if your function is left idle for some period of time, e.g. 15 or 20 minutes, the process of the function is stopped until it is called again. The latency the user experiences calling your function again after it has stopped is high. If the user calls your function again within the idle period, before it is stopped, the latency is low because the function is still running.

You can mitigate cold starts in Azure with [Premium Plan](https://docs.microsoft.com/en-us/azure/azure-functions/functions-premium-plan?tabs=portal#eliminate-cold-starts) features and a [App Service](https://docs.microsoft.com/en-us/azure/azure-functions/dedicated-plan) plan.

# The test

For me to show you how PowerShell and C# perform against each other in Azure Function Apps, I have two HTTP-trigger functions: [GraphAPI-Mail-PS](https://github.com/codaamok/PoSH/tree/master/Azure/AzureFunctions/GraphAPI-Mail-PS) and [GraphAPI-Mail-CSharp](https://github.com/codaamok/PoSH/tree/master/Azure/AzureFunctions/GraphAPI-Mail-CSharp).

When I send a simple HTTP request to either one, they send an email to me, from myself, using Microsoft's Graph API, e.g.:

```powershell
# Calls the C# Function App
Invoke-RestMethod -Uri "https://graphapi-mail-csharp.azurewebsites.net/api/GraphAPI_Mail"

# Calls the PowerShell Function App
Invoke-RestMethod -Uri "https://graphapi-mail-ps.azurewebsites.net/api/GraphAPI-Mail"
```

I intend to run the above, a hundred times for each while recycling the Function App in-between each time to force a cold start, and see what the numbers are. This is what I intend to use to carry out the test: [Compare-PSCSharpGraphAPIMail.ps1](https://github.com/codaamok/PoSH/blob/master/Azure/AzureFunctions/Compare-PSCSharpGraphAPIMail.ps1).

# The results

| Function App | Average (s) | Minimum (s) | Maximum (s) | 
| --- | --- | --- | --- |
| GraphAPI-Mail-CSharp | 1.564564157 | 0.418345 | 3.5649865 |
| GraphAPI-Mail-PS | 18.533579388 | 13.5226983 | 47.0977745 |

# Discussion

If you've been on the fence about trying C# and you consider yourself handy with PowerShell, hopefully this post has given you some motivation to at least give C# some serious thought. 

I have to be honest though, writing code with C# is not as easy as PowerShell. I guess this applies to all new things but I felt a fair bit of frustration while learning C# and trying to apply it. Thankfully I work with some great people that helped me, mainly [Cody](https://twitter.com/CodyMathis123) and a bit of [Ben](https://twitter.com/powers_hell) too!

I immediately sense that there is a lot of 'scaffolding' involved to write in C# compared to PowerShell, however doing this test has given me a good insight and understanding. For an example of an Azure Function App, C# fits in well because it can ran what you want significantly quicker so if you have anything that involve some user experience, it is for sure superior.
