# 开发Zero UI的第一个页面

本文是一个类似`Hello World`的例子，这个例子会告诉开发人员一步一步去开发一个Zero UI中的页面需要怎么办。

## 基本概念

一个Zero UI的页面主要包含下边几部分：

> `<m>`表示模块名字，`<p>`表示页面名字，比如开发用户管理可以直接设置成：src/container/user/manage和src/components/user/manage，这样就可以通过：[http://&lt;host&gt;:&lt;port&gt;/&lt;Z\_ROUTE&gt;/user/manage ](http://<host>:<port>/<Z_ROUTE>/user/manage )的地址访问了。
>
> * host：服务器的主机名
> * port：服务器端口，可通过PORT环境变量配置
> * Z\_ROUTE：环境变量设置的当前应用的根路径

环境变量说明可以参考：[ZUI-002 Zero UI中的环境变量](/zero-ui/1-zero-ui-guide/zui-002-zero-uizhong-de-huan-jing-bian-liang.html)。

| 路径 | 文件名 | 说明 |
| :--- | :--- | :--- |
| src/container/&lt;m&gt;/&lt;p&gt; | UI.js | 每一个模板的入口页 |
|  | Cab.less | 模板风格文件 |
| src/components/&lt;m&gt;/&lt;p&gt; | UI.js | 每一个页面的入口页 |
|  | Cab.less | 页面风格文件 |
| src/cab/cn/components/&lt;m&gt;/&lt;p&gt; | UI.json | 资源文件 |

有些内容可能对读者比较陌生，不过后边会有相关说明，因此，我们就直接来`Hello World`一把，上边提到的标签中的路径表示：

* `<m>`——Module，表示模块。
* `<p>`——Page，表示页面。

上边的整体结构参考：[ZUI-003 模板和页面连接文件](/zero-ui/1-zero-ui-guide/zui-003-mo-ban-he-ye-mian-lian-jie-wen-jian.html)。

示例中的默认信息：

| 环境变量 | 值 |
| :--- | :--- |
| Z\_ROUTE | ox |
| Z\_LANGUAGE | cn |
| Z\_CSS\_PREFIX | ox |

## 1. 模板文件

在`src/container/`目录中创建下边文件：

```shell
src/container/test/page/UI.js
```

文件内容如下：

```jsx
import React from 'react'
import {Col, Row} from 'antd';

class Component extends React.PureComponent {
    render() {
        const {component: Component} = this.props;
        return (
            <Row>
                <Col span={8}>
                    Left
                </Col>
                <Col span={8}>
                    <Component/>
                </Col>
                <Col span={8}>
                    Right
                </Col>
            </Row>
        )
    }
}

export default Component
```

注意这里不是使用的React默认的`children`变量，是因为Container执行完成过后可能会传给Component一些特殊的变量，所以Component渲染的赋值流程是在内部。

## 2. 页面文件

然后在`src/components/`下创建下边文件：

```
src/components/test/page/UI.js
```

文件内容如下：

```jsx
import React from 'react'
import {Tag} from 'antd';

class Component extends React.PureComponent {
    render() {
        return (
            <div>
                <Tag color={"red"}>
                    Hello World
                </Tag>
            </div>
        )
    }
}

export default Component
```

## 3. 修改route.json

由于这个例子中添加了新的模板，所以需要修改route.json，下边是改动过后的结果：

```json
{
    "defined": "_module_page",
    "special": {
        "_login_index": [
            "_login_index"
        ],
        "_test_page": [
            "_test_page"
        ]
    }
}
```

## 4. 运行

运行好过后，直接使用浏览器打开：[http://localhost:5000/ox/test/page](http://localhost:5000/ox/test/page)

![](/assets/images/zua/001/browser.png)

