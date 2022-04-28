为什么选consul作为微服务注册中心（百度一面）

consul如何进行服务发现、注册中心、熔断、限流（百度一面）

> Consul 官网中介绍了 Consul 的以下几个核心功能：
>
> - **服务发现(Service Discovery)**：提供 HTTP 与 DNS 两种方式。
> - **健康检查(Health Checking)**：提供多种健康检查方式，比如 HTTP 状态码、内存使用情况、硬盘等等。
> - **键值存储(KV Store)**：可以作为服务配置中心使用，类似 Spring Cloud Config。
> - **加密服务通信(Secure Service Communication)**
> - **多数据中心(Multi Datacenter)**：Consul 通过 WAN 的 Gossip 协议，完成跨数据中心的同步。
>
> 负载均衡由客户端实现，从注册中心拿到要访问的服务的ip列表后，使用轮询
>
> 降级是指突发流量来了，保住核心服务，非核心就不接流量了
>
> 熔断是下游挂了，不访问下游返回默认