# 容器类型的Control

容器类型的Control定义在：`src/app/web/container`目录中，主要用于在整个Origin X动态页面链中扮演中间连接点的角色，它会对所有它内置的子组件实现属性和数据的完美继承和解析。

## 1. 容器表

目前Origin X中存在的容器组件列表如下：

| 容器名（可配置在container的name中） | 含义 |
| :--- | :--- |
| OxAssist | 辅助数据容器：[SPO-009 Assist辅助数据Ajax](/specification/3-specification-origin-x/spo-009-assistfu-zhu-shu-ju-ajax.html) |
| OxBoundary | 纯&lt;div/&gt;容器 |
| OxCard | 对应Ant Design中的Card（带有标准标题文字或者Extra右上角工具栏） |
| OxContainer | 支持Grid布局的虚拟容器，什么都不做，仅把子组件进行像page中的layout一样的Grid编排。[SPO-004 页面布局说明](/specification/3-specification-origin-x/spo-004-ye-mian-bu-ju-shuo-ming.html) |
| OxDivide | 专用左右布局容器，使用Col和Row。 |
| OxTabContainer | 对应Ant Design中的Tabs标签页。 |

## 2. 容器示例

这里列举一个容器配置的例子。

```json
{
    "control": {
        "54740a4b-1257-4985-a8f5-d49ad76852d5": {
            "container": {
                "name": "OxCard",
                "config": {
                    "header": false
                }
            },
            "component": {
                "name": "OxContainer",
                "config": {
                    "grid": [
                        [
                            {
                                "span": 16,
                                "lg": 14,
                                "xxl": 16,
                                "value": [
                                    "f47ef525-b4b4-44cb-9870-478ba5ce71ff"
                                ]
                            },
                            {
                                "span": 4,
                                "lg": 5,
                                "xxl": 4,
                                "value": [
                                    "1e023544-3b19-4c4b-8dee-92617c0ae6c0"
                                ]
                            },
                            {
                                "span": 4,
                                "lg": 5,
                                "xxl": 4,
                                "value": [
                                    "98d5f0ed-274e-4919-b0de-3a9b432b60b8"
                                ]
                            }
                        ],
                        "1d43acbc-0079-4475-b274-d32f65b7ebe7"
                    ]
                }
            }
        }
    }
}
```

> 实际上它实现的是一个内嵌的Grid布局的示例，里面所有的UUID进行容器往下的二次连接。



