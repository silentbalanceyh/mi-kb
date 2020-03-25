# Table表格中的列渲染

由于Zero UI采取了配置的模式来驱动Form表单和表格Table，本文主要讲解标准的列配置，用于渲染表格专用。

## 1. 基本介绍

Zero UI中的标准表格使用Ant Design中的`<Table/>`标签来完成，所以表格的配置属性是根据Ant Design中的Table的属性来设置的，参考：[https://ant.design/components/table-cn/\#components-table-demo-dynamic-settings](https://ant.design/components/table-cn/#components-table-demo-dynamic-settings)。

可以从上边的链接看到相关的API定义：

* Table
* Column
* ColumnGroup

## 2. 表格列渲染

关于列渲染的配置部分包括Ai属性解析器可参考：

* [ZAI-002 Ai属性解析器](/zero-ui/5-zero-ui-attribute-analyzer/zai-002-aizhi-neng-shu-xing-jie-xi-qi.html)
* [ZAI-003 $KV$表达式处理](/zero-ui/5-zero-ui-attribute-analyzer/zai-003-kvbiao-da-shi-chu-li.html)

这里主要看看Zero UI中支持多少种列渲染相关信息

| 渲染类型 $render的值 | 说明 |
| :--- | :--- |
| 默认值（不设置） | 直接渲染原值，字段值是什么，就渲染什么。 |
| LOGICAL | 「支持icon属性」针对布尔值设置，即为true时显示什么，而为false时显示什么。 |
| PERCENT | 显示百分号，会直接计算成百分比带上百分号。 |
| DATE | 时间格式化，可设置时间的显示格式，使用moment支持的格式化。 |
| CURRENCY | 货币格式化，将值转换成货币（带精度、千分位），然后跟上货币单位。 |
| EXPRESSION | 表达式，直接设置字符串，使用:value来填充需要计算的值。 |
| MAPPING | 「支持icon属性」一般针对固定下拉进行渲染，真实值/显示值，数据本身存储的是真实值，然后会根据映射的结果计算成显示值。 |
| ICON | 「支持icon属性」渲染带图标的文字信息，如果需要图标修饰可设置图标信息，包括定义图标的类型、颜色、大小。 |
| DATUM | 直接和Asssit/Tabular/Category绑定，设置动态数据源，并且根据key主键值，显示对应的其他字段值。 |
| DOWNLOAD | 渲染下载链接，可直接通过配置信息下载相关附件。 |
| LINK | 渲染React Router的链接，可直接执行跳转，类似传统页面的HyperLink组件。 |
| BUTTON | 渲染不同的按钮。 |
| OP | 渲染编辑/删除标准按钮。 |

## 3. 配置详细说明（非操作类）

> $render 的配置就是标题的值，所以就不列在配置表中。

### 3.1. LOGICAL

不带图标的列配置：

```json
[
    {
        "title": "POS转账",
        "dataIndex": "pos",
        "$render": "LOGICAL",
        "$mapping": {
            "true": "是",
            "false": "否"
        }
    }
]
```

带图标的列配置：

```json
[
        {
        "metadata": "male,性别,LOGICAL",
        "$mapping": {
            "true": "男,user:#096",
            "false": "女,user:#903"
        }
    }
]
```

第一个列是标准模式、第二个列是扩展模式，设置LOGICAL时必须设置`$mapping`配置，表达式对应：`显示值、图标类型、图标颜色`。

| 配置 | 含义 |
| :--- | :--- |
| $mapping | 配置 true / false值对应的显示信息 |
|  | 值：显示值,图标类型:图标颜色 |

### 3.2. PERCENT

直接渲染带百分号的值：

```json
[
    {
        "title": "折扣",
        "dataIndex": "discount",
        "$render": "PERCENT"
    }
]
```

百分比不需要设置其他内容，直接解析小于`1`的浮点数，并且会解析成百分比的值：如`0.35`会解析成`35%`，无配置。

### 3.3. DATE

渲染时间格式的值：

```json
[
    {
        "dataIndex": "arriveTime",
        "title": "预计抵达时间",
        "$render": "DATE",
        "$format": "MM月DD日 HH点mm分"
    }
]
```

渲染时间格式需要设置配置`$format`的值，用于对时间进行格式化，采用Moment的时间模式。

| 配置 | 含义 |
| :--- | :--- |
| $format | Moment中的时间格式：YYYY-MM-DD HH:mm:ss |
| $empty | 启用空值模式，如果为空则直接不显示内容。 |

### 3.4. CURRENCY

渲染货币格式的值：

```json
[
    {
        "title": "单价",
        "dataIndex": "unitPrice",
        "$render": "CURRENCY",
        "$unit":"￥"
    }
]
```

渲染货币格式默认的`$unit`的值是`￥`，所以如果是人民币，可以不设置`$unit`。

| 配置 | 含义 |
| :--- | :--- |
| $unit | 货币单位，默认值为￥，可设置其他货币单位。 |

### 3.5. EXPRESSION

渲染表达式的值：

```json
[
    {
        "dataIndex": "roomCounter",
        "title": "预定数量",
        "$render": "EXPRESSION",
        "$expr": ":value间"
    }
]
```

渲染表达式格式，可使用任意表达式。

| 配置 | 含义 |
| :--- | :--- |
| $expr | 专用表达式，使用:value代替原始的值。 |

### 3.6. MAPPING

渲染映射值：

```json
[
    {
        "metadata": "type,类型,MAPPING",
        "$mapping": {
            "MODEL": "MODEL,业务模型,deployment-unit,16,#C03",
            "SCHEMA": "SCHEMA,数据库实体,database,16,#339"
        }
    }
]
```

渲染可带图标的映射值。

| 配置 | 含义 |
| :--- | :--- |
| $mapping | 配置不同的值的呈现信息。 |
|  | 值：键,显示值,图标类型,图标大小,图标颜色 |

### 3.7. ICON

渲染图标信息：

```json
[
    {
        "dataIndex": "status",
        "title": "状态",
        "$render": "ICON",
        "$mapping": {
            "SCHEDULED": {
                "text": "已保存",
                "icon": "save",
                "style": {
                    "color": "#48a55d"
                }
            },
            "TAKEN": {
                "text": "已入住",
                "icon": "user",
                "style": {
                    "color": "#C03"
                }
            },
            "ROOMTYPE": {
                "text": "房型不匹配",
                "icon": "close-circle-o",
                "style": {
                    "color": "#f50"
                }
            },
            "SELECTED": {
                "text": "已选择",
                "icon": "check",
                "style": {
                    "color": "#48a55d"
                }
            },
            "PENDING": {
                "text": "可排房",
                "icon": "exclamation",
                "style": {
                    "color": "#054cab"
                }
            }
        }
    }
]
```

可配置不同的值呈现的图标值

| 配置 | 含义 |
| :--- | :--- |
| $mapping | 设置Icon对应的显示值 |

### 3.8. DATUM

设置Tabular/Assist的数据绑定操作：

```json
[
    {
        "metadata": "modelId,体系信息,DATUM,,className=zero-column-width-40",
        "$datum": "source=model.selected,value=key,display=name"
    },
    {
        "metadata": "type,试题类型,DATUM",
        "$config.items": [
            "SINGLE,单选题",
            "MULTI,多选题",
            "QUESTION,问答题"
        ]
    },
    {
        "title": "佣金码",
        "dataIndex": "codeCommission",
        "$render": "DATUM",
        "$datum": {
            "source": "code.commission",
            "value": "key",
            "display": "name"
        }
    }
]
```

* 可直接设置绑定的数据源，挂载到Assist/Tabular中。
* 可设置固定数据源，类似`$config.items`中的配置。

| 配置 | 含义 |
| :--- | :--- |
| $config.items | 直接配置选项 |
| $config.datum | 可配置数据源：source, value, label |
| $datum（旧写法，Deprecated） | 可配置数据源：source, value, display |

### 3.9. DOWNLOAD

渲染下载链接：

```json
[
    {
        "metadata": "checkinTable,签到表,DOWNLOAD",
        "$download": {
            "flag": "下载签到表",
            "ajax": "/mbenv/api/attachments/download/:value"
        }
    }
]
```

| 配置 | 含义 |
| :--- | :--- |
| $download | 设置下载信息 |
| $download.flag | 下载链接显示的值 |
| $download.ajax | 下载链接Uri地址 |

## 4. 配置详细说明（操作类）

Pending



