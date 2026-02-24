# Headless Service：稳定的网络“身份证”

普通的 Service 会提供一个虚拟 IP（ClusterIP），负载均衡地将请求转发给后端的 Pod。但对于数据库集群（如 Redis, MongoDB），Pod 之间需要互相知道对方的确切地址进行数据同步。

##  什么是 Headless？

在定义 Service 时，将 `.spec.clusterIP` 设置为 **`None`**，它就变成了“无头”服务。

- **没有 VIP**：K8s 不会为它分配统一的虚拟 IP。
- **DNS 直连**：当你访问该 Service 域名时，DNS 直接返回所有后端 Pod 的具体 IP 列表。
- **可预测的域名**：StatefulSet 中的每个 Pod 会获得一个唯一的 DNS 域名： `$(pod_name).$(service_name).$(namespace).svc.cluster.local`

例如： 在 `mysql` 集群中，`mysql-0` 永远可以通过 `mysql-0.mysql` 找到，无论它的 IP 怎么变。

# 为什么 StatefulSet 必须用 Headless

StatefulSet 管理的应用（如数据库、ZooKeeper、Kafka）通常要求每个节点有独立的身份。需要知道每个节点地址，节点直接互相通信。