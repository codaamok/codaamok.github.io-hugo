---
title: "ConfigMgr throttling distribution point bandwidth"
date: 2019-02-17T00:00:00+01:00
draft: true
categories:
    - ConfigMgr
---

> :information_source: It looks like as I was writing this Microsoft have just publicly released a former internal document detailing some things that I had claimed to be undocumented [click here](https://support.microsoft.com/en-us/help/4482728/understand-troubleshoot-content-distribution-in-configuration-manager)
> {{< tweet 1092469116349440003 >}}

In this post I'll cover ways you can control bandwidth for Configuration Manager using the rate limit config available for each distribution point. Outside of rate limits, a modern approach to traffic shaping content distribution to DPs is LEDBAT. [Read this great write up](https://deploymentresearch.com/Research/Post/657/LEDBAT-with-ConfigMgr-Pure-Love-By-Daniel-Olsson] by [Daniel Olsson](https://deploymentresearch.com/Research/Post/657/LEDBAT-with-ConfigMgr-Pure-Love-By-Daniel-Olsson) for more LEDBAT info in this use case.

![Distribution Point rate limits](images/configmgr-throttling-distribution-point-bandwidth-01.jpg)
