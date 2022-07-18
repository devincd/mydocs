## 分析和监控CoreDNS日志
容器服务 Kubernetes 部署了 CoreDNS 作为集群内的 DNS 服务器，可以通过开启 CoreDNS 日志来查看分析 CoreDNS 解析慢，访问高危请求域名等问题。
本文主要介绍如何开启 CoreDNS 日志分析。

### 开启CoreDNS日志服务
> 说明：开启 CoreDNS 日志会消耗额外的 CPU（约10%，与实际请求有关）及网络流量。如果当前已经部署的 CoreDNS 副本 CPU 使用量较高，可以提前对 CoreDNS 进行扩容。

在命令空间 kube-system 下，集群有一个 coredns 配置项，可以在 coredns 配置项中的 Corefile 字段中加上 log 插件，开启 CoreDNS 每次域名解析的日志。

下面以以下容器集群为例来说明如何开启 CoreDNS 的日志服务

- Kubernetes 版本: 1.19.3
- CoreDNS 的镜像版本:1.7.0

此容器集群环境下的 coredns 的配置如下：
```
Corefile: |
  .:53 {
      errors
      log  // 此处添加 log 插件
      health
      kubernetes cluster.local in-addr.arpa ip6.arpa {
        pods insecure
        fallthrough in-addr.arpa ip6.arpa
      }
      # hosts can add hosts's item into dns, see https://coredns.io/plugins/hosts/
      hosts {
          198.18.96.191 hub.kce.ksyun.com
          fallthrough
      }
      prometheus :9153
      forward . /etc/resolv.conf
      cache 30
      loop
      reload
      loadbalance
  }
```
然后保存 configmap 之后，coreDNS 会动态加载新的配置项，如果 coredns 实例出现了以下日志，代表重新加载配置成功
```
[INFO] Reloading
[INFO] plugin/reload: Running configuration MD5 = a47f280bf7332d7736ee30906be59f52
[INFO] Reloading complete
[INFO] 10.88.0.9:36708 - 62905 "AAAA IN pgc.kce.sdns.ksyun.com.kube-system.svc.cluster.local. udp 70 false 512" NXDOMAIN qr,aa,rd 163 0.000125187s
[INFO] 10.88.0.9:60115 - 51855 "AAAA IN pgc.kce.sdns.ksyun.com.svc.cluster.local. udp 58 false 512" NXDOMAIN qr,aa,rd 151 0.000062956s
[INFO] 10.88.0.9:48034 - 45224 "AAAA IN pgc.kce.sdns.ksyun.com.cluster.local. udp 54 false 512" NXDOMAIN qr,aa,rd 147 0.000071719s
[INFO] 10.88.0.9:46189 - 27402 "AAAA IN pgc.kce.sdns.ksyun.com. udp 40 false 512" NOERROR qr,aa,rd,ra 130 0.000376155s
[INFO] 10.88.0.9:33099 - 42668 "AAAA IN pgc.kce.sdns.ksyun.com.kube-system.svc.cluster.local. udp 70 false 512" NXDOMAIN qr,aa,rd 163 0.00003966s
[INFO] 10.88.0.9:59724 - 17101 "AAAA IN pgc.kce.sdns.ksyun.com.svc.cluster.local. udp 58 false 512" NXDOMAIN qr,aa,rd 151 0.000049661s
[INFO] 10.88.0.9:45877 - 15357 "AAAA IN pgc.kce.sdns.ksyun.com.cluster.local. udp 54 false 512" NXDOMAIN qr,aa,rd 147 0.000031935s
[INFO] 10.88.0.9:36687 - 55803 "AAAA IN pgc.kce.sdns.ksyun.com. udp 40 false 512" NOERROR qr,aa,rd,ra 130 0.000025454s
```

### CoreDNS log插件使用说明
coreDNS 插件比较灵活，具体以下功能特性：
- 修改完配置后，coreDNS 自动加载，无需重启。
- 可以自定义日志 format。
- 可以只记录指定域名（NAME）的日志记录。
- 可以为日志设置级别：
  - sucess: 成功的返回
  - denial: NXDOMAIN 或者 NODATA 响应（名称存在，类型不存在），NODATA 响应将返回代码设置为 NOERROR
  - error: SERVFAIL，NOTIMP，REFUSED等等，任何远程服务器不愿意解决请求的任何东西。
  - all: 默认值，记录所有。

以下为例子
```
Corefile: |
  .:53 {
      errors
      // 此处添加 log 插件 记录所有查询不成功的 DNS 请求
      log . {
          class denial error
      }
      health
      kubernetes cluster.local in-addr.arpa ip6.arpa {
        pods insecure
        fallthrough in-addr.arpa ip6.arpa
      }
      # hosts can add hosts's item into dns, see https://coredns.io/plugins/hosts/
      hosts {
          198.18.96.191 hub.kce.ksyun.com
          fallthrough
      }
      prometheus :9153
      forward . /etc/resolv.conf
      cache 30
      loop
      reload
      loadbalance
  }
```

### 参考链接
- https://github.com/coredns/coredns/tree/master/plugin/log