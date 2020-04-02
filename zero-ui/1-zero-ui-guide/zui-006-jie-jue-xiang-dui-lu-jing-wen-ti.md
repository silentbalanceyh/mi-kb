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

| 包名 | 职责 |
| :--- | :--- |
| environment | 环境信息专用包 |
| entity | 数据模型专用包，包含了DataObject、DataArray、DataTree、DataEvent等核心DTO |
| lang | 语言包 |
| web | Zero中的自定义组件包 |
| ux | Utility X工具包 |
| app | 应用程序专用包，Origin X中的内容就放在app包中 |

Origin X中扩展了两个包出来：

| 包名 | 职责 |
| :--- | :--- |
| ox.fun | Origin X中的Fn函数专用包，调用Ox专有函数。 |
| ox.web | Origin X中的自定义组件专用包。 |



