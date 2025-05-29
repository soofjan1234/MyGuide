------

# k8s

> 作者：hou
------
## 1.1 k8s架构
```shell
Control Plane (Master)
├── API Server：所有操作的唯一入口，来接收请求
├── Scheduler：负责将未调度（Pending）的 Pod 分配到合适的工作节点
├── Controller Manager：维护集群的期望状态，通过循环不断检查集群状态并发出调整指令
├── etcd：分布式存储系统，用于存储集群的所有状态数据
└── Cloud Controller Manager (可选)

Worker Nodes
├── kubelet：每个节点上的代理，负责将控制平面下发的 Pod 配置转换为实际运行状态。
├── kube-proxy：每个节点上的网络代理，负责实现 Kubernetes 的 Service 抽象。
└── Container Runtime (Docker/containerd)：负责运行容器化应用的实际容器引擎。
```

## 1.2 k8s组件

1. Pod: 最小部署单元，包含1个或多个共享资源的容器
2. Controller：自动化的管理者
    2.1. Deployment：定义了 Pod 的期望状态，确保指定数量的 Pod 始终运行
    2.2. StateFulSet：唯一身份 + 顺序部署 + 固定存储卷
        使用场景：数据库、ZooKeeper、Kafka、Redis（集群模式）等需要状态保存、身份区分的应用。
    2.3. DaemonSet：每个节点都运行一个 Pod，用于部署系统守护进程或监控代理
3. Service：内部通信与访问的桥梁
    为什么需要 Service？因为 Pod 的 IP 是不固定的，每次重建都会变。没有 Service，我们根本找不到 Pod
    3.1. ClusterIP：默认服务类型，用于集群内部访问
    3.2. NodePort：将服务暴露到每个节点的指定端口
    3.3. LoadBalancer：将服务暴露到云平台的负载均衡器
    3.4. ExternalName：将服务映射到外部域名
4. Ingress：
    - HTTP/HTTPS 流量的反向代理与路由
    - 公开从集群外部到集群内[**服务**]的 HTTP 和 HTTPS 路由。
    - 必须拥有一个 **Ingress 控制器** 才能满足 Ingress 的要求。 仅创建 Ingress 资源本身没有任何效果