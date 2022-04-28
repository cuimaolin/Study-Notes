链路追踪的调研（字节系统研发一面）

追踪的原理（字节系统研发一面）

> 基本概念
>
> - Trace就表示一个完整的调用链
>
> - span表示一个RPC调用
>
> - 使用grpc发送数据至落库（ES）
>
> Java 使用字节码增强进行接入，在运行Java命令时指定agent的位置
>
> [Dapper,大规模分布式系统的跟踪系统](https://cloud.tencent.com/developer/article/1031932)

链路追踪的作用，技术选型的依据（百度一面）
