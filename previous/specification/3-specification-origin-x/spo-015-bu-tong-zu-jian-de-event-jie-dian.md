# 不同组件的event节点

`event`节点主要配置了当前组件触发事件的一些行为，而且会默认被传入到`props`中，而不是直接从`config`提取，不同格式的`event`在配置上有些区别。

## 1. 基本结构

每一个event的配置基本结构主要包括下边几部分：

| 节点 | 子节点 | 含义 |
| :--- | :--- | :--- |
| name |  | 当前事件的名称，必须以on开头。 |
| parallel |  | 是否并行对象，默认为false。 |
| config |  | 当前事件的配置节点。 |
|  | target | 当前事件的作用对象，绑定事件的基础，这个值必须存在，表示当前事件触发过后发送给哪个Control。 |

## 2. 事件的模式

每一个event节点可以配置出三种不同的事件触发模式：

* 一个元素的数组Array：单事件触发。
* 多个元素的数组Array：串行执行该事件。
* 不带parallel元素或parallel=false：单事件触发（不推荐）。
* parallel=true的Object：并行执行所有事件。

其中上边的情况中一和三只是event的配置格式不同，而效果是一样的。

## 3. onBind方法

onBind方法属于事件绑定的专用方法，它会生成一个类似：

```js
event => {

}
```

的函数，这个函数可以直接赋值给组件的对象如：onClick、onChange、onSearch等，它的方法源代码如下：

```js
/**
 * 绑定传入值必须是一个合法的
 * Object包含键：parallel
 *    parallel = true：并行绑定
 *    parallel = false：单事件绑定
 * Array（长度为1）：单事件绑定
 * Array（长度>1）：顺序事件绑定
 */
const onBind = (event, reference) => {
    if (U.isArray(event)) {
        if (0 === event.length) {
            // 空事件绑定，不允许
            return Bind.bindResult({event}, "[OX] Event, 数组中的事件为空！");
        } else if (1 === event.length) {
            // 如果只有一个元素，执行单事件绑定
            return Bind.bindSingle(event, reference);
        } else {
            // 串行顺序执行
            return Bind.bindResult({event}, "[OX] Event Warning, 暂时不支持", "#c93");
        }
    } else {
        if (U.isObject(event) && !!event) {
            const {parallel = false, ...rest} = event;
            // 如果是一个对象，转换成数组执行单事件绑定
            if (parallel) {
                // TODO: 并行绑定，待定
                return Bind.bindResult({event}, "[OX] Event Warning, 暂时不支持", "#c93");
            } else {
                return Bind.bindSingle([rest], reference);
            }
        } else {
            return Bind.bindResult({event}, "[OX] Event Failure, 事件 event 的格式不合法");
        }
    }
};
```

> 目前版本的串行和并行执行还不支持，后续提供。

自定义组件中的使用代码如下（OxActSearch的源代码）：

```jsx
import React from 'react'
import {Input} from 'antd'
import Fn from 'ox.fun';

class Component extends React.PureComponent {
    render() {
        const {config = {}, event} = this.props;
        return (
            <Input.Search {...config} onSearch={Fn.onBind(event, this)}/>
        );
    }
}

export default Component
```



