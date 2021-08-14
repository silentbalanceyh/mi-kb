# Assist辅助数据Ajax

辅助数据在Zero UI中是一直存在的一个概念，它有三个特征：

* 辅助数据需要前端发送Ajax请求到后端去取。
* 辅助数据是帮助当前页面的主实体完成业务用的，包括关联关系计算等。
* 辅助数据不是来自于字典表，后端有两种字典表：Tabular/Category，一个是列表字典，一个是类型字典，辅助数据一般是通过Ajax发送数据从另外一张表中读取得到的。

## 1. OxAssist

在Origin X中，辅助数据使用专用的辅助数据容器：`src/app/web/container/OxAssist/`来实现，它的示例配置如下：

```json
{
    "control": {
        "73ac2517-15f9-42be-b187-e571af54ede9": {
            "container": null,
            "component": {
                "name": "OxAssist",
                "config": {
                    "assist": [
                        {
                            "ajax": "default.column.full",
                            "assist": {
                                "key": "column.valid.full"
                            }
                        },
                        {
                            "ajax": "default.column.my",
                            "assist": {
                                "key": "column.valid.my"
                            }
                        }
                    ],
                    "grid": "46c0551c-17f6-4556-8d03-1e52dc3ba479"
                }
            }
        }
    }
}
```

## 2. 节点说明

| 节点 | 子节点 | 含义 |
| :--- | :--- | :--- |
| assist |  | 数组节点，支持多个Ajax处理（Lazy和非Lazy都支持） |
|  | ajax | 绑定的ajaxKey，参考：[SPO-006 Ajax请求基本说明](/specification/3-specification-origin-x/spo-006-ajaxqing-qiu-ji-ben-shuo-ming.html) |
|  | assist -&gt; key | 生成的Assist变量 |
| grid |  | 布局参数，参考：[SPO-004 页面布局说明](/specification/3-specification-origin-x/spo-004-ye-mian-bu-ju-shuo-ming.html) |

## 3. 关于Assist变量

在Zero UI中存在两种核心变量，详细内容参考：[KMZ-002 Tabular和Assist变量](/uniform-documentation/zero-concept/kmz-002-tabularhe-assist-bian-liang.html)。

比如一个变量的键值：`user.model`。

| 变量 | Redux树 | 键值 | 变量名 |
| :--- | :--- | :--- | :--- |
| Tabular变量 | out.tabular.user\_model | user.model | $t\_user\_model |
| Assist变量 | out.assist.user\_model | user.model | $a\_user\_model |

而Assist容器使用的就是Assist变量，关于Redux树的部分后续在Zero UI中会有所说明。

## 4. 在Js中提取

OxAssist是辅助数据容器，那么它从远程Ajax返回的数据主要是应用于它所包含的控件，即`grid`配置的内容，在使用的时候可以有两种方式来读取（推荐使用标准提取），一般在React组件的render方法中。

### 4.1. 原生读取

```js
const { $t_user_model } = this.props;
// $t_user_model的数据类型不是Object，而是DataObject
if($t_user_model.is()){
    const result = $t_user_model.to();
    // 这里的result有可能是一个Object，有可能是一个Array
}
```

### 4.2. 标准提取

```js
const result = Ux.onDatum(this, "user.model");
// 推荐使用标准提取，是因为参数本身在几处位置可以维持统一。
// 如在配置文件中：
{
    "assist":{
        "key":"user.model"
    }
}
// 在代码中
const result = Ux.onDatum(this, "user.model");
```



