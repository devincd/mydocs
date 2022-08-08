## DNS概述
DNS 为 Kubernetes 集群内的工作负载提供域名解析服务。本文主要介绍 Kubernetes 集群中 DNS 域名解析原理和 CoreDNS。


## Kubernetes 集群中 DNS 域名解析原理
容器集群中`kubelet`的启动参数有`--cluster-dns=<dns-service-ip>`和`--cluster-domain=<default-local-domain>`，
这两个参数分别被用来设置集群 DNS 服务器的 IP 地址和主域名后缀。

容器集群一般会部署一套 CoreDNS 工作负载，并通过`kube-dns`的服务名暴露 DNS 服务。集群会根据 Pod 内的配置，
将域名请求发往集群 DNS 服务器获取结果。Pod 内的 DNS 域名解析配置文件为`/etc/resolv.conf`，文件内容如下：
```shell
nameserver 172.xx.x.xx
search kube-system.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```
文件内容说明：


DNS 解析原理示例图
![dns-theory.png](./images/dns-theory.png)

| 序号  | 描述                                                                                                                                                                |
|-----|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ①   | 业务Pod（Pod Client）试图访问Nginx服务（Service Nginx）时，先会请求本地DNS配置文件（/etc/resolv.conf）中指向的DNS服务器（nameserver 172.21.0.10，即Service kube-dns）获取服务IP地址，得到解析结果为172.21.0.30的IP地址。 |
| ②   | 业务Pod（Pod Client）再直接发起往该IP地址的请求，请求最终经过Nginx服务（Service Nginx）转发到达后端的Nginx容器（Pod Nginx-1和Pod Nginx-2）上。                                                             |


## 参考文献
1.https://support.huaweicloud.com/intl/zh-cn/dns/index.html
2.https://intl.cloud.tencent.com/zh/document/product/457/39125