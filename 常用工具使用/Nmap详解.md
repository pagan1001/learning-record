# Nmap 端口扫描从入门到精通
![alt text](photos/Nmap-logo.png)

## 如何让nmap在速度上远超其他工具，同时又不牺牲扫描的质量
### <mark>***高效扫描策略***</mark> 
在进行安全测试时，我们通常面对两种情况：针对单一目标的深入测试，或是对多个目标进行广泛扫描。今天，主要介绍后者。

首先，我们需要准备好目标列表。可以将目标分为两类：
- ips.txt：包含IP地址和IP范围
- hosts.txt：包含主机名，主要是网站的主页或www子域名

扫描主机名对于hosts.txt中的目标，我们可以使用以下nmap命令：
```shell
nmap -iL hosts.txt -Pn --min-rate 5000 --max-retries 1 --max-scan-delay 20ms -T4 --top-ports 1000 --exclude-ports 22,80,443,53,5060,8080 --open -oX nmap.xml
```

<big> **命令解析**</big>

:pushpin: -iL hosts.txt：从文件读取目标列表<br>
:pushpin: -Pn：跳过主机发现，提高扫描速度<br>
:pushpin: --min-rate 5000：每秒至少发送5000个数据包，加快扫描速度<br>
:pushpin: --max-retries 1：最多重试一次，避免过多等待<br>
:pushpin: --max-scan-delay 20ms：控制扫描间隔，平衡速度和准确性<br>
:pushpin: -T4：使用较激进的时间模板，但不至于太过激进<br>
:pushpin: --top-ports 1000：扫描最常见的1000个端口<br>
:pushpin: --exclude-ports 22,80,443,53,5060,8080：排除一些常见端口，避免重复工作<br>
:pushpin: --open：只显示开放的端口<br>
:pushpin: -oX nmap.xml：将结果保存为XML格式，便于后续处理<br>

使用这个命令，我们可以在短时间内完成大量主机的扫描。例如，500个主机只需要约6分钟。<br>

### <mark>***结果处理***</mark>

获得XML格式的扫描结果后，我们需要将其转换为更易于处理的格式。使用此脚本，可以将XML结果转换为IP:PORT格式：
```shell
//源码
#!/bin/bash

# Check if an argument was provided
if [ $# -eq 0 ]; then
    NMAP_XML_OUTPUT="/dev/stdin"
else
    NMAP_XML_OUTPUT="$1"
fi

# Use xmllint to parse IP addresses and ports from the Nmap XML output
xmllint --xpath '//host[status/@state="up"]/address[@addrtype="ipv4"]/@addr' $NMAP_XML_OUTPUT | \
sed 's/ addr="/\n/g' | sed 's/"//g' | grep -v '^$' | while read IP; do
    xmllint --xpath "//host[address/@addr=\"$IP\"]/ports/port[state/@state='open']/@portid" $NMAP_XML_OUTPUT | \
    sed 's/ portid="/\n/g' | sed 's/"//g' | grep -v '^$' | while read PORT; do
        echo "$IP:$PORT"
    done
done

//使用命令
curl -s 'https://gist.githubusercontent.com/ott3rly/7bd162b1f2de4dcf3d65de07a530a326/raw/83c68d246b857dcf04d88c2db9d54b4b8a4c885a/nmap-xml-to-httpx.sh' | bash -s - nmap.xml
```

此脚本会输出类似以下格式的结果：
```
123.321.22.100:1333
13.32.222.13:1334
123.31.222.103:1335
```

你还可以将结果传递给httpx，进一步筛选出可访问的服务：
```
curl -s 'https://gist.githubusercontent.com/ott3rly/7bd162b1f2de4dcf3d65de07a530a326/raw/83c68d246b857dcf04d88c2db9d54b4b8a4c885a/nmap-xml-to-httpx.sh' | bash -s - nmap.xml | httpx -mc 200
```

### <mark>***扫描IP范围***</mark>

对于ips.txt中的目标，我们可以使用类似的命令：
```
nmap -iL ips.txt -Pn --min-rate 5000 --max-retries 1 --max-scan-delay 20ms -T4 --top-ports 1000 --exclude-ports 22,80,443,53,5060,8080 --open -oX nmap2.xml
```

这个过程可能会比较耗时，特别是当目标IP数量众多时。建议在空闲时间运行这种大规模扫描。

### <mark>***进阶技巧***</mark>

**1、避免使用云服务器进行端口扫描：** 许多云服务提供商禁止使用他们的服务器进行端口扫描。违反这一规定可能导致账户被封锁。建议在本地计算机上进行端口扫描，特别是对重要目标进行扫描时。

**2、使用cron作业自动化扫描：** 对于需要定期扫描的目标，可以使用cron作业来自动化这个过程。例如，要每天中午12点运行扫描，可以在crontab中添加以下内容：
```shell
0 12 * * * /path/to/nmap/command
```
**3、大规模快速扫描：** 如果需要快速扫描大量IP，可以考虑使用jfscan工具。它结合了nmap和masscan的优点，能够在短时间内完成大规模扫描，但可能会牺牲一些覆盖范围。

### <mark>***结语***</mark>
掌握高效的端口扫描技巧，能让你在安全测试中事半功倍。通过合理配置nmap参数，结合自动化脚本和定时任务，你可以构建一个强大的端口扫描体系，为后续的渗透测试和安全评估打下坚实基础。<br>
工具只是辅助，真正的高手还需要深厚的网络安全知识和丰富的实战经验。