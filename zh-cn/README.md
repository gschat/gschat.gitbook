#gs***Chat*** 简介

###一句话简介

> 一个可伸缩的IM即时通讯服务端实现： 它既能容易部署于小企业内部，也能方便的扩展为IM云服务的基础；


###开发语言

> * golang 主力开发语言
> * c/c++ 辅助开发性能关键部件（微服务）


###协议

> * gs***Chat*** 未采用XMPP等业界知名开源协议，而是采用了类似微信的通讯协议。gsChat协议和微信协议类似专门为当前移动网络环境优化。
> * gs***Chat*** 协议基于gs***RPC***开发，gsRPC的详细信息可以参考[这里](http://gsrpc.github.io/)

###架构

> * gs***Chat*** 基于微服务的概念进行开发，所有独立的功能都被实现为单独的服务；
> * gs***Chat*** 计划通过采用[Docker](https://www.docker.com)来简化部署；
> * gs***Chat*** 计划开发一系列工具简化开发/上线/运维管理过程，为实施[DevOps](https://zh.wikipedia.org/wiki/DevOps)提供强有力支持；
