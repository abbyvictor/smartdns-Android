# SmartDNS

![SmartDNS](https://raw.github.com/pymumu/smartdns/master/doc/smartdns-banner.png)  
SmartDNS是一个运行在本地的DNS服务器，SmartDNS接受本地客户端的DNS查询请求，从多个上游DNS服务器获取DNS查询结果，并将访问速度最快的结果返回给客户端，避免DNS污染，提高网络访问速度。
同时支持指定特定域名IP地址，并高性匹配，达到过滤广告的效果。  
与dnsmasq的all-servers不同，smartdns返回的是访问速度最快的解析结果。

详情请看[官方文档](https://github.com/pymumu/smartdns/blob/master/ReadMe.md)

## 特性

1. **多DNS上游服务器**  
   支持配置多个上游DNS服务器，并同时进行查询，即使其中有DNS服务器异常，也不会影响查询。  

1. **返回最快IP地址**  
   支持从域名所属IP地址列表中查找到访问速度最快的IP地址，并返回给客户端，避免DNS污染，提高网络访问速度。

1. **支持多种查询协议**  
   支持UDP，TCP，TLS, HTTPS查询，以及非53端口查询，有效避免DNS污染。

1. **特定域名IP地址指定**  
   支持指定域名的IP地址，达到广告过滤效果，避免恶意网站的效果。

1. **域名高性能后缀匹配**  
   支持域名后缀匹配模式，简化过滤配置，过滤20万条记录时间<1ms

1. **域名分流**  
   支持域名分流，不同类型的域名到不同的DNS服务器查询。

1. **支持IPV4, IPV6双栈**  
   支持IPV4，IPV6网络，支持查询A, AAAA记录，支持双栈IP速度优化，并支持完全禁用IPV6 AAAA解析。

1. **高性能，占用资源少**  
   多线程异步IO模式，cache缓存查询结果。

## 工作原理

![Architecture](https://raw.github.com/pymumu/smartdns/master/doc/architecture.png)

1. SmartDNS接收本地网络设备的DNS查询请求，如PC，手机的查询请求。  
2. SmartDNS将查询请求发送到多个上游DNS服务器，可采用标准UDP查询，非标准端口UDP查询，及TCP查询。  
3. 上游DNS服务器返回域名对应的Server IP地址列表。SmartDNS检测与本地网络访问速度最快的Server IP。  
4. 将访问速度最快的Server IP返回给本地客户端。  

## 使用

   配置位于/system/etc/smartdns/smartdns.conf

    安装完成后，可配置smartdns的上游服务器信息。具体配置参数参考`配置参数`说明。  
    一般情况下，只需要增加`server [IP]:port`, `server-tcp [IP]:port`配置项，
    尽可能配置多个上游DNS服务器，包括国内外的服务器。配置参数请查看`配置参数`章节。

## 配置参数

|参数|  功能  |默认值|配置值|例子|
|--|--|--|--|--|
|server-name|DNS服务器名称|操作系统主机名/smartdns|符合主机名规格的字符串|server-name smartdns
|bind|DNS监听端口号|[::]:53|IP:PORT|bind 192.168.1.1:53
|bind-tcp|TCP模式DNS监听端口号|[::]:53|IP:PORT|bind-tcp 192.168.1.1:53
|cache-size|域名结果缓存个数|512|数字|cache-size 512
|tcp-idle-time|TCP链接空闲超时时间|120|数字|tcp-idle-time 120
|rr-ttl|域名结果TTL|远程查询结果|大于0的数字|rr-ttl 600
|rr-ttl-min|允许的最小TTL值|远程查询结果|大于0的数字|rr-ttl-min 60
|rr-ttl-max|允许的最大TTL值|远程查询结果|大于0的数字|rr-ttl-max 600
|log-level|设置日志级别|error|fatal,error,warn,notice,info,debug|log-level error
|log-file|日志文件路径|/var/log/smartdns.log|路径|log-file /var/log/smartdns.log
|log-size|日志大小|128K|数字+K,M,G|log-size 128K
|log-num|日志归档个数|2|数字|log-num 2
|audit-enable|设置审计启用|no|[yes\|no]|audit-enable yes
|audit-file|审计文件路径|/var/log/smartdns-audit.log|路径|audit-file /var/log/smartdns-audit.log
|audit-size|审计大小|128K|数字+K,M,G|audit-size 128K
|audit-num|审计归档个数|2|数字|audit-num 2
|conf-file|附加配置文件|无|文件路径|conf-file /etc/smartdns/smartdns.more.conf
|server|上游UDP DNS|无|可重复<br>`[ip][:port]`：服务器IP，端口可选。<br>`[-blacklist-ip]`：blacklist-ip参数指定使用blacklist-ip配置IP过滤结果。<br>`[-whitelist-ip]`：whitelist-ip参数指定仅接受whitelist-ip中配置IP范围。<br>`[-check-edns]`：edns过滤。<br>`[-group [group] ...]`：DNS服务器所属组，比如office, foreign，和nameserver配套使用。<br>`[-exclude-default-group]`：将DNS服务器从默认组中排除| server 8.8.8.8:53 -blacklist-ip -check-edns -group g1
|server-tcp|上游TCP DNS|无|可重复<br>`[ip][:port]`：服务器IP，端口可选。<br>`[-blacklist-ip]`：blacklist-ip参数指定使用blacklist-ip配置IP过滤结果。<br>`[-whitelist-ip]`：whitelist-ip参数指定仅接受whitelist-ip中配置IP范围。<br>`[-group [group] ...]`：DNS服务器所属组，比如office, foreign，和nameserver配套使用。<br>`[-exclude-default-group]`：将DNS服务器从默认组中排除| server-tcp 8.8.8.8:53
|server-tls|上游TLS DNS|无|可重复<br>`[ip][:port]`：服务器IP，端口可选。<br>`[-spki-pin [sha256-pin]]`: TLS合法性校验SPKI值，base64编码的sha256 SPKI pin值<br>`[-host-name]`：TLS SNI名称。<br>`[-blacklist-ip]`：blacklist-ip参数指定使用blacklist-ip配置IP过滤结果。<br>`[-whitelist-ip]`：whitelist-ip参数指定仅接受whitelist-ip中配置IP范围。<br>`[-group [group] ...]`：DNS服务器所属组，比如office, foreign，和nameserver配套使用。<br>`[-exclude-default-group]`：将DNS服务器从默认组中排除| server-tls 8.8.8.8:853
|server-https|上游HTTPS DNS|无|可重复<br>`https://[host][:port]/path`：服务器IP，端口可选。<br>`[-spki-pin [sha256-pin]]`: TLS合法性校验SPKI值，base64编码的sha256 SPKI pin值<br>`[-host-name]`：TLS SNI名称<br>`[-http-host]`：http协议头主机名。<br>`[-blacklist-ip]`：blacklist-ip参数指定使用blacklist-ip配置IP过滤结果。<br>`[-whitelist-ip]`：whitelist-ip参数指定仅接受whitelist-ip中配置IP范围。<br>`[-group [group] ...]`：DNS服务器所属组，比如office, foreign，和nameserver配套使用。<br>`[-exclude-default-group]`：将DNS服务器从默认组中排除| server-https https://cloudflare-dns.com/dns-query
|address|指定域名IP地址|无|address /domain/[ip\|-\|-4\|-6\|#\|#4\|#6] <br>`-`表示忽略 <br>`#`表示返回SOA <br>`4`表示IPV4 <br>`6`表示IPV6| address /www.example.com/1.2.3.4
|nameserver|指定域名使用server组解析|无|nameserver /domain/[group\|-], `group`为组名，`-`表示忽略此规则，配套server中的`-group`参数使用| nameserver /www.example.com/office
|ipset|域名IPSET|None|ipset /domain/[ipset\|-], `-`表示忽略|ipset /www.example.com/pass
|ipset-timeout|设置IPSET超时功能启用|auto|[yes]|ipset-timeout yes
|bogus-nxdomain|假冒IP地址过滤|无|[ip/subnet]，可重复| bogus-nxdomain 1.2.3.4/16
|ignore-ip|忽略IP地址|无|[ip/subnet]，可重复| ignore-ip 1.2.3.4/16
|whitelist-ip|白名单IP地址|无|[ip/subnet]，可重复| whitelist-ip 1.2.3.4/16
|blacklist-ip|黑名单IP地址|无|[ip/subnet]，可重复| blacklist-ip 1.2.3.4/16
|force-AAAA-SOA|强制AAAA地址返回SOA|no|[yes\|no]|force-AAAA-SOA yes
|prefetch-domain|域名预先获取功能|no|[yes\|no]|prefetch-domain yes
|dualstack-ip-selection|双栈IP优选|no|[yes\|no]|dualstack-ip-selection yes
|dualstack-ip-selection-threshold|双栈IP优选阈值|30ms|毫秒|dualstack-ip-selection-threshold [0-1000]

## 感谢

- SmartDNS | [pymumu](https://github.com/pymumu/smartdns)

## 捐赠

如果你觉得此项目对你有帮助，请捐助他们，以使项目能持续发展，更加完善。

### PayPal

[![Support via PayPal](https://cdn.rawgit.com/twolfson/paypal-github-button/1.0.0/dist/button.svg)](https://paypal.me/PengNick/)

### Alipay 支付宝

![alipay](https://raw.github.com/pymumu/smartdns/master/doc/alipay_donate.jpg)

### Wechat 微信
  
![wechat](https://raw.github.com/pymumu/smartdns/master/doc/wechat_donate.jpg)

## 声明

* `SmartDNS`著作权归属Nick Peng (pymumu at gmail.com)。
* `SmartDNS`为免费软件，用户可以非商业性地复制和使用`SmartDNS`。
* 禁止将 `SmartDNS` 用于商业用途。
* 使用本软件的风险由用户自行承担，在适用法律允许的最大范围内，对因使用本产品所产生的损害及风险，包括但不限于直接或间接的个人损害、商业赢利的丧失、贸易中断、商业信息的丢失或任何其它经济损失，不承担任何责任。
* 本软件不会未经用户同意收集任何用户信息。

## 说明

目前核心代码未开源。