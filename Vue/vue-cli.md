# Vue-cli

## 简介

Vue CLI 是一个基于 Vue.js 进行快速开发的完整系统，提供：

- 通过 `@vue/cli` 实现的交互式的项目脚手架。

- 通过 `@vue/cli` + `@vue/cli-service-global` 实现的零配置原型开发。

- 一个运行时依赖 (`@vue/cli-service`)，该依赖：

  - 可升级；
  - 基于 webpack 构建，并带有合理的默认配置；
  - 可以通过项目内的配置文件进行配置；
  - 可以通过插件进行扩展。
  
- 一个丰富的官方插件集合，集成了前端生态中最好的工具。

- 一套完全图形化的创建和管理 Vue.js 项目的用户界面

## 组件

Vue CLI 有几个独立的部分：

- CLI
- CLI 服务
- CLI 插件

### CLI

CLI (`@vue/cli`) 是一个全局安装的 npm 包，提供了终端里的 `vue` 命令。它可以通过 `vue create` 快速搭建一个新项目，或者直接通过 `vue serve` 构建新想法的原型。你也可以通过 `vue ui` 通过一套图形化界面管理你的所有项目。我们会在接下来的指南中逐章节深入介绍

### CLI 服务

CLI 服务 (`@vue/cli-service`) 是一个开发环境依赖。它是一个 npm 包，局部安装在每个 `@vue/cli` 创建的项目中。

