---
title: Preface
date: '2021-11-08'
publish: false
---

# Preface

本主题主要内容是学习煎鱼([eddycjy](https://github.com/eddycjy))的开源教程**[ go-gin-example](https://github.com/eddycjy/go-gin-example)**的笔记，会在自己的理解上添加代码示例和说明

学习笔记针对原教程新增了如下改动：

**引用库**：

-  [spf13](https://github.com/spf13)/**[cast](https://github.com/spf13/cast)**取代 [unknwon](https://github.com/unknwon)/**[com](https://github.com/unknwon/com)**（已停止维护）
-  gorm 使用 v2.0 版本，API进行重构
-  使用 [viper](https://github.com/spf13/viper) 进行配置管理

**项目结构**：

```
go-gin-example
├── api
├── conf
├── core
├── dao
├── global
├── middleware
├── models
├── routers
├── runtime
├── service
└── utils
```

- api : 路由处理逻辑
- conf : 用于存储配置文件
- middleware : 应用中间件
- models : 应用数据库模型
- global : 全局变量，viper, gorm 等
- routers : 路由逻辑处理
- runtime : 应用运行时数据
- core : 应用核心组件
- service : 业务服务层
- dao : 数据库访问层
- utils : 工具包

将 request 和 response 封装起来，取代简单的 map

## 链接

1. [eddycjy](https://github.com/eddycjy)/**[go-gin-example](https://github.com/eddycjy/go-gin-example)** github repo

2. [eddycjy](https://github.com/eddycjy)/**[blog](https://github.com/eddycjy/blog)** 煎鱼 blog

