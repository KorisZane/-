## 远程执行漏洞自检（代码执行等）

Nessus号称是世界上最流行的漏洞扫描程序，全世界有超过75000个组织在使用它。该工具提供完整的电脑漏洞扫描服务，并随时更新其漏洞数据库。Nessus不同于传统的漏洞扫描软件，Nessus可同时在本机或远端上遥控，进行系统的漏洞分析扫描

安装参考 https://mp.weixin.qq.com/s/LFojXu3PThJlqAXMgNS6cw

Nexpose是Rapid7出品一款著名、极佳的商业漏洞扫描工具。跟一般的扫描工具不同，Nexpose自身的功能非常强大，可以更新其漏洞数据库，以保证最新的漏洞被扫描到。漏洞扫描效率非常高，对于大型复杂网络，可优先考虑使用；对于大型复杂网络，可以优先考虑使用。可以给出哪些漏洞可以被 Metasploit Exploit，哪些漏洞在 Exploit-db里面有exploit的方案。可以生成非常详细的，非常强大的Report，涵盖了很多统计功能和漏洞的详细信息。虽然没有Web应用程序扫描，但Nexpose涵盖自动漏洞更新以及补丁星期二漏洞更新。

安装参考 [https://www.rapid7.com/products/insightvm/download/](https://www.rapid7.com/products/insightvm/download/)

搭建启动失败切换jdk版本（课程演示为环境变量设置的jdk17）

Goby是一款新的网络安全测试工具，由赵武Zwell（Pangolin、JSky、FOFA作者）打造，它能够针对一个目标企业梳理最全的攻击面信息，同时能进行高效、实战化漏洞扫描，并快速的从一个验证入口点，切换到横向。能通过智能自动化方式，帮助安全入门者熟悉靶场攻防，帮助攻防服务者、渗透人员更快的拿下目标

## 本地执行漏洞自检（溢出提权等）

Windows：
https://i.hacking8.com/tiquan（已关）
https://github.com/bitsadmin/wesng
Linux：
https://github.com/mzet-/linux-exploit-suggester
POC&CVE：
https://github.com/1nnocent1/PoC-in-GitHub

Win10-漏洞提权-CVE-2021-1732
https://github.com/KaLendsi/CVE-2021-1732-Exploit
Win08-漏洞提权-CVE-2019-1458
https://github.com/rip1s/CVE-2019-1458
Linux-漏洞提权-CVE-XXXX-XXXX
https://github.com/mzet-/linux-exploit-suggester

```
msconsole
use exploit/muliti/handler
search ms17_010_eternablue
use exploit/windows/smb/ms17_010_eternablue
set rohost 目标地址
run
```

偏向于内网不联网的机器

```
sysinfo > system.txt
python wes.py system.txt
```