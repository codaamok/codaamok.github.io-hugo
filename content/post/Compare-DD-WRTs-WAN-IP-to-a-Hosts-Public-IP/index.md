---
title: "Compare DD-WRT WAN IP to a Host public IP"
date: 2018-10-27T00:00:00+01:00
draft: false
image: images/cover.jpg
categories:
    - Linux
---

For my home lab on a particular VM that runs Ubuntu Server, I'm interested to know the public IP of my LAN's gateway and the VM's gateway (as it's always connected to VPN). Below is a script I wrote to pull the WAN IP from my DD-WRT router and compare that to VM's gateway public IP.

My copy does other things like start/stop services, configure local applications in my home lab as well as use Pushbullet to notify me if certain conditions are met. This should get you going, hopefully it helps someone.

```bash
#!/bin/bash

x=$(wget "http://192.168.0.1/Status_Internet.live.asp" --user="USERNAME" --password="PASSWORD" -q -O - | sed -e 's/}/\n/g' -e 's/{//g')
declare -A results
while read line; do
        IFS='::' read -ra array <<< "$line"
        results[${array[0]}]=${array[2]}
done<<<"$x"

wanip=${results["wan_ipaddr"]}
vmip=$(wget -q -O - https://checkip.amazonaws.com)

echo $0': WAN IP: '$wanip
echo $0': VM IP: '$vmip
```
