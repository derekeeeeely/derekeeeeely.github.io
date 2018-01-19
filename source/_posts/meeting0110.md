---
title: meeting0110
abbrlink: cb23723f
date: 2018-01-10 15:39:04
tags:
---

帮大佬整理下工作中的项目注意事项，算作新手村攻略。
<!-- more -->

### 项目

#### new-hawkeye

- 数据流
  - database -> server -> gateway -> redux -> react -> dom

- 项目结构
  - 举个🌰
  - logs
    - monitor_log [matrix](http://matrix.nidianwo.com/dashboard/db/hawkeye?orgId=15)
    - dwd_log [kibana](http://kibana.nidianwo.com/app/kibana#/discover?_g=())
    - action_log  记录登录人和时间，但是现在好像没啥用
  - sql
    - sql留存

- 网关
  - dubbo
    - pom [nexus](http://192.168.1.177:8081/nexus/#nexus-search;quick~mall-service-api)
    - javadoc -> js -> rpcProxy -> rpcInvoker -> hessain序列化 -> rpcInvoker -> remoteApi
    - 订阅/注册 [dubbo admin](http://192.168.11.39:8069/)
  - 权限
    - auth  mountController
    - sql平台 [dsq](http://192.168.11.39:43280/list) [dms](http://dms.nidianwo.com/)
    - 分配权限 [mis](http://192.168.11.20:18680/#!/roleadd/1/edit)
  - 问题集
    - javadoc未更新，run dubbo时下不到，通知server重发
    - rpc调用过程中查错
      - can't invoke xxx  [dubbo admin](http://192.168.11.39:8069/)检查 提供者/消费者
      - timeout，可能是服务的锅，可能需要加时间
      - 类型错误，根据meta.json node_dubbo会自动makeDto，有时需要手动转
      - 看不懂的错，一般都是服务的锅

#### hawkeye

- 项目结构
- 数据流
  - databse -> server -> gateway(new-hawkeye/java gateway) -> reflux -> react -> dom
- 网关
  - java网关
  - new-hawkeye网关

### 团队协作

#### 开发/测试

- 内部系统
  - [禅道](http://zentao.dwb123.com/index.php?m=my&f=index) 建任务
  - [wiki](http://wiki.nidianwo.com/dashboard.action) 文档
  - [Sentry](http://sentry.dianwoda.com/sentry/) 线上报错
- 分支
  - master checkout feature分支
  - 开发环境通用分支 dwd/dev-in-one
  - 测试环境通用分支 dwd/qa-in-one
  - 主干回归分支 preRelease

#### 上线

- [deploy](http://deploy.nidianwo.com/)
- [Doraemon](http://192.168.11.20:19999/#/config)
