[**组件总览**](#**组件总览**)
[**控制平面（Control Plane）组件**](#**控制平面（Control%20Plane）组件**)
[**工作节点（Worker Node）组件**](#**工作节点（Worker%20Node）组件**)


### **组件总览**

Kubernetes 集群由控制平面和一个或多个工作节点组成。

![](https://kubernetes.io/zh-cn/docs/images/components-of-kubernetes.svg)

### **控制平面（Control Plane）组件**
集群的大脑，不跑业务的pod

#### **kube-apiserver（最核心）**
集群入口，提供认证、授权，kubectl、controller、schedule的操作都要经过它

#### **etcd（集群数据库）**
保存集群状态，配置，分布式、强一致性、高可用

#### **kube-schedule（调度器）**
决定pod调度到哪台Node

#### **kube-controller-manager（控制器管家）**
维持“期望状态 = 实际状态”，包括 Deployment Controller、ReplicaSet Controller、Node Controller、Job Controller
```
期望：3 个 Pod
实际：2 个 Pod
→ Controller 自动补 1 个

```

K8s集群 **自愈能力的核心**

#### **cloud-controller-manager（云环境用）**
对接云厂商 API，比如创建LoadBalancer，挂载云硬盘

### **工作节点（Worker Node）组件**
真正跑业务的地方

#### **kubelet**
Node 上最重要的组件，接收 apiserver 指令，创建删除Pod，执行健康检查（liveness / readiness）

#### **Container Runtime（容器运行时）**
Kubernetes 不直接管 Docker, 而是通过 **CRI 接口**

#### **kube-proxy（网络代理）**
实现 Service 的关键组件,负责Service → Pod 的流量转发，负载均衡，iptables / ipvs 规则

### **[官方文档](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#core-components)**
