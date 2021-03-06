### 心动网络游戏后端评审要点

* 维护公告与CDN与游戏服务器集群进行线路级别的隔离

	避免游戏服务器集群维护或故障导致公告不可见，也避免浪费宝贵的游戏服带宽。建议使用阿里云的OSS服务来作为公告的数据源。

* 公告要支持多地区、多语言

* 关于入口网关Gateway

	整体架构支持仅暴露1个公网网关入口，即 1个IP 1个端口（[了解原因](../misc/ddos.md)）。
	要求这个网关在需要时可以水平扩展。可以自己开发网关，
	例如：[心动网络游戏通用网关](https://github.com/xindong/frontd)

* 容器部署方案

	使用基于Docker的容器化部署方案。减少软硬件系统环境差异带来的不可控因素。并方便运维管理、灾难处理和迁移。

* 关于容灾能力

	- 无可导致全平台无法提供服务的单点故障源
	- 单点故障时（例如服务器硬件损毁）不能造成数据丢失
	- 当出现故障进行修复或维护，要有充分准备的方案，例如主从冷热备份恢复等，使宕机时间可控
	- 数据具备缓写机制，避免造成数据库性能瓶颈
	- 具备登录排队机制或逐渐递增的重试延迟来预防雪崩效应（用户登录请求量爆发超过阈值时，有登录排队或重试延迟机制）

* 所有对外开放的端口都有提供健康监测的方法

	研发侧应使用第三方监控服务监控健康状态。包括TCP，并能区分显示 pretransfer/starttransfer/total time。

* 不依赖HTTP X-Forwarded-For 或TCP连接的对端IP地址来工作或统计

	由客户端通讯延迟统计和来源IP记录。
	可以区分tcp建立连接的时间，服务端计算信息，和服务端返回的具体错误。以便分析和优化线路质量。
	使用 [MyIP服务](../services/myip.md) 来由客户端获取自己公网IP、地区信息等上传给服务端。
	服务端应该允许客户端IP异步上传，而不是登录和进入游戏的必要条件。

* 连接网络入口均有fallback机制并使用备用域名

	备用连接方案可以是：
	使用HTTPDNS或提供类似地址查询服务来确保客户端获得正确所有网络入口IP
	当DNS被污染或其他原因没有及时生效时，使用HTTPDNS或类似方式让客户端获得正确的服务端IP
	可以使用[aliyun提供的httpdns服务](https://help.aliyun.com/document_detail/dpa/sdk/RESTful/httpdns.html)。

	也可以是入口网关提供备用域名（或IP）入口，当原连接域名被攻击时，可以使用备用的位于高防IP上的入口。
	备用域名（或IP）固化在客户端，当通过HTTPDNS方式仍无法取到正确域名时，使用备用域名（或IP）连接。

* 是否已经完成与GM工具和核心数据的对接

	游戏数据和日志必须符合财务审计要求

* 关于主要货币的数据存储规格

	- 使用 64 bits 记录。不能在数值大于43亿时溢出
	- 元宝等重要货币要可以被扣成负数

* 定期的数据库备份和快照

	每日有一份完整的数据库备份或关键数据快照（所有玩家对应的经验、元宝、装备信息）。
	并永久存档（存储方式由心动提供 OSS 或 S3）供审计等随时备查

* 服务器数据库回档速度

	当有回档需求时，是否有准备操作流程文档。回档所需的维护时间是否可控

* 项目上线如果用阿里云，应使用RDS。

* 有服务端性能日志

	特别是各类请求对应的处理消耗时间记录。用于优化负载能力，若版本不稳定可后迅速反应。

* 有 CrashDump 和分析能力

* 具备可靠的压力测试方案并进行充分的测试

        万人级别机器人模拟登录和随机游戏流程

* CDN更新流程已经同步

	CDN 更新要尽量避免文件覆盖
	- 一方面CDN缓存通常不能妥善处理文件内容被替换的场景
	- 一方面历史文件应该均保留存档，以备回滚或事故分析等

	使用HTTPS CDN
	
	CDN上的文件是区分大小写的，这一点与Win/Mac不同。测试时应该使用Linux服务模拟CDN以便确认。

* 准备上架应用宝的游戏需要部署到腾讯云


#### 安全
* 参考 [心动网络游戏安全评议要点](security.md)