CLI 服务是构建于 [webpack](http://webpack.js.org/) 和 [webpack-dev-server](https://github.com/webpack/webpack-dev-server) 之上的。它包含了：

- 加载其它 CLI 插件的核心服务；
- 一个针对绝大部分应用优化过的内部的 webpack 配置；
- 项目内部的 `vue-cli-service` 命令，提供 `serve`、`build` 和 `inspect` 命令

### CLI 插件

CLI 插件是向你的 Vue 项目提供可选功能的 npm 包，例如 Babel/TypeScript 转译、ESLint 集成、单元测试和 end-to-end 测试等。Vue CLI 插件的名字以 `@vue/cli-plugin-` (内建插件) 或 `vue-cli-plugin-` (社区插件) 开头，非常容易使用。

当你在项目内部运行 `vue-cli-service` 命令时，它会自动解析并加载 `package.json` 中列出的所有 CLI 插件。

插件可以作为项目创建过程的一部分，或在后期加入到项目中。它们也可以被归成一组可复用的 preset

## 安装

可以使用下列任一命令安装这个新的包：

```sh
npm install -g @vue/cli
# OR
yarn global add @vue/cli
```

安装之后查看版本

```sh
vue --version
```

这里我使用 yarn 全局安装后，提示无法找到命令(OS:Manjaro,SHELL:zsh)，需要将 `$HOME/.yarn/bin`添加加到`PATH`环境变量:

在`$HOME/.zshrc`中添加

```sh
export PATH=$PATH:$HOME/.yarn/bin
```

### 升级

升级全局的 Vue CLI 包，运行：

```sh
npm update -g @vue/cli
# OR
yarn global upgrade --latest @vue/cli
```

### 项目依赖

上面列出来的命令是用于升级全局的 Vue CLI。如需升级项目中的 Vue CLI 相关模块（以 `@vue/cli-plugin-` 或 `vue-cli-plugin-` 开头），请在项目目录下运行 `vue upgrade`：

```
用法： upgrade [options] [plugin-name]

（试用）升级 Vue CLI 服务及插件

选项：
  -t, --to <version>    升级 <plugin-name> 到指定的版本
  -f, --from <version>  跳过本地版本检测，默认插件是从此处指定的版本升级上来
  -r, --registry <url>  使用指定的 registry 地址安装依赖
  --all                 升级所有的插件
  --next                检查插件新版本时，包括 alpha/beta/rc 版本在内
  -h, --help            输出帮助内容
```

## 快速原型开发

可以使用 `vue serve` 和 `vue build` 命令对单个 `*.vue` 文件进行快速原型开发，不过这需要先额外安装一个全局的扩展：

```bash
npm install -g @vue/cli-service-global
```

`vue serve` 的缺点就是它需要安装全局依赖，这使得它在不同机器上的一致性不能得到保证。因此这只适用于快速原型开发

#### vue serve

```text
Usage: serve [options] [entry]

在开发环境模式下零配置为 .js 或 .vue 文件启动一个服务器


Options:

  -o, --open  打开浏览器
  -c, --copy  将本地 URL 复制到剪切板
  -h, --help  输出用法信息
```

你所需要的仅仅是一个 `App.vue` 文件：

```vue
<template>
  <h1>Hello!</h1>
</template>
```

然后在这个 `App.vue` 文件所在的目录下运行：

```bash
vue serve
```

`vue serve` 使用了和 `vue create` 创建的项目相同的默认设置 (webpack、Babel、PostCSS 和 ESLint)。它会在当前目录自动推导入口文件——入口可以是 `main.js`、`index.js`、`App.vue` 或 `app.vue` 中的一个。你也可以显式地指定入口文件：

```bash
vue serve MyComponent.vue
```

如果需要，你还可以提供一个 `index.html`、`package.json`、安装并使用本地依赖、甚至通过相应的配置文件配置 Babel、PostCSS 和 ESLint

#### vue build

```text
Usage: build [options] [entry]

在生产环境模式下零配置构建一个 .js 或 .vue 文件


Options:

  -t, --target <target>  构建目标 (app | lib | wc | wc-async, 默认值：app)
  -n, --name <name>      库的名字或 Web Components 组件的名字 (默认值：入口文件名)
  -d, --dest <dir>       输出目录 (默认值：dist)
  -h, --help             输出用法信息
```

你也可以使用 `vue build` 将目标文件构建成一个生产环境的包并用来部署：

```bash
vue build MyComponent.vue
```

`vue build` 也提供了将组件构建成为一个库或一个 Web Components 组件的能力

## 创建一个项目

#### vue create

使用`vue create `来创建一个新项目：

```sh
vue create hello-world
```

`vue create` 命令有一些可选项，你可以通过运行以下命令进行探索：

```bash
vue create --help
用法：create [options] <app-name>

创建一个由 `vue-cli-service` 提供支持的新项目


选项：

  -p, --preset <presetName>       忽略提示符并使用已保存的或远程的预设选项
  -d, --default                   忽略提示符并使用默认预设选项
  -i, --inlinePreset <json>       忽略提示符并使用内联的 JSON 字符串预设选项
  -m, --packageManager <command>  在安装依赖时使用指定的 npm 客户端
  -r, --registry <url>            在安装依赖时使用指定的 npm registry
  -g, --git [message]             强制 / 跳过 git 初始化，并可选的指定初始化提交信息
  -n, --no-git                    跳过 git 初始化
  -f, --force                     覆写目标目录可能存在的配置
  -c, --clone                     使用 git clone 获取远程预设选项
  -x, --proxy                     使用指定的代理创建项目
  -b, --bare                      创建项目时省略默认组件中的新手指导信息
  -h, --help                      输出使用帮助信息
```

#### 使用 GUI 界面

你也可以通过 `vue ui` 命令以图形化界面创建和管理项目：

```bash
vue ui
```

上述命令会打开一个浏览器窗口，并以图形化界面将你引导至项目创建的流程

## 插件和 Preset

### 插件

Vue CLI 使用了一套基于插件的架构。如果你查阅一个新创建项目的 `package.json`，就会发现依赖都是以 `@vue/cli-plugin-` 开头的。插件可以修改 webpack 的内部配置，也可以向 `vue-cli-service` 注入命令。在项目创建的过程中，绝大部分列出的特性都是通过插件来实现的

#### 现有项目中安装插件

每个 CLI 插件都会包含一个 (用来创建文件的) 生成器和一个 (用来调整 webpack 核心配置和注入命令的) 运行时插件。当你使用 `vue create` 来创建一个新项目的时候，有些插件会根据你选择的特性被预安装好。如果你想在一个已经被创建好的项目中安装一个插件，可以使用 `vue add` 命令：

```bash
vue add eslint
```

`vue add` 的设计意图是为了安装和调用 Vue CLI 插件。这不意味着替换掉普通的 npm 包。对于这些普通的 npm 包，你仍然需要选用包管理器。

这个命令将 `@vue/eslint` 解析为完整的包名 `@vue/cli-plugin-eslint`，然后从 npm 安装它，调用它的生成器。

```bash
# 这个和之前的用法等价
vue add cli-plugin-eslint
```

如果不带 `@vue` 前缀，该命令会换作解析一个 unscoped 的包。例如以下命令会安装第三方插件 `vue-cli-plugin-apollo`：

```bash
# 安装并调用 vue-cli-plugin-apollo
vue add apollo
```

你也可以基于一个指定的 scope 使用第三方插件。例如如果一个插件名为 `@foo/vue-cli-plugin-bar`，你可以这样添加它：

```bash
vue add @foo/bar
```

你可以向被安装的插件传递生成器选项 (这样做会跳过命令提示)：

```bash
vue add eslint --config airbnb --lintOn save
```

如果一个插件已经被安装，你可以使用 `vue invoke` 命令跳过安装过程，只调用它的生成器。这个命令会接受和 `vue add` 相同的参数。

如果出于一些原因你的插件列在了该项目之外的其它 `package.json` 文件里，你可以在自己项目的 `package.json` 里设置 `vuePlugins.resolveFrom` 选项指向包含其它 `package.json` 的文件夹。

例如，如果你有一个 `.config/package.json` 文件：

```json
{
  "vuePlugins": {
    "resolveFrom": ".config"
  }
}
```

#### 项目本地的插件

如果你需要在项目里直接访问插件 API 而不需要创建一个完整的插件，你可以在 `package.json` 文件中使用 `vuePlugins.service` 选项：

```json
{
  "vuePlugins": {
    "service": ["my-commands.js"]
  }
}
```

每个文件都需要暴露一个函数，接受插件 API 作为第一个参数

你也可以通过 `vuePlugins.ui` 选项添加像 UI 插件一样工作的文件：

```json
{
  "vuePlugins": {
    "ui": ["my-ui.js"]
  }
}
```

### Preset

一个 Vue CLI preset 是一个包含创建新项目所需预定义选项和插件的 JSON 对象，让用户无需在命令提示中选择它们。

在 `vue create` 过程中保存的 preset 会被放在你的 home 目录下的一个配置文件中 (`~/.vuerc`)。你可以通过直接编辑这个文件来调整、添加、删除保存好的 preset。

这里有一个 preset 的示例：

```json
{
  "useConfigFiles": true,
  "cssPreprocessor": "sass",
  "plugins": {
    "@vue/cli-plugin-babel": {},
    "@vue/cli-plugin-eslint": {
      "config": "airbnb",
      "lintOn": ["save", "commit"]
    },
    "@vue/cli-plugin-router": {},
    "@vue/cli-plugin-vuex": {}
  }
}
```

Preset 的数据会被插件生成器用来生成相应的项目文件。除了上述这些字段，你也可以为集成工具添加配置：

```json
{
  "useConfigFiles": true,
  "plugins": {...},
  "configs": {
    "vue": {...},
    "postcss": {...},
    "eslintConfig": {...},
    "jest": {...}
  }
}
```

这些额外的配置将会根据 `useConfigFiles` 的值被合并到 `package.json` 或相应的配置文件中。例如，当 `"useConfigFiles": true` 的时候，`configs` 的值将会被合并到 `vue.config.js` 中。

#### Preset 插件的版本管理

你可以显式地指定用到的插件的版本：

```json
{
  "plugins": {
    "@vue/cli-plugin-eslint": {
      "version": "^3.0.0",
      // ... 该插件的其它选项
    }
  }
}
```

注意对于官方插件来说这不是必须的——当被忽略时，CLI 会自动使用 registry 中最新的版本。不过**我们推荐为 preset 列出的所有第三方插件提供显式的版本范围**。

#### 允许插件的命令提示

每个插件在项目创建的过程中都可以注入它自己的命令提示，不过当你使用了一个 preset，这些命令提示就会被跳过，因为 Vue CLI 假设所有的插件选项都已经在 preset 中声明过了。

在有些情况下你可能希望 preset 只声明需要的插件，同时让用户通过插件注入的命令提示来保留一些灵活性。

对于这种场景你可以在插件选项中指定 `"prompts": true` 来允许注入命令提示：

```json
{
  "plugins": {
    "@vue/cli-plugin-eslint": {
      // 让用户选取他们自己的 ESLint config
      "prompts": true
    }
  }
}
```

#### 远程 Preset

你可以通过发布 git repo 将一个 preset 分享给其他开发者。这个 repo 应该包含以下文件：

- `preset.json`: 包含 preset 数据的主要文件（必需）。
- `generator.js`: 一个可以注入或是修改项目中文件的 [Generator](https://cli.vuejs.org/zh/dev-guide/plugin-dev.html#generator)。
- `prompts.js` 一个可以通过命令行对话为 generator 收集选项的 [prompts 文件](https://cli.vuejs.org/zh/dev-guide/plugin-dev.html#第三方插件的对话)。

发布 repo 后，你就可以在创建项目的时候通过 `--preset` 选项使用这个远程的 preset 了：

```bash
# 从 GitHub repo 使用 preset
vue create --preset username/repo my-project
```

GitLab 和 BitBucket 也是支持的。如果要从私有 repo 获取，请确保使用 `--clone` 选项：

```bash
vue create --preset gitlab:username/repo --clone my-project
vue create --preset bitbucket:username/repo --clone my-project

# 私有服务器
vue create --preset gitlab:my-gitlab-server.com:group/projectname --clone my-project
vue create --preset direct:ssh://git@my-gitlab-server.com/group/projectname.git --clone my-project
```

#### 加载文件系统中的 Preset

当开发一个远程 preset 的时候，你必须不厌其烦的向远程 repo 发出 push 进行反复测试。为了简化这个流程，你也可以直接在本地测试 preset。如果 `--preset` 选项的值是一个相对或绝对文件路径，或是以 `.json` 结尾，则 Vue CLI 会加载本地的 preset：

```bash
# ./my-preset 应当是一个包含 preset.json 的文件夹
vue create --preset ./my-preset my-project

# 或者，直接使用当前工作目录下的 json 文件：
vue create --preset my-preset.json my-project
```
