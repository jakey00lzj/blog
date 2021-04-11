+++
Description = "windows10 wsl2 网络互通"
Tags = ['windows10','wsl2','linux']
date = "2021-03-14T21:28:21+08:00"
title = "windows10 wsl2 网络互通设置"
authors = ["jakey"]

+++

wsl2 开的ubuntu 如何与宿主机window10 相互访问？

<!--more-->

1. 从 Windows访问 wsl2 

无需配置，直接按 localhost 访问即可， 比如 wsl2 上起了一个 8080 服务，那么windows 上可直接访问 localhost:8080

2. 从 wsl2 访问 windows 

需要几步，

第一步： windows 上面通过 powershell 添加WSL网卡的防火墙策略

```powershell
New-NetFirewallRule -DisplayName "WSL" -Direction Inbound -InterfaceAlias "vEthernet (WSL)" -Action Allow
```

第二步：在wsl2 调用的对应windows上的程序，在”高级安全 Windows Defender 防火墙中选择入站规则“ 里面找到 公用 网络的规则配置添加信任信息，可以是wsl2的ip段，端口等等
注：wsl2的网络在windows防火墙策略里面算是公用网络！

第三步：wsl2 选取的windows 对应ip 可以用 /etc/resolv.conf 里面获悉