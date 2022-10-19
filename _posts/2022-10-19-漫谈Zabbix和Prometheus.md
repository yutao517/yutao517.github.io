---
layout: article
title: 漫谈Zabbix和Prometheus
tags: Prometheus Zabbix
category: blog
date: 2022-10-19 16:08:00 +08:00
mermaid: true
---
**Zabbix**

Zabbix 是一个开源的分布式监控系统，支持多种采集方式和采集客户端，同时支持 SNMP、IPMI、JMX、Telnet、SSH 等多种协议。它将采集到的数据存放到数据库中，然后对其进行分析整理，如果符合告警规则，则触发相应的告警。

Zabbix 的核心组件主要是 Agent 和 Server。其中 ：

- Agent 主要负责采集数据并通过主动或者被动的方式将采集数据发送到 Server/Proxy。除此之外，为了扩展监控项，Agent 还支持执行自定义脚本。

- Server 主要负责接收 Agent 发送的监控信息，并进行汇总存储、触发告警等。Server 将收集的监控数据存储到 Zabbix Database 中，Zabbix Database 支持常用的关系型数据库，如 MySQL、PostgreSQL、Oracle 等（默认是 MySQL），并提供 Zabbix Web 页面（PHP 编写）数据查询。


由于使用了关系型数据存储时序数据，Zabbix在监控大规模集群时常常在数据存储方面捉襟见肘。所以从 4.2 版本后 Zabbix开始支持 TimescaleDB 时序数据库，不过目前成熟度还不高。



**Prometheus**

Prometheus 是开源监控报警系统和时序列数据库，由 Google 发起的 Linux 基金会旗下的云原生计算基金会（CNCF）在2016年将 Prometheus 纳入作为其第二大开源项目。Prometheus 在开源社区十分受欢迎，截至目前（2021.05.20）在 GitHub 上的 Star数超过36K。

从字面上理解，Prometheus 由两个部分组成，一个是监控报警系统，另一个是自带的时序数据库（TSDB）。

Prometheus 的基本原理是通过 HTTP 周期性抓取被监控组件的状态。任意组件只要提供对应的 HTTP 接口并且符合 Prometheus 定义的数据格式，就可以接入 Prometheus 监控。

Prometheus Server 负责定时在目标上抓取 metrics（指标）数据并保存到本地存储。它采用了一种 Pull（拉）的方式获取数据，不仅降低客户端的复杂度，客户端只需要采集数据，无需了解服务端情况，也让服务端可以更加方便地水平扩展。如果监控数据达到告警阈值，Prometheus Server 会通过 HTTP 将告警发送到告警模块 alertmanger，通过告警的抑制后触发邮件等。


Prometheus 支持 PromQL 提供多维度数据模型和灵活的查询，通过监控指标关联多个 tag 的方式，将监控数据进行任意维度的组合以及聚合。



**综合对比**



通过上面的简单介绍，我们可以看出Prometheus和Zabbix有很多相似之处：

- Prometheus 使用各种 Exporter 进行监控，Exporter 的功能类似于 Zabbix 的 Agent，负责收集监控对象端的数据。
- Prometheus 的 AlertManager 类似于 Zabbix 的 Action，可以进行报警触发，比如发送短信和邮件。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2202c93fdf584474b83302ee323a44bb.png)

系统成熟度：Zabbix 是老牌的监控系统，它在 1998 年就出现了，系统功能比较稳定，成熟度较高。而 Prometheus 是最近几年才诞生的，虽然功能还在不断迭代更新，但站在巨人的肩膀之上，在架构设计上借鉴了很多老牌监控系统的经验。

系统扩展性：Zabbix可以自定义各种监控脚本，并且不仅可以做到主动推送，还可以做到被动拉取。Prometheus 则定义了一套监控数据规范，并通过各种 exporter 扩展系统采集能力。

数据存储：Zabbix 采用关系数据库保存，这极大限制了 Zabbix 的采集性能；而 Prometheus 则自研了一套高性能的时序数据库，在 V3 版本可以达到每秒千万级别的数据存储，通过对接第三方时序数据库扩展历史数据的存储。

配置复杂度：Prometheus 只有一个核心 server 组件，一条命令便可以启动。相对而言，Zabbix的配置则显得略麻烦。

社区活跃度：目前 Zabbix 的社区活跃度比较低，Prometheus 在这方面占据绝对优势，社区活跃度非常高，并且受到 CNCF 的支持，后期的发展值得期待。

容器支持：由于 Zabbix 出现得比较早，当时容器还没有诞生，它们对容器的支持自然比较差；而Prometheus 出现较晚，其动态发现机制，既支持 Swarm 原生集群，又支持 Kubernetes 容器集群的监控，是目前容器监控最好的解决方案。

**监控的技术选型**



可能读者跟我一样，时常会听到很多研发人员、甚至是运维的小伙伴在争论，Zabbix 和 Prometheus 哪一个更好? 在我看来，脱离实际应用场景讨论技术的优劣其实是没有任何意义的。



如果选择对象仅仅是Zabbix 和 Prometheus（除了两者之外，业界还有许多优秀的监控系统，比如小米开源的企业级监控工具Open-Falcon，是一款灵活、可扩展并且高性能的监控方案，包括小米、滴滴、美团等互联网公司都在使用），笔者的建议是：

1.当环境是一个纯容器的环境，毫无疑问 Prometheus 是更适合的选择，Prometheus 是天生为容器化平台打造的监控系统。

2.而当环境有各种操作系统、硬件、中间件、数据库等，那么 Zabbix 可能是更适合的监控平台，Zabbix 在传统监控系统中，尤其是在服务器相关监控方面，占据绝对优势。

3.当整个环境中既有容器又有其他的系统，则可以考虑使用Zabbix 和 Prometheus两套监控系统互相补足。
