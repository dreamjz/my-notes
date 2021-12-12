---
title: Preface
date: '2021-11-08'
publish: false
---

# Preface

在学习了后端的 Gin 和 Gorm 和前端的 Vuex 和 Vue Router之后，是时候实践和巩固已学习的知识点了

接下来将创建一个简单的 blog application，将实现以下功能：

- 登录验证
- 路由鉴权
- 文章的添加，删除，修改和查询
- Markdown 格式文章的展示

开发环境：

- OS: 5.10.79-1-MANJARO
- Database: 10.6.5-MariaDB Arch Linux
- Golang: 1.17
- Node: 16.11.1

## 1. Frontend

前端项目将以 [PanJiaChen](https://github.com/PanJiaChen)/**[vue-admin-template](https://github.com/PanJiaChen/vue-admin-template)**作为模板进行创建

**目录结构**

```
├── build                      # 构建相关
├── mock                       # 项目mock 模拟数据
├── plop-templates             # 基本模板
├── public                     # 静态资源
│   │── favicon.ico            # favicon图标
│   └── index.html             # html模板
├── src                        # 源代码
│   ├── api                    # 所有请求
│   ├── assets                 # 主题 字体等静态资源
│   ├── components             # 全局公用组件
│   ├── directive              # 全局指令
│   ├── filters                # 全局 filter
│   ├── icons                  # 项目所有 svg icons
│   ├── lang                   # 国际化 language
│   ├── layout                 # 全局 layout
│   ├── router                 # 路由
│   ├── store                  # 全局 store管理
│   ├── styles                 # 全局样式
│   ├── utils                  # 全局公用方法
│   ├── vendor                 # 公用vendor
│   ├── views                  # views 所有页面
│   ├── App.vue                # 入口页面
│   ├── main.js                # 入口文件 加载组件 初始化等
│   └── permission.js          # 权限管理
├── tests                      # 测试
├── .env.xxx                   # 环境变量配置
├── .eslintrc.js               # eslint 配置项
├── .babelrc                   # babel-loader 配置
├── .travis.yml                # 自动化CI配置
├── vue.config.js              # vue-cli 配置
├── postcss.config.js          # postcss 配置
└── package.json               # package.json
```

## 2. Backend

**项目文件结构**：

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

