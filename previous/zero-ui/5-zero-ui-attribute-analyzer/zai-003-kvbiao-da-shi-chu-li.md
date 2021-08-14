# $KV$表达式处理

$KV$表达式是属性表达式的扩展，也就是说在简化写法中，除了原生的内容以外（常用的），还支持一些属性信息。

## 1. 配置片段

参考下边的一段配置信息，Form中的某个field配置：

```json
{
    "metadata": "category,审核/评价类别,24,,aiCheckbox,itemClass=page-grid-left,colon=false",
    "optionJsx.config.items": [
        "INFORMATION-SECURITY,信息安全",
        "INFORMATION-SERVICE,信息技术服务"
    ],
    "optionJsx.className": "web-radio-padding"
}
```

关于属性本身如何解析可参考：[ZAI-002 Ai属性解析器](/zero-ui/5-zero-ui-attribute-analyzer/zai-002-aizhi-neng-shu-xing-jie-xi-qi.html)，这里可以看到，配置使用了写法中的扩展写法，参考field的定义可以知道：

```shell
field,optionItem.label,span,optionJsx.style.width,render,$KV$
```

那么上述的配置会被解析成如下标准格式：

```json
{
    "field":"category",
    "optionItem":{
        "label":"审核/评价类别",
        "className":"page-grid-left",
        "colon":false
    },
    "span":24,
    "render":"aiCheckbox",
    "optionJsx":{
        "className":"web-radio-padding",
        "config":{
            "items":[
                {
                    "key":"INFORMATION-SECURITY",
                    "label":"信息按钮"
                },
                {
                    "key":"INFORMATION-SERVICE",
                    "label":"信息技术服务"
                }
            ]
        }
    }
}
```

可能对于初学者而言，基本的内容不会太困惑，但$KV$部分呢？按照定义属性表达式在render属性的地方就截止了，那么后边那部分：`itemClass=page-grid-left,colon=false`究竟表示什么呢，这就是这个章节需要找到的答案。

## 2. 支持的$KV$属性

### 2.1. 对照表

关于$KV$部分的内容和标准模式的对照表参考如下：

| 名称 | 数据类型 | 目标节点（标准化后） |
| :--- | :--- | :--- |
| normalize | String | optionConfig.normalize |
| addonAfter | String / Object | optionJsx.addonAfter（Ant标准） |
| addonBefore | String / Object | optionJsx.addonBefore（Ant标准） |
| readOnly | Boolean | optionJsx.readOnly |
| disabled | Boolean | optionJsx.disabled |
| placeholder | String | optionJsx.placeholder |
| valuePropName | String | optionConfig.valuePropName |
| format | String | optionJsx.format |
| listType | String | optionJsx.listType |
| withCredentials | Boolean | optionJsx.withCredentials |
| text | String | optionJsx.text |
| labelSpan | Number | optionItem.labelCol.span |
| wrapperSpan | Number | optionItem.wrapperCol.span |
| allowClear | Boolean | optionJsx.allowClear |
| sorter | String | params.sorter |
| rows | Number | optionJsx.rows |
| className | String | className |
| prefix | String | optionJsx.prefix |
| suffix | String | optionJsx.suffix |
| \_submit | String | submit |
| fixed | String | fixed |
| group | String | group |
| moment | Boolean | moment |
| itemClass | String | optionItem.className |
| colon | Boolean | optionItem.colon |
| type | String | optionJsx.type |
| status | String | optionItem.status |
| showTime | Boolean | optionJsx.showTime |

### 2.2. 属性含义

| 名称 | 含义 |
| :--- | :--- |
| normalize | 参考：《2.3.1. normalize说明》，用于定义输入的文本框的格式化，限制输入专用。 |
| addonAfter | Ant Design中的addonAfter属性，Input专用，后置修饰。 |
| addonBefore | Ant Design中的addonBefore属性，Input专用，前置修饰。 |
| readOnly | 设置是否只读：readOnly |
| disabled | 设置是否禁用：disabled |
| placeholder | 参考：《2.3.2.placeholder说明》，水印效果设置，包括双水印效果。 |
| valuePropName | Ant Design中的valuePropName原生属性，一般用于CheckBox设置值为checked，替换掉value属性。 |
| format | Ant Design中的时间控件专用，用于设置时间格式，调用的是moment中的format方法。 |
| listType | Ant Design中上传控件的呈现模式，Upload专用。 |
| withCredentials | Ant Design中的上传控件专用，设置上传过程中是否带凭证。 |
| text | Ant Design中的上传控件默认显示的文字信息。 |
| labelSpan | Form中的Grid布局专用，标签宽度。 |
| wrapperSpan | Form中的Grid布局专用，字段宽度。 |
| allowClear | 一般用于下拉，设置是否允许当前选中项被清空。 |
| sorter | 设置排序的参数信息。 |
| rows | TextArea专用，设置多行文本的行信息。 |
| className | 当前组件的className专用，设置风格信息。 |
| prefix | Ant Design中的prefix属性，输入控件专用。 |
| suffix | Ant Design中的suffix属性，输入控件专用。 |
| \_submit | 设置Button按钮的提交模式，DIRECT或其他。 |
| fixed | 设置Table中的列的fixed属性，固定列专用。 |
| group | 设置分组状态信息。 |
| moment | 设置字段是否一个Moment，如果是Moment，提交表单时需要执行转换，初始化时也需要执行格式转换。 |
| itemClass | 设置Form中的Form.Item中的className属性。 |
| colon | 设置Form中的Form.Item的colon属性，是否带有分号。 |
| type | 特殊属性用于设置类型，自定义组件专用。 |
| status | 特殊属性用于设置状态，自定义组件专用。 |
| showTime | 选择日期时间时，是否显示时间信息。 |

### 2.3. 特殊说明

#### 2.3.1. normalize说明

设置normalize属性，该属性的配置如下：

```json
{
    "metadata":"price,价格,24,,aiInput,normalize=integer`8"
}
```

normalize合计支持四种格式，不同格式的含义不同，上述配置表示当前输入框只能输入整数，长度为8，注意基本语法：

```shell
normalize=<type>`<length>`<scale>   // 注：只有decimal类型会使用scale属性
```

| 支持类型，type参数 | length参数 | scale参数 | 含义 |
| :--- | :--- | :--- | :--- |
| integer | 8 | x | 只能输入正整数 |
| number | 8 | x | 只能输入数字 |
| length | 8 | x | 只能输入长度为8的字符串 |
| decimal | 8 | 2 | 只能输入2位浮点数 |

#### 2.3.2. placeholder说明

设置placeholder属性：

直接设置

```shell
optionJsx.placeholder = "请填写";
```

双设置

```shell
optionJsx.placeholder = ["从","到"];   // 时间区间控件专用
```



