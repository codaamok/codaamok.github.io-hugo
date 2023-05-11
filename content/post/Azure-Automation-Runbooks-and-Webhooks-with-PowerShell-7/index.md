---
title: "Azure Automation Runbooks and Webhooks With PowerShell 7"
date: 2023-05-11T23:07:03+01:00
draft: true
image: images/cover.jpg
categories:
---

I want to walk you through starting a PowerShell 7.x Azure Automation runbook via webhook. There are some significant limitations with PowerShell 7 in Azure Automation runbooks today, and starting one via webhook (with payload data in the body of the request) is one of them.

> _"When you start PowerShell 7 runbook using the webhook, it auto-converts the webhook input parameter to an invalid JSON."_

Read more here: [Azure Automation runbook types | Limitations and Known issues](https://learn.microsoft.com/en-us/azure/automation/automation-runbook-types?tabs=lps71%2Cpy27#limitations-and-known-issues)

# What is Azure Automation?

There is no shortage of material online and this post isn't an introduction, but if you're unsure, here's a ChatGPT special: 

> Azure Automation is a cloud-based automation and orchestration service that helps organizations streamline and manage repetitive, manual tasks and processes.

In practical terms, especially in the context of PowerShell; you can start your PowerShell script in an Azure sandbox or a [hybrid runbook worker](https://learn.microsoft.com/en-gb/azure/automation/automation-hybrid-runbook-worker). Hybrid runbook workers are great because you can start a script on an Azure VM (Windows or Linux) in your subscription, or perhaps better yet, on an on-premises (non-Azure) server via [Azure Arc](https://learn.microsoft.com/en-gb/azure/azure-arc/servers/overview)!

# Webhook data as invalid JSON

One of the greatest things about runbooks is that you can start them via webhook, and you can pass parameters to your runbook scripts by putting data in the body of your HTTP POST request.

Ordinarily with code like below it's a piece of cake yadeyada:

```powershell

```
