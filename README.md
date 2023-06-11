# cdnChecker

一款识别域名是否使用cdn的工具



## 背景

红队打点时经常会有收集子域名然后转成ip进而扩展ip段进行脆弱点寻找的需求，如果域名使用cdn，会导致收集错误的ip段，因此我们需要排除cdn来收集更准确的ip地址。

现有的一些识别cdn的工具存在如下问题：

- *仅根据cname或ip范围判断cdn，cname与ip范围不全导致遗漏*

- *输出字段较多，不方便直接与其他工具结合*i



## 使用

速度测试：实际测试13361个域名耗时82s

参数说明

```
Usage of ./cdnChecker:
  -cf string         //cdn cname文件，默认为同目录cdn_cname
        cdn cname file (default "cdn_cname")
  -df string         //域名列表文件，注意为host部分，不要带http://
        domain list file
  -r string       //dns服务器列表文件
        dns resolvers file
```

### 单独使用
__*不建议单独使用，这个 fork 修改版是为了满足服务器 pipeline 使用的*__
```
$ cat /tmp/1
www.baidu.com
www.qq.com
www.alibabagroup.com
aurora.tencent.com
$ cat /tmp/1 | go run cdnChecker.go -df -  
{"Domain":"www.baidu.com","Ip":["182.61.200.7","182.61.200.6"],"Cdn":true,"Cname":["www.a.shifen.com"]}
{"Domain":"aurora.tencent.com","Ip":["43.137.23.148"],"Cdn":false,"Cname":null}
{"Domain":"www.alibabagroup.com","Ip":["43.243.246.103","43.243.246.100","43.243.246.106","43.243.246.101","43.243.246.104","43.243.246.102","43.243.246.105","43.243.246.107"],"Cdn":true,"Cname":["www.alibabagroup.com.w.cdngslb.com"]}
{"Domain":"www.qq.com","Ip":["109.244.211.100","109.244.211.81"],"Cdn":false,"Cname":["ins-r23tsuuf.ias.tencent-cloud.net"]}
```

### pipeline

```
root@nmmd:/whereris/#  cat /tmp/1 | ./cdnChecker -df -   | jq -c 'select(.Cdn==false)' | jq -r  .Domain | ./yscan -ip - -port 8080,443,80 -mode 1 -json ./result.json
INFO[0000] Loading technologies default
SYNSCAINING 100% |████████████████████████████████████████| (78/78, 78 it/s)INFO[0019] 49.234.124.95 443 https Matched  [Nginx PHP]
SYN-TCPSCAINING  21% |████████                                | (3/14, 33 it/min) [2s:19s]INFO[0019] 42.192.184.173 443 https Matched DayBreak [Nginx]
SYN-TCPSCAINING  28% |███████████                             | (4/14, 33 it/min) [2s:18s]INFO[0019] 182.254.150.199 443 https Matched 登录中心 [Bootstrap PHP jQuery Nginx]
SYN-TCPSCAINING  35% |██████████████                          | (5/14, 33 it/min) [2s:16s]INFO[0020] 106.75.231.100 443 https Matched  [Nuxt.js Nginx Node.js Vue.js]
SYN-TCPSCAINING  42% |█████████████████                       | (6/14, 3 it/s) [2s:2s]INFO[0020] 39.105.209.249 443 https Matched 404 Not Found []
SYN-TCPSCAINING  50% |████████████████████                    | (7/14, 3 it/s) [3s:2s]INFO[0021] 182.254.150.199 80 http Matched 403 Forbidden [Nginx]
SYN-TCPSCAINING  57% |██████████████████████                  | (8/14, 3 it/s) [3s:2s]INFO[0021] 117.50.12.71 80 http Matched 403 Forbidden [Nginx]
SYN-TCPSCAINING  64% |█████████████████████████               | (9/14, 3 it/s) [3s:1s]INFO[0021] 42.192.184.173 80 http Matched 403 Forbidden [Nginx]
SYN-TCPSCAINING  71% |████████████████████████████            | (10/14, 3 it/s) [3s:1s]INFO[0021] 49.234.124.95 80 http Matched Welcome to CentOS [Nginx]
SYN-TCPSCAINING  78% |███████████████████████████████         | (11/14, 3 it/s) [3s:1s]INFO[0021] 106.75.231.100 80 http Matched  [Vue.js Nuxt.js Nginx Node.js]
SYN-TCPSCAINING  85% |██████████████████████████████████      | (12/14, 4 it/s) [4s:0s]INFO[0022] 39.105.209.249 80 http Matched 404 Not Found []
SYN-TCPSCAINING  92% |█████████████████████████████████████   | (13/14, 3 it/s) [4s:0s]INFO[0023] 124.70.145.246 80 http Matched APTP-Discovery [Nginx]
SYN-TCPSCAINING 100% |████████████████████████████████████████| (14/14, 2 it/s)INFO[0025] company.tophant.com 443 https Matched 404 Not Found [Nginx]
INFO[0026] account.tophant.com 443 https Matched 登录中心 [Nginx Bootstrap PHP jQuery]
INFO[0026] api.tophant.com 443 https Matched  [Nginx PHP]
INFO[0026] daybreak.tophant.com 443 https Matched DayBreak [Nginx]
INFO[0026] tophant.com 443 https Matched 斗象科技 - 网络安全数据分析与安全运营提供商 [Font Awesome Nginx]
INFO[0026] www.tophant.com 443 https Matched 斗象科技 - 网络安全数据分析与安全运营提供商 [Font Awesome Nginx]
INFO[0026] vip.tophant.com 443 https Matched 漏洞情报中心 [Vue.js Nuxt.js Nginx Node.js]
INFO[0026] pay.tophant.com 443 https Matched  []
INFO[0028] api.tophant.com 80 http Matched Welcome to CentOS [Nginx]
INFO[0028] company.tophant.com 80 http Matched 404 Not Found [Nginx]
INFO[0028] pay.tophant.com 80 http Matched  []
INFO[0028] tophant.com 80 http Matched 斗象科技 - 网络安全数据分析与安全运营提供商 [Font Awesome Nginx]
INFO[0028] account.tophant.com 80 http Matched 登录中心 [Bootstrap PHP jQuery Nginx]
INFO[0028] daybreak.tophant.com 80 http Matched DayBreak [Nginx]
INFO[0028] www.tophant.com 80 http Matched 斗象科技 - 网络安全数据分析与安全运营提供商 [Font Awesome Nginx]
INFO[0029] vip.tophant.com 80 http Matched 漏洞情报中心 [Nuxt.js Nginx Vue.js Node.js]
INFO[0029] discovery.tophant.com 80 http Matched APTP-Discovery [Nginx]
INFO[0031] crs.tophant.com 80 http Matched 斗象智能安全 - 全息安全智能驱动 [Nginx]
INFO[0031] 31.248616064s


root@nmmd:/whereris# cat result.json | jq .[].title
null
"DayBreak"
"登录中心"
null
"404 Not Found"
"403 Forbidden"
"403 Forbidden"
"403 Forbidden"
"Welcome to CentOS"
null
"404 Not Found"
"APTP-Discovery"
"404 Not Found"
"登录中心"
null
"DayBreak"
"斗象科技 - 网络安全数据分析与安全运营提供商"
"斗象科技 - 网络安全数据分析与安全运营提供商"
"漏洞情报中心"
null
"Welcome to CentOS"
"404 Not Found"
null
"斗象科技 - 网络安全数据分析与安全运营提供商"
"登录中心"
"DayBreak"
"斗象科技 - 网络安全数据分析与安全运营提供商"
"漏洞情报中心"
"APTP-Discovery"
"斗象智能安全 - 全息安全智能驱动"
```


结合[mapcidr](https://github.com/projectdiscovery/mapcidr ) 可直接生成ip段

```
$./cdnChecker -df domains.txt -cf cdn_cname -r resolvers.txt | jq -c 'select(.Cdn==false)' | jq -r  .Ip[] | sort | uniq  | mapcidr -aggregate-approx -silent
43.137.23.148/32
```

**强烈推荐dns服务器列表使用自带的resolvers.txt（均为国内dns服务器且验证可用），如果服务器数量过少，大量的dns查询会导致timeout，影响查询准确度**



## 识别cdn思路

主要通过多个dns服务器节点获取域名解析ip，如果存在4个以上不同的ip段，则判断使用cdn，反之未使用cdn。但是直接通过dns服务器查询会增加网络开销影响速度，因此先通过以下方法完成初步筛选：

1.通过https://github.com/projectdiscovery/dnsx 自带的checkCdn方法（通过ip范围判断，主要为国外cdn厂商，对国内cdn识别效果不理想）

2.存在A记录但不存在cname的域名直接判断未使用cdn

3.存在cname的与cdn name列表对比，如果包含cdn cname列表则判断使用cdn



## 常见问题

- 结果中使用cdn域名列表与未使用cdn域名列表数量相加与实际测试域名数量不符？

​       答：对于无法获取解析ip的域名，程序会默认为域名无效过滤掉

- 判定站点使用了cdn，不在cdn的ip范围，不在cdn的cname范围，同时domaininfo文件中对应的也是一个ip

​       答：判定站点是否使用cdn时是参考多个dns服务器解析结果，而写入domaininfo文件获取的ip地址为单个dns服务器解析结果，个别域名会存在多服务器解析多个ip而单服务器仅解析一个ip的情况



## 感谢

https://github.com/xiaoyiios/chinacdndomianlist
