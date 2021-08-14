# 解决相对路径问题

Zero UI在传统前端开发中有一个比较大的变动，就是直接在webpack级别配置了相对路径。

示例之前，假设：

```shell
组件目录：src/components/user/manage/
工具目录：src/ux/
```

## 1. 示例对比

### 1.1. 原始代码

如果是原始前端开发，代码部分引用：

```js
import React from 'react';
import Ux from '../../../ux';    // 相对路径

class Component extends React.Component{
    render(){
        const single = Ux.elementUnique(this.props.data,"code","Single");
        return false;
    }
}

export default Component
```

### 1.2. Zero中的代码

```js
import React from 'react';
import Ux from 'ux';    // 直接引用包名

class Component extends React.Component{
    render(){
        const single = Ux.elementUnique(this.props.data,"code","Single");
        return false;
    }
}

export default Component
```

### 1.3. 优点

* 目录和目录之间不形成相互依赖，当脚本要使用相对路径的时候，仅支持`当前目录`的想对路径，这样就不会造成当前目录中的代码和外部目录之间从路径上形成依赖。
* 如果要重构，比如调整目录的时候，可直接在不改变代码的情况下直接重构，改动会小很多。

## 2. 定义

这种语法的定义部分参考两处，第一个定义文件是webpack中的配置文件，另外一个文件是Ts的定义文件。

### 2.1. webpack定义

config/modules.js

```js
const path = require("path");
module.exports = {
    // Environment包：统一环境包
    environment: path.resolve(__dirname, "../src", "environment"),
    // Entity包：模型包
    entity: path.resolve(__dirname, "../src", "entity"),
    // Language：资源包
    lang: path.resolve(__dirname, "../src", "cab"),
    // Control
    web: path.resolve(__dirname, "../src", "economy"),
    // 新的Hotel组件包
    app: path.resolve(__dirname, "../src", "app"),
    // 新统一工具包
    ux: path.resolve(__dirname, "../src", "ux"),
    // Fix moment issu
    moment$: "moment/moment.js",
    // Zero Extension
    ex: path.resolve(__dirname, "../src", "extension/library"),     // Ex 库
    ei: path.resolve(__dirname, "../src", "extension/ecosystem"),   // Ex 专用组件
    oi: path.resolve(__dirname, "../src", "extension/eclat"),       // Ox 专用组件
    mock: path.resolve(__dirname, "../src", "mock"),                // Mock 专用数据
    editor: path.resolve(__dirname, "../src", "editor"),            //
    plugin: path.resolve(__dirname, "../src", "plugin"),            // 插件
};
```

### 2.2. TypeScript编译定义

tsconfig.json

```json
    "compilerOptions": {
        "...":"其他编译项配置",
        "paths": {
            "environment": [
                "src/environment/"
            ],
            "entity": [
                "src/entity/"
            ],
            "lang": [
                "src/cab/"
            ],
            "web": [
                "src/economy/"
            ],
            "ux": [
                "src/ux/"
            ],
            "app": [
                "src/app/"
            ],
            "ex": [
                "src/extension/library/"
            ],
            "ei": [
                "src/extension/ecosystem/"
            ],
            "oi": [
                "src/extension/eclat/"
            ],
            "editor": [
                "src/editor/"
            ],
            "mock": [
                "src/mock/"
            ],
            "plugin": [
                "src/plugin/"
            ]
        }
    },
    "exclude": [
        "node_modules/**/*.ts",
        "src/**/*.js",
        "config/**/*.js"
    ],
    "include": [
        "src/economy/**/*.ts",
        "src/economy/**/*.tsx",
        "src/extension/**/*.ts",
        "src/extension/**/*.tsx",
        "src/editor/**/*.ts",
        "src/editor/**/*.tsx",
        "src/entity/**/*.ts",
        "src/entity/**/*.tsx",
        "src/c*/**/*.ts"
    ]
```

## 3. 说明

最后针对几个包说明一下，描述以下这些包在系统中的职责。

| 包名 | 路径 | 职责 |
| :--- | :--- | :--- |
| environment | src/environment | 基础环境信息专用包，定义了当前系统需要使用的所有环境，react-router，redux，reducers，store，zero注解等。 |
| entity | src/entity | TypeScript定义的数据模型专用包，包含了DataObject、DataArray、DataTree、DataEvent等核心DTO，以及查询引擎专用的查询树：QQuery专用查询条件包。 |
| lang | src/cab | 语言包，和Z\_LANGUAGE绑定的语言资源包，编程过程不会常用，但zero注解会使用该包中的资源文件信息，目前框架所支持：cab/cn/下的中文语言资源包。 |
| web | src/economy | Zero UI标准交互式组件包，包含了标准的Web组件，主要包含了**表单组件**。 |
| ux | src/ux | Utility X工具包，核心工具包，以Ux为规约的常用函数工具包。 |
| app | src/app | 【项目上层】第三方专用组件包，用于处理第三方扩展组件（每个不同的项目有所区别）。 |
| mock | src/mock | 模拟数据包（Ajax专用模拟数据包，开启mock = true时使用客户端数据）。 |
| editor | src/editor | g6编辑器专用包，TypeScript书写的图编辑器工具。 |
| plugin | src/plugin | 【项目下层】临时插件包，不同项目有所不同，用于处理特殊的业务逻辑专用。 |
| ex | extension/library | Zero UI Extension工具包，等价于强化版的Utility X工具。 |
| ei | extension/ecosystem | Zero UI Extension专用组件包，包含了扩展Web组件包，包含了内聚性更强的**综合性组件**。 |
| oi | extension/eclat | Origin X Extension配置专用组件包，主要用于连接Origin X 的配置引擎，通过配置数据渲染的**可配置组件**。 |



