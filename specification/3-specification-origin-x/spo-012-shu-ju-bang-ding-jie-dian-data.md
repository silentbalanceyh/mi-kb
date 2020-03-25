# 数据绑定节点data

## 1. 基本说明

当一个组件和Ajax中定义的ajaxKey绑定过后，那么这个被绑定的数据内容会出现在props中的专用属性`data`里，也就是说一旦实现了绑定，那么您就可以直接使用如下代码来提取这个数据：

```js
const { data = {} } = this.props;
// 或
const { data = [] } = this.props;
```

从某种意义上说，这里的`data`变量才表示当前控件Control使用的“主实体”数据。

## 2. 配置

一般这个绑定关系是在`component`的`data`节点中配置的，如下：

```json
{
    "control": {
        "1d43acbc-0079-4475-b274-d32f65b7ebe7": {
            "component": {
                "name": "OxListTask",
                "data": "personal.pager.zone",
                "config": {
                }
            }
        }
    }
}
```

上述配置就让一个组件`OxListTask`的实例和`ajaxKey = personal.pager.zone`的Ajax远程接口实现了绑定，那么该接口返回的数据可以直接通过上边的JS代码获取到。

## 3. 转换

除了上边提到的数据本身的提取以外，所有的组件/容器的配置中都支持一个特殊节点：`mapping`，实现数据映射的功能，这个地方不详细说明数据映射的细节，直接通过例子来阐述。

### 3.1. 远程Ajax读取的数据

比如有如下一份数据：

```json
[
    {
        "title": "我的服务器",
        "description": "我所管理的服务器列表。",
        "icon": "desktop",
        "cis": 12,
        "added": 3,
        "tag": "服务器",
        "category": "OWNER",
        "updatedAt": "2019-03-15T20:59:56",
        "key": "f1a3c82d-2ab1-4b7d-8076-eafe43ac745a",
    }
]
```

如果它作为主数据，那么提取的代码如：

```js
const { data = [] } = this.props
```

### 3.2. mapping的配置

在使用这份数据的Control中配置了如下信息：

```json
{
    "config": {
        "pagination": {
            "showSizeChanger": false,
            "showQuickJumper": true
        },
        "size": "large",
        "action": {
            "text": "编辑圈子",
            "uri": "/dp/"
        },
        "columns": [
            {
                "metadata": "count,配置项数量,EXPRESSION",
                "$expr": ":value个",
                "style": {
                    "fontWeight": 700,
                    "color": "black"
                }
            },
            {
                "metadata": "added,新增数量,EXPRESSION",
                "$expr": ":value个",
                "style": {
                    "fontWeight": 700,
                    "color": "red"
                }
            },
            {
                "metadata": "createdAt,创建时间,DATE",
                "$format": "YYYY-MM-DD HH:mm:ss"
            }
        ],
        "mapping": {
            "count":"cis"
        }
    }
}
```

其中可以看到mapping的配置如：

```json
{
    "count":"cis"
}
```

它表示将原始数据中的`cis`字段转换成`count`的最终字段，然后执行渲染，而且在config配置中可以看到如下：

```json
{
        "columns": [
            {
                "metadata": "count,配置项数量,EXPRESSION",
                "$expr": ":value个",
                "style": {
                    "fontWeight": 700,
                    "color": "black"
                }
            }
        ]
}
```

也就是说columns最终在List中绑定的字段名是`count`而不是`cis`，那么mapping配置就直接将后端Ajax请求接口中的数据和前端绑定的字段重新做了一次“映射”，实现了绑定，倘若接口数据发生变化，只需要更改mapping配置就可以完成数据的完整对接。

