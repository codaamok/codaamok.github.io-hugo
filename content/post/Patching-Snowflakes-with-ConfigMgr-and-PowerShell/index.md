---
title: "Patching Snowflakes With ConfigMgr and PowerShell"
date: 2022-09-17T10:22:56+01:00
draft: true
image: images/cover.jpg
categories:
    - PowerShell
    - ConfigMgr
---

- Do you have servers in your environment that require special or manual patching?
- Do you have non-redundant application servers that must have a co-ordinated patching routine? Wish you used ConfigMgr for these but haven't figured out a way?
- Do you have spaghetti or unfinished code that attempts to orchestrate complicated patching cycles?

By "special patching", I mean the customer or application owners are terrified of the word "patching" and demand it be performed manually on a handful of finicky application servers out of fear of downtime. _(yeah, because hitting 'check for updates' manually reduces risk_ :unamused: _)_

This leaves you feeling frustrated in having to perform tedious manual labour. Login, point, click, sit, wait etc.. You're also annoyed the business has invested in a behemoth like ConfigMgr, and aren't using it to its full potential just for these handful of special "snowflake" servers.

This is where I tell you I wrote a PowerShell module that automates this! Which is funny because you can't automate it! You must do it manually! :laughing: but hang on let me explain..

## PSCMSnowflakePatching

[PSCMSnowflakePatching](https://github.com/codaamok/PSCMSnowflakePatching) is a PowerShell module which contains a few functions but the primary interest is in [`Invoke-CMSnowflakePatching`](https://github.com/codaamok/PSCMSnowflakePatching/blob/main/docs/Invoke-CMSoftwareUpdateInstall.md). 

In this post I want to show you why I think this function will help you patch these snowflake systems without needing to sacrifice the use of ConfigMgr. It can streamline your manual labour, or your existing automation process, or hopefully inspire you to automate a complex patching routine with it.

Using `Invoke-CMSnowflakePatching`, you can either:

- a) give it one or more hostnames via the `-ComputerName` parameter
- b) give it a ConfigMgr device collection ID via the `-CollectionId` parameter
- c) use the `-ChooseCollection` parameter and select a device collection from your environment

For each host, it will remotely start the software update installation for all updates deployed to it. By default it doesn't reboot or make any retry attempts, but there parameters for this if you need it:

- `-AllowReboot` switch will reboot the system(s) if any update returned an exit code indicating a reboot is required
- `-Retry` parameter will let you indicate the maximum number of retries you would like the function to install updates if there was a failure in the previous attempt

All hosts are processed in parallel, and you will get a live progress update from each host as it has finished patching with a break down of what updates were installed, success or failure.

If you use the either the `-ComputerName` or `-CollectionId` parameters, an output object is returned at the end with the result of patching for each host. This is great because if you want to orchestrate a complex patching routine with tools such as System Center Orchestrator or Azure Automation, you can absolutely do this with the feedback from the function.

![Live progress written to console](images/1-1.png) 

In the above screenshot, you can see we're calling `Invoke-CMSnowflakePatching` and giving it a ConfigMgr device collection ID. We're capturing the output of the command by assigning it to the `$result` variable. 

It's worth calling out the two time columns seen on all lines: the first column is the time when that line was written, and the second column is the elapsed time since the start of execution.

Beyond the first few lines revolving around startup, you can see essential information as the jobs finish patching each host. We can see the list of updates that successfully installed, and a yellow text warning indicating one of the updates put the system in a pending reboot state.

If you were running this ad-hoc and interactively, I suspect you might find this realtime information helpful. This output likely can't easily be captured by most automation tools as it's just `Write-Host`. However, this is why an output object is returned instead (see below) - this output can be captured and used however you please.

![Output object in the $result variable](images/1-2.png)

Looking at the above screenshot, this is where more automation possibilities become available for you.

The function returned a summary of the patch jobs for each host as output objects, and we captured this in the `$result` variable from the last screenshot. Here we can see valuable information that might help feed as input to other automation things.

Here is another example, if I ran the following command:

```ps
$result = Invoke-CMSnowflakePatching -ComputerName 'VEEAM' -AllowReboot -Retry 3
```

This time we're just targetting the one server, permitting the server to reboot if an update returned a hard/soft pending reboot, and allow a maximum of 3 retries if there were any installation failures.

At the end of the process, you will receive an below output object similar to the below in the `$result` variable:

```
PS C:\> $result

ComputerName    : VEEAM
Result          : Failure
Updates         : {@{Name=7-Zip 22.01 (MSI-x64); ArticleID=PMPC-2022-07-18; EvaluationState=Error;
                  ErrorCode=2147944003}, @{Name=2022-08 Security Update for Microsoft server operating system version 21H2 for x64-based Systems
                  (KB5012170); ArticleID=5012170; EvaluationState=InstallComplete; ErrorCode=0}, @{Name=2022-08 Cumulative Update for .NET Framework 3.5 and 4.8 for
                  Microsoft server operating system version 21H2 for x64 (KB5015733); ArticleID=5015733; EvaluationState=InstallComplete; ErrorCode=0},
                  @{Name=Microsoft Edge 105.0.1343.33 (x64); ArticleID=PMPC-2022-09-09; EvaluationState=InstallComplete; ErrorCode=0}}
IsPendingReboot : False
NumberOfReboots : 1
NumberOfRetries : 3
RunspaceId      : af37488e-dad9-4d56-b72a-5aa642e589e4
```

