# Control的基本结构

## 1. 连接布局

前文中已经讲解了Origin X Engine中的`page`和`ajax`两部分内容，本文主要针对`control`节点进行说明，在章节《[SPO-004 页面布局说明](/specification/3-specification-origin-x/spo-004-ye-mian-bu-ju-shuo-ming.html)》中，我们看到了下边的布局信息：

```json
{
    "page":{
        "layout":[
            "2ba7d8a5-c840-4bdf-9449-72c1407c5262",
            [
                {
                    "span":8,
                    "value":[
                        "6066a32c-d0b4-4243-8e21-b495cf49b1d4"
                    ]
                },
                {
                    "span":16,
                    "value":[
                        "b2fbb200-0371-4ca0-a174-080562613087"
                    ]
                }
            ]
        ]
    }
}
```

上边的UUID代表什么呢？实际上在对应的配置中如果出现了上述的节点，那么需要配置另外的节点，就是`control`，配置的内容如下：

```json
{
    "control":{
        "2ba7d8a5-c840-4bdf-9449-72c1407c5262":{},
        "6066a32c-d0b4-4243-8e21-b495cf49b1d4":{},
        "b2fbb200-0371-4ca0-a174-080562613087":{
            "container":{}
            "component":{}
        }
    }
}
```

上边配置中的每一个UUID的节点会设置当前控件的相关配置。

## 2. 基本结构

每一个Control的节点主要结构如下：

| 节点 | 子节点 | 含义 |
| :--- | :--- | :--- |
| container |  | 可选，当前控件使用的容器组件 |
|  | name | 容器组件的名称 |
|  | event | 事件绑定节点 |
|  | config | 容器组件的配置 |
| component |  | 必须，当前控件使用的组件名称（可以继续使用容器组件） |
|  | name | 组件的名称 |
|  | data | 数据绑定节点 |
|  | event | 事件绑定节点 |
|  | config | 组件的配置 |

* 每一个control节点可以包含两个核心节点：container容器和component组件。
* container容器节点不能包含绑定的data数据节点。
* component组件虽然是组件信息，但这个组件同样可以是容器组件，此时是可以使用data数据绑定节点的。
* container容器节点属于可选，如果不适用容器直接渲染组件，那么可以不配置。

关于数据和事件部分可以参考：

* [SPO-012 数据绑定节点data](/specification/3-specification-origin-x/spo-012-shu-ju-bang-ding-jie-dian-data.html)
* [SPO-015 不同组件的event节点](/specification/3-specification-origin-x/spo-015-bu-tong-zu-jian-de-event-jie-dian.html)



