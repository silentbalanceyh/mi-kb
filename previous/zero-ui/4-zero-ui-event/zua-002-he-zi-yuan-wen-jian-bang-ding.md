# 和资源文件绑定

由于Zero UI中已经包含了默认模板，对于模板细节的开发，这里不再做详细说明，主要的连接参考前文，有下边这段代码来渲染模板内的子页面。

```jsx
const { component:Component } = this.props;
return (
    <div>
        <Component/>
    </div>
)
```

本章我们继续在第一章的Demo中修改，把中间显示的文字改成多语种。

## 1. 创建资源文件

创建下边两个文件

```
src/cab/cn/components/test/page/UI.json
src/cab/en/components/test/page/UI.json
```

上述文件中的内容分别为：

```json
{
    "_message": {
        "title": "你好，浪浪"
    }
}
```

```json
{
    "_message": {
        "title": "Hello Lang"
    }
}
```

## 2. 创建Cab.json

在当前页面目录中创建`Cab.json`文件：`src/components/test/page/Cab.json`。

```json
{
    "ns": "components/test/page"
}
```

## 3. 修改UI.js

将UI.js修改成下边这种：

```jsx
import React from 'react'
import {Tag} from 'antd';
import Ux from 'ux';

const {zero} = Ux;

@zero(Ux.rxEtat(require("./Cab"))
    .cab("UI")  // 资源文件文件名
    .to()
)
class Component extends React.PureComponent {
    render() {
        // 提取
        const message = Ux.fromHoc(this, "message");
        return (
            <div>
                <Tag color={"red"}>
                    {message.title}
                </Tag>
            </div>
        )
    }
}

export default Component
```

* 上边代码中使用了`Ux.fromHoc`的方式提取资源文件，这里传入参数不传下划线。
* require是原生代码，直接从当前目录中读取Cab.json文件内容。
* 绑定的资源文件名称可根据实际情况而定。

## 4. 运行

| 运行次数 | 环境变量 |
| :--- | :--- |
| 第一次运行 | Z\_LANGUAGE = cn |
| 第二次运行 | Z\_LANGUAGE = en |

两次的截图分别：

![](/assets/images/zua/002/cn-language.png)![](/assets/images/zua/002/en-language.png)

