# Act操作组件

Act是Action的简称，在Origin X中，所有名字以`OxAct`打头的组件就属于Act操作组件，操作组件一般是主动的事件发送者，简单说在整个Origin X中，只有以`OxAct`打头的组件会被用户触发并且发送 DataEvent 给其他组件，这些组件全部位于`src/app/web/action`目录中。

## 1. 示例

下边是一个Act操作组件的示例：

```json
{
    "control": {
        "1e023544-3b19-4c4b-8dee-92617c0ae6c0": {
            "component": {
                "name": "OxActRadio",
                "event": [
                    {
                        "name": "onSearch",
                        "config": {
                            "target": "1d43acbc-0079-4475-b274-d32f65b7ebe7",
                            "query": {
                                "category": ":value"
                            }
                        }
                    }
                ],
                "config": {
                    "type": "BUTTON",
                    "style": {
                        "paddingTop": 8,
                        "paddingBottom": 8
                    },
                    "defaultValue": "__DELETE__",
                    "items": [
                        {
                            "label": "全部",
                            "key": "__DELETE__"
                        },
                        {
                            "label": "创建的",
                            "key": "OWNER"
                        },
                        {
                            "label": "加入的",
                            "key": "JOINED"
                        }
                    ]
                }
            }
        }
    }
}
```

## 2. 配置说明

Act组件的基本配置和一个Control配置一样，它和Control的配置区别如下：

* Act组件一般没有`data`数据节点，因为它通常不会去做数据的展现，都是事件发送者。
* Act组件必须带一个`event`配置，表示当前Act触发的DataEvent事件相关信息。
* 配置节点`config`根据不同的组件会有所区别。

关于事件配置的信息可参考：[SPO-015 不同组件的event节点](/specification/3-specification-origin-x/spo-015-bu-tong-zu-jian-de-event-jie-dian.html)，而DataEvent的结构可以参考：[SPO-014 DataEvent结构说明](/specification/3-specification-origin-x/spo-014-dataeventpei-zhi-shuo-ming.html)。

