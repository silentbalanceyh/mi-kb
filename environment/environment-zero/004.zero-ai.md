# Zero Ai的安装

Zero Ai是针对Zero/Zero UI量身打造的命令行工具项目（[http://www.vertxai.cn](http://www.vertxai.cn)），这个工具箱项目可以帮我们完成很多开发过程中的自动化操作（**暂时没在Windows中测试过**）。

## 1. 安装

先保证您的系统中有nodejs，然后使用下边命令安装：

```shell
npm install -g vertx-ai
```

看到命令行中输出：

```
/usr/local/bin/aj -> /usr/local/lib/node_modules/vertx-ai/src/index.js
/usr/local/bin/ai -> /usr/local/lib/node_modules/vertx-ai/src/ai.js
+ vertx-ai@0.2.13
added 2 packages from 1 contributor and updated 62 packages in 34.953s
```

证明您的工具就安装好了。

## 2. 使用帮助

所有的命令语法如下：

```shell
ai <command> <arguments>
```

### 2.1. 独立工具

独立工具的含义就是可以直接使用的工具，不依赖外部环境。

### 2.2. ZT工具

ZT工具是针对Zero设置的专用工具，当它作用于某个组件时（参考：[ZUI-001 Zero UI项目结构说明](/zero-ui/1-zero-ui-guide/zui-001-zero-uixiang-mu-jie-gou-shuo-ming.html)），那么ZT的环境变量会帮助我们对组件进行环境初始化，步骤如下：

```shell
# 这行命令执行完成后，会作用于src/components/module/page这个页面
# ZT工具每次使用之前都需要设置ZT环境变量
export ZT=module/page
```

上边环境变量设置好了过后，就会拥有下边的信息（只支持中文）：

```shell
src/components/module/page：                    # 组件开发目录
src/cab/cn/components/module/page：             # 资源文件目录
```

> ZT命令必须在项目的根目录中执行！！！

目前所有支持的command列表：

| 命令 | 分类 | 说明 |
| :--- | :--- | :--- |
| ai zero | 独立 | 初始化Zero UI工程 |
| ai key | 独立 | 为已存在的数据文件追加UUID格式的字段 |
| ai data | 独立 | 帮助开发人员生成模拟数据，主要生成Mock部分的数据 |
| ai ipc | 独立 | 分析Zero所有的微服务项目中的内部服务通讯Rpc调用 |
| ai csv | 独立 | 将一个json文件转换成CSV文件 |
| ai ui.card | ZT | 直接创建一个带有PageCard的JS文件 |
| ai ui.form | ZT | 直接创建一个带有Form的表单JS文件 |
| ai rs.left | ZT | 修改PageCard的资源文件（左工具） |
| ai rs.right | ZT | 修改PageCard的资源文件（右工具） |
| ai rs.success | ZT | 修改当前文件的modal遮罩success部分 |
| ai rs.error | ZT | 修改当前文件的modal遮罩error部分 |
| ai rs.confirm | ZT | 修改当前文件的modal遮罩confirm部分 |



