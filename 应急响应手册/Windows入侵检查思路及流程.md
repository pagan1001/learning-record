# ***应急响应操作手册（Windows篇）***</big>

### **什么时候该开展应急响应**

服务器被入侵，业务出现蠕虫事件，用户以及公司员工被钓鱼攻击，业务被 DDoS 攻击，核心业务出现DNS、链路劫持攻击等等

### **如何开展应急响应**

>确定攻击时间<br>
查找攻击线索<br>
梳理攻击流程<br>
实施解决方案<br>
定位攻击者

### **常见应急响应事件分类**

>web入侵：网页挂马、主页篡改、Webshell<br>
系统入侵：病毒木马、勒索软件、远控后门、系统异常、RDP爆破、SSH爆破、主机漏洞、数据库入侵等<br>
网络攻击：DDOS攻击、DNS劫持、ARP欺骗<br>
路由器/交换机异常：内网病毒，配置错误等

# 入侵排查思路

## ***1、检查系统账号安全***

- 查看服务器是否存在可疑账号、新增账号
```
1、打开 cmd 窗口或 Win+R，输入 lusrmgr.msc 命令，查看是否有新增/可疑的账号，如有管理员群组的（Administrators）里的新增账户
```
![alt text](photos/win1.png)

- 查看服务器是否存在隐藏账号、克隆账号
```
1、打开cmd窗口，输入regedit，查看注册表编辑器,观察管理员对应键值；
2、选择 HKEY_LOCAL_MACHINE/SAM/SAM，默认无法查看该选项内容，右键菜单选择权限，打开权限管理窗口；
3、选择当前用户（一般为 administrator），将权限勾选为完全控制，然后确定。关闭注册表编辑器；
4、再次打开注册表编辑器，即可选择HKEY_LOCAL_MACHINE/SAM/SAM/Domains/Account/Users；
5、在 Names 项下可以看到实例所有用户名；
```
![alt text](photos/win2.png)
![alt text](photos/win3.png)

<mark>***注意：如出现本地账户中没有的账户，即为隐藏账户，在确认为非系统用户的前提下，可删除此用户。***</mark>
```
使用D盾_web查杀工具，集成了对克隆账号检测的功能
```

- 查看服务器是否有弱口令，远程管理端口是否对公网开放
```
可咨询服务器相关管理员
```

- 结合日志，查看管理员登录时间、用户名是否存在异常。
```
1、 打开cmd，输入"eventvwr.msc"，回车运行，打开“事件查看器”
2、 导出 Windows 日志 – 安全
3、 用微软官方工具 Log Parser 进行分析 
下载地址：https://www.microsoft.com/enus/download/details.aspx?id=24659
```
![alt text](photos/win4.png)

## ***2、检查端口、进程***
- 检查异常端口

<mark>***是否有远程连接，可疑连接***</mark>
```
1、netstat -ano 查看目前的网络连接，
2、定位可疑的ESTABLISHED：netstat -ano | findstr "ESTABLISHED"
3、根据netstat 定位出的pid，再通过tasklist命令进行进程定位 tasklist | findstr “PID”
4、通过D盾web查杀工具进行端口查看
```

![alt text](photos/win5.png)
![alt text](photos/win6.png)
![alt text](photos/win7.png)

- 检查异常进程
```
1、开始-运行-输入msinfo32，依次点击“软件环境→正在运行任务”就可以查看到进程的详细信息，比如进程路径、进程ID、文件创建日期、启动时间等。
2、打开D盾_web查杀工具，进程查看，关注没有签名信息的进程。
3、通过微软官方提供的 Process Explorer 等工具进行排查 。
```
<mark>***查看可疑进程主要关注点***</mark>
>没有签名验证信息的进程<br>
没有描述信息的进程<br>
进程的属主<br>
进程的路径是否合法<br>
CPU 或内存资源占用长时间过高的进程<br>

## ***3、启动项、计划任务、服务***
## ***4、事件日志***
## ***5、系统相关信息***
## ***6、各中间件/服务器日志默认存放位置***
## ***7、工具进行查杀***