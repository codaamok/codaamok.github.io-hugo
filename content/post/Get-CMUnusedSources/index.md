---
title: "Get CMUnusedSources"
date: 2019-07-07T00:00:00+01:00
draft: false
categories:
    - ConfigMgr
    - PowerShell
---

I recently wrote a PowerShell script that reports on what folders are used or unused by System Center Configuration Manager. 

[You can find it here on GitHub](https://github.com/codaamok/Get-CMUnusedSources).

I demoed the script at the [London Windows Manager User Group](https://wmug.co.uk/), you can watch the recording below on YouTube.

{{< youtube YGwQIUhYJsY >}}

Among a bunch of technical topics, I also learned a huge amount of valuable soft/personal things too. 

## Enough is enough

I started writing in February and since then I have refactored the script and the idea itself many times. It's now start of July and my girlfriend and I want to do fun summer things.

I first "finished" it in April but it was only good if you stored your content sources on a local file system to a site server. I realised that a lot of admins store their sources on a remote file server. I tried to "drop in" accommodations for that but quickly accepted that I was creating fragile flows and hard to read solutions. This is one of the few times I did a "sod it, let's start again".

Around end of May I had a working script that worked with local and remote storage but it was slow. Because it iterates over all gathered folders, and for each folder, iterates over all ConfigMgr content objects, this meant in an environment with 10k content objects and 70k folders it took over 10 hours. I did my best to create efficiencies, such as to skip sub folders of folders already marked as "Not used" but 10 hours really wasn't good.

I was almost at a cross roads because I thought performance shouldn't matter too much because this would be something that you would run once in a blue moon. Then I had that fear of posting online and being bombarded with "you should have done x instead of y". I'm still at risk of that now (which I welcome by the way, especially for something I'm not aware of). It was that thought which pushed me to strive for better because I knew I could improve, and the improvement would be significant enough.

Then I discovered runspaces! Sure, I could have used jobs or set a dependency to use the PoshRSJobs module, but I just wanted to learn runspaces, how to use them and not have a dependency on another module. So of course doing that added more time and testing. It was worth it though.

Throughout the entire time I kept tricking myself in to thinking "I'm almost done", "it'll be done next week", "it's pretty much there". Constant ideas and learning new techniques for readability / performance was another time killer.

I once read that managers hate dealing with sysadmins because when tasked to create something they don't lay the foundations first and get too hung up on the features. I sometimes saw myself as that guy.

With that said, I also felt like I had a short fuse with it and had a _just want to get it done_ mindset. While working on this I started reading (and still yet to finish) [Zen and the Art of Motorcycle Maintenance](https://www.goodreads.com/book/show/629.Zen_and_the_Art_of_Motorcycle_Maintenance). In the early chapters there was emphasis on people rushing their day-to-day and the work they do. I related to this and was guilty of that at times with this script. So then I decided if I'm not taking my time to do it carefully then there's no point doing it at all. 

At this point, summer had started, and for me that means cricket season was on and while working during the week, that meant every Saturday was spent all day playing cricket. I just wanted to spend every free hour I had finishing this but conscious of not wanting to rush it.

I had to draw a line in the sand and make my primary goal to finish it with what I had. I wanted to spend time with my girlfriend, finish that book, learn something else, fix that broken window motor on my car I've had for over a year, play a few nights of CS:GO.

I really enjoyed the brain teasers though. I would be drawing what the flow of some loops looked like in my head while in the shower or driving. Realistically, I know I'm never finished. No doubt if someone found an issue with it I'll be curious and want to fix it. 

I think next time I have some idea that I think will consume any amount of time, I should first spend time thinking about what the end product looks like. Then I should be able to know at what point I would want to bring the idea/project to a close.

## Asking for help

A buddy [Chris Kibble](http://www.christopherkibble.com) shared with me [How to ask questions the smart way](http://www.catb.org/~esr/faqs/smart-questions.html). Looks like this is a link from the [The XY Problem](http://xyproblem.info/).

These two resources are huge and you may not yet appreciate why. I've been on the Internet for as long as I can remember and have been posting stupid questions for most of my life.

When I became mindful of the XY problem and how to ask for help from people who are insanely good, I became much more effective. A part of asking for help is about first having a bloody good stab at it yourself. This often pushed me to not necessarily find the answers but to help me think more inquisitively. I found as a result of asking myself more questions, it enabled me to understand problems better and sometimes find the answer.

I'm in the [Windows Admins](https://winadmins.io) Discord and some people in there are not only clever but incredibly helpful. Just to say thanks to people who helped me:
- Cody Mathis ([@codymathis123](https://github.com/codymathis123))
- Chris Kibble ([@ChrisKibble](https://github.com/ChrisKibble))
- Chris Dent ([@idented-automation](https://github.com/idented-automation))
- Kevin Crouch ([@PsychoData](httos://github.com/PsychoData))
- Patrick (the guy who wrote [MakeMeAdmin](https://github.com/pseymour/MakeMeAdmin/wiki))

## PowerShell

So I wrote a PowerShell script and I learnt some PowerShell. Of course. Well here's what I learnt.

## Debugger

VSCode is my go-to editor. One of the many reasons why I like it, is because of the debugger with the integrated console. Once I read [Life after Write-Debug](https://foxdeploy.com/2019/02/07/life-after-write-debug/) by Stephen Owen I became much more efficient at troubleshooting my dodgy code.

My first manager as an IT professional told me what the [rubber duck debugging technique was](https://en.wikipedia.org/wiki/Rubber_duck_debugging). So when you discover something like debugging for PowerShell in ISE or VSCode, it's a huge sense of relief. The days of intensely reading line by line or including a bunch of print / Write-Host commands are long gone.

[You can also debug other running PowerShell processes!!](https://community.idera.com/database-tools/powershell/powertips/b/tips/posts/debugging-other-powershell-processes)

## Standards and formatting

For quick scripts with limited audience, I make some effort to format and make it at least somewhat presentable for next the guy. However taking on something fairly big that might be used and read by the Internet, I appreciated the need for not necessarily pretty looking code because that's always subjective but at least adopt a standard.

Again, kind of tapping in to that fear of posting online and being scrutinized.

## .NET

I had always known you could do .NET stuff within PowerShell and whenever I saw it in conversation it was always in the context of scale or performance. I figured with what I was trying to achieve, this would fall under the scale category. I've seen environments with tens of thousands of ConfigMgr content objects and hundreds of thousands of folders.

There's a need here as well to determine when to draw the line. You could spend a long time looking up .NET classes and methods when you could save yourself a lot of time just using PowerShell cmdlets. Sure, there's a performance gain, but as a sysadmin I think readability and time is more valuable. So I won't make a habit of using them at every opportunity, purely because I'm not fluent enough. Being mindful of them and how or when to use them is good enough.

## EnumerateDirectories()

On the topic of .NET, I did really want to use the [EnumerateDirectories method from the Directory class](https://docs.microsoft.com/en-us/dotnet/api/system.io.directory.enumeratedirectories). But it falls short where you can't control its on-error preference. No matter what, on error, it's a terminating error. Which sucked because it sounded perfect for what I wanted and crazy fast too.

In situations where it encountered a folder with access denied, it wouldn't continue.

## Runspaces

As I mentioned already I wanted concurrent processing so I had several options already available like jobs or the PoSHRSJobs module that gives you runspaces but with the familiar syntax as jobs. I opted to use runspaces and spend extra time learning more about how they work for two reasons mainly:

- To use something and make a good effort at actually trying to understand it.
- Not set too many dependencies on modules. I had a train of thought that not all ConfigMgr admins are full time admins, no doubt they juggle other stuff too. What if they really wanted to just clean up their storage but wasn't 100% sure how to install a third party module just to get the basics working?

## Collections the += operator

I saw [this thread on Twitter](https://twitter.com/AndySvints/status/1130135528815439874) and my mind was blown how much quicker it was to not use the += operator when adding new elements to an array.

## Join-Path

I guess I saw some quirks of PowerShell along the way but the biggest for me that sticks out in memory was Join-Path. Check out this madness:

```
PS C:\> Test-Path "\\fakeserver\fakeshare\fakefolder"
False
PS C:\> Join-Path -Path "\\fakeserver" -ChildPath "fakeshare" | Join-Path -ChildPath "fakefolder"
\\fakeserver\fakeshare\fakefolder
```

Join-Path copes well with paths that don't exist. Awesome.

```
PS C:\> Test-Path "K:\I\Do\Not\Have\K\Drive\Mapped"
False
PS C:\> Join-Path -Path "K:\" -ChildPath "I\Do\Not\Have\K\Drive\Mapped"
Join-Path : Cannot find drive. A drive with the name 'K' does not exist.
At line:1 char:1
+ Join-Path -Path "K:\" -ChildPath "I\Do\Not\Have\K\Drive\Mapped"
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (K:String) [Join-Path], DriveNotFoundException
    + FullyQualifiedErrorId : DriveNotFound,Microsoft.PowerShell.Commands.JoinPathCommand
```

...unless it's a local path on a drive letter that doesn't exist. Worth remembering. You can't really say Join-Path validates path as it joins them, because it doesn't, nor can you say its sole purpose is to verify just "syntax" or "validity" of path structure, because it isn't.

Looks like there's [an open issue on this](https://github.com/PowerShell/PowerShell/issues/4386).

## Regex

Early doors I was trying to grab the server, share and remainder share from a string: `\\server\share\folder\folder`.

[Cody Mathis](https://twitter.com/CodyMathis123) threw me a little snippet of regex and from then on my eyes were opened. I'm even thinking of going back to a previous project [PoSH Hyper Cloud](https://github.com/codaamok/PoSHHyperCloud) to use more regex and maybe even jobs! [regex101.com](http://regex101.com) is my go to for testing expressions.

## Enumerations

Patrick Seymour recently wrote Test-FileSystemAccess and I enhanced it a little to return exit codes and determine if elevation is required. Some feedback by Chris Dent taught me about enums! They're awesome. An excellent way to create a relationship between numbers and strings. I used the below resources to implement an enum in to Test-FileSystemAccess:

- [Working with enums in PowerShell](https://blog.pauby.com/post/working-with-enums-in-powershell)
- [Creating enums in PowerShell](https://blog.pauby.com/post/creating-enums-in-powershell/)

Tip! If you want to get the full Parameter Type of a parameter that's an enum, so you can use GetNames:: on it, the below will help:

```powershell
(Get-Command Set-ExecutionPolicy).Parameters['ExecutionPolicy'] | Select -ExpandProperty ParameterType
```

## DataTables and filtering

I recently tackled an issue with the script where collections of large scale and filtering them was an issue. Chris Kibble once showed me DataTables and I remember a remark about them being quick. He wasn't joking! Checkout the below!

```powershell
$winFiles = Get-ChildItem c:\windows -Recurse -ErrorAction SilentlyContinue

$commands = @{
    'Where-Object' = {
        $exeFiles = $winFiles | Where-Object { $_.Extension -eq ".exe" }
    }
    'Where-Object (no script block)' = {
        $exeFiles = $winFiles | Where-Object Extension -eq ".exe"
    }
    '.Where' = {
        $exeFiles = $winFiles.Where{ $_.Extension -eq ".exe" }
    }
    'DataTable' = {
        $fileTable = New-Object System.Data.DataTable
        $fileTable.TableName = "AllMuhFiles"

        [void]$fileTable.Columns.Add("FileName")
        [void]$fileTable.Columns.Add("Parent")
        [void]$fileTable.Columns.Add("Extension")
        [void]$fileTable.Columns.Add("IsInf")

        ForEach($file in $winFiles) {
            [void]$fileTable.Rows.Add($file.Name, $file.Parent, $file.Extension)
        }

        $exeFiles = $fileTable.Select("Extension like '.exe'")
    }
    'foreach' = {
        $exefiles = foreach ($file in $winFiles) {
            if ($file.Extension -eq '.exe') {
                $file
            }
        }
    }
}

$commands.Keys | ForEach-Object {
    $testName = $_
    1..3 | ForEach-Object {
        [PSCustomObject]@{
            TestName  = $testName
            Attempt   = $_
            ElapsedMS = (Measure-Command -Expression $commands[$testName]).TotalMilliseconds
        }
    }
}
```

Results:

```
Testing Where-Object
2849.362
2926.2761
2876.9806

Testing .Where
1045.712
1037.4888
1031.191

Testing DataTable select
280.6466
272.8503
272.5341
```

I started looking in to DataTables because on large collections (hundreds of thousands sort of size) `Where-Object` was slow. While the `Where()` method was faster, both were nothing compared to DataTables.

With that said, if a cmdlet has a -Filter parameter available to let you work directly with a provider, then use that. "Keep left" or "filter left, format right" are phrases I learnt and try to keep in mind when filtering.

I abandoned the use of DataTables in the end as I didn't need it. Even though they need additional set up, as you can see from the previous snippet, the results might be worth it in some cases.

Resources I used to learn about filtering and the performance impact

- [Explains where-object and where, and the -filter for provider specific stuff](https://4sysops.com/archives/where-object-vs-the-where-method-array-filtering-in-powershell/)
- [Explains the object type returned by where-object and where](https://stackoverflow.com/questions/50956909/powershell-where-object-vs-where-method)
- [Talks about "keeping left" for best performance. -Filter is king if available but only if you don't want all the objects in an array for later consumption](https://www.jonathanmedd.net/2017/05/powershell-where-where-or-where.html)
