# Ai属性解析器

Ai属性解析器是Zero提供的一套简化配置文件写法的核心工具，会在渲染过程中执行预处理，将整个配置进行规范化，本文主要介绍Ai属性解析器中的表达式含义，提供给开发人员流畅地书写表达式信息，该表达式的解析定义位于文件：

```
src/ux/ai/expr/AI.Expr.js
```

关于$KV$部分的解析，可参考：[ZAI-003 $KV$表达式处理](/zero-ui/5-zero-ui-attribute-analyzer/zai-003-kvbiao-da-shi-chu-li.html)，而在属性列表中包含了$KV$的则是支持 $KV$ 表达式。

## 1. 支持的节点

| 键值 | 属性列表（注意顺序） | 使用场景 |
| :--- | :--- | :--- |
| filter | source,field,type,cond | Filter表单中的字段处理，最终会解析成criteria查询条件。 |
| bind | key,text,id,type,$KV$ | 直接绑定某个不带图标的按钮专用的，该按钮会点击 HTML中id属性的另外一个按钮，主要做连接（Connect点击）专用。 |
| field | field,optionItem.label,span,optionJsx.style.width,render,$KV$ | Form表单中的字段渲染器专用表达式，用于渲染Zero中不同的表格字段。 |
| flag | name,icon,style | 对于一些标签文字专用的表达式。 |
| icon | key,label,icon,style.fontSize,style.color | Ant Design中的Icon部分的表达式。 |
| column | dataIndex,title,$render,sorter,$KV$ | Table表格中的列渲染器，用于渲染不同的列。 |
| option | key,label,style | 选项表达式，主要用于下拉、多选、单选组等所有和选项相关的配置。 |
| button | key,text,connectId,type,icon,disabledKey,$KV$ | 用于外置特殊按钮的解析表达式，如Tab页的右上按钮，或者外层页面的虚拟按钮。 |
| action | key,text,type,icon,confirm | 特殊带Confirm表单的按钮的属性解析，onClick事件通过外置代码来实现。 |
| steps | title,key,icon,description,status | Help流程帮助信息专用的属性表达式。 |
| window | title,okText,cancelText,visible,width,maskClosable,onOk | Ant Design中的Modal绑定的窗口表达式专用，用于解析不同的窗口信息。 |
| drawer | title,placement,width,closable,maskClosable,visible | Ant Design中的抽屉表达式专用，用于解析抽屉组件信息。 |
| popover | title,placement,width,closable,visible | Ant Design中的浮游面板表达式专用，用于解析浮游面板组件信息。 |
| ajax | method,uri,page,params.pager.size,$KV$ | Ajax特殊解析表达式，用于解析Query类型的复杂查询。 |
| tabs | tab,key | Tabs组件专用的标签页表达式。 |
| direct | key,text,onClick,type,className | 直接绑定按钮专用的直接按钮表达式。 |

## 2. 支持的写法

这里的写法以Table中的columns为例，如下：

```json
{
    "table": {
        "columns": [
            {
                "title": "操作",
                "dataIndex": "key",
                "$render": "OP",
                "fixed": true,
                "$option": {
                    "edit": "编辑",
                    "delete": "删除",
                    "delete-confirm": "确认删除选择的用户记录？"
                }
            },
            "username,登录账号",
            "realname,姓名",
            "alias,备注",
            "mobile,手机号",
            "email,邮箱地址",
            {
                "title": "是否启用",
                "dataIndex": "active",
                "$render": "MAPPING",
                "$mapping": {
                    "true": "true,启用,check,16,#393",
                    "false": "false,锁定,lock,16,#f03"
                }
            },
            {
                "title": "创建时间",
                "dataIndex": "createdAt",
                "$render": "DATE",
                "$format": "YYYY-MM-DD"
            }
        ]
    }
}
```

### 2.1. 标准写法

对于column和field而言，都支持两种格式：**对象式和字符串式**。标准写法则是使用对象式：

```json
{
    "title": "是否启用",
    "dataIndex": "active",
    "$render": "MAPPING",
    "$mapping": {
        "true": "true,启用,check,16,#393",
        "false": "false,锁定,lock,16,#f03"
    }
}
```

这种模式下，所有的Object中的字段都不会被解析。

### 2.2. 简化写法

简化写法就是字符串式的，如上边看到的：

```json
[
    "email,邮箱地址"
]
```

这种简化的写法就需要启用章节一中的属性解析器，从前边可以看到`columns`中的结构如下：

```shell
dataIndex,title,$render,sorter,$KV$
```

也就是说，上边会被解析成：

```json
{
    "dataIndex":"email",
    "title":"邮箱地址",
    "$render":"aiInput",
    "sorter":false
}
```

### 2.3. 扩展写法

除了简化写法和标准写法以外，Zero UI中还支持扩展写法，用于处理一些特殊的比较复杂的需求，如：

```json
{
    "metadata": "timeliness,固定/临时时效",
    "className": "zero-column-width-19"
}
```

上边的写法含义如下：

* metadata会使用简化写法中的表达式解析。
* 解析不出来的属性则使用原始对象中的其他属性。

> 使用了属性表达式的顺序不能乱掉，顺序乱了则会失效。