From the output above you can see the overall result from patching my Veeam server; the list of updates that were installed, whether there is a pending reboot, how many times the server was rebooted during patching, and how many times it retried.

We can see `NumberOfRetries` is 3, and that `7-Zip 22.01 (MSI-x64)` finished in a state of `Error` - it failed to install the update despite 3 attempts.

It looks like it installed all updates, and likely one of the Windows updates returned a pending reboot so the server rebooted. As the 7-Zip update failed in the previous attempt, it was retried. It retried for a maximum of 3 attempts before returning.

Here is the full content of the `Updates` property from the output object in list view:

```
PS C:\> $result.Updates | fl

Name            : 7-Zip 22.01 (MSI-x64) - (Republished on 2022-09-15 at 17:52)
ArticleID       : PMPC-2022-07-18
EvaluationState : Error
ErrorCode       : 2147944003

Name            : 2022-08 Security Update for Microsoft server operating system version 21H2 for x64-based Systems (KB5012170)
ArticleID       : KB5012170
EvaluationState : InstallComplete
ErrorCode       : 0

Name            : 2022-08 Cumulative Update for .NET Framework 3.5 and 4.8 for Microsoft server operating system version 21H2 for x64 (KB5015733)
ArticleID       : KB5015733
EvaluationState : InstallComplete
ErrorCode       : 0

Name            : Microsoft Edge 105.0.1343.33 (x64)
ArticleID       : PMPC-2022-09-09
EvaluationState : InstallComplete
ErrorCode       : 0
```

With this output, you have a lot of opportunities. For example, you could feed this output to other scripts to:

- Send an email as a report with the result
- Invoke some other custom remedial actions on the server or application it hosts
- Send the data to a ticketing system using some web API, or raise an alert somewhere

Another benefit here is that you are using what you've paid for - ConfigMgr! With this approach, there will be no need to abandon usual deployment, content delivery, and reporting capabilities. The function is installing the software updates deployed to the client by ConfigMgr.

Here are a couple more example screenshots:

> image of failure here  
> image of success and no pending reboot here

## Interatively use PSCMSnowflakePatching with the `-ChooseCollection` Parameter

The `-ChooseCollection` is the only parameter that you can't use in an automated fashion. This is because it produces a pop-up `Out-GridView` window prompting you to choose a ConfigMgr device collection.

This is just demonstrating that you can still use `Invoke-CMSnowflakePatching` to 'manually' patch systems if you wish to, just without the hassle of login, point, click, etc. 

> gif image of `-ChooseCollection` here

## Why a module? Why not just a script?

This could just be my subconscious self playing games in my mind, but I feel the need to justify why this ended up being a module and not a single script file. An irrational pressure builds in my mind when I publish code online - "it must be perfect and justified!"

Jobs are used to process multiple hosts at once for patching, and I found I was re-using code to account for things like loop waiting, timeouts, and retries. You can't easily use nor pass functions to jobs, so instead, for better management and readability, I opted for a module and for each job, I leverage the module for reusing code.

## Why not use a task sequence?

I guess there could be an argument made to use a task sequence if you needed to orchestrate complex patching routines. I wouldn't disagree with the idea, whatever floats your boat. For me personally, code offers more flexibility. 

For instance, in a task sequence I don't think it's trivial to perform actions in a loop on a timer until it eventually succeeds. This sort of logic is littered within `Invoke-CMSnowflakePatching`, with thanks to [Cody Mathis](http://twitter.com/codymathis123) and [`New-LoopAction`](https://github.com/CodyMathis123/CM-Ramblings/blob/master/New-LoopAction.ps1) :heart:

## Why?

{{< unsafe >}}
<iframe src="https://giphy.com/embed/3cXmze4Y8igXdnkc3U" width="480" height="360" frameBorder="0" class="giphy-embed" allowFullScreen style="display: block; margin: 0 auto;"></iframe>
{{< /unsafe >}}

Andrew Porter pinged me asking if I would help making one. He found a few scripts online here and there which collectively sort of did what he wanted but not quite.

These are the scripts which he found:

- Only installs updates, doesn’t reboot or cycle through
https://timmyit.com/2016/08/01/sccm-and-powershell-force-install-of-software-updates-thats-available-on-client-through-wmi/ 
 
- Triggers up0dates remotely and reboots but doesn’t cycle through again or notify results
https://eskonr.com/2021/05/using-scripts-to-trigger-software-updates-remotely-from-the-sccm-console/ 
 
- Install updates, reboot and loops to install further updates – not completely working
https://docs.microsoft.com/en-us/answers/questions/642405/powershell-loop-install-of-available-software-upda.html 
 
- Installs updates on all servers in a collection, reboots and loops to install further updates
https://www.itreliable.com/wp/powershell-script-to-install-software-updates-deployed-by-sccm-and-reboot-the-computer-remotely/

Method 4 seem ideal as its code was more or less complete. I personally wasn't a fan of the style of code, so I decided to rewrite it.
