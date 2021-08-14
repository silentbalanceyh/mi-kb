# Ajax中的查询引擎

Origin X中在执行复杂查询时遵循Zero中定义的查询引擎语法，主要包含以下四个基本功能：

* 分页
* 排序
* 列过滤
* 查询树

## 1. 基本格式

回到前边的配置例子：

```json
{
    "ajax": {
        "personal.pager.zone": {
            "uri": "/api/personal/circle/search",
            "method": "POST",
            "query": {
                "projection": [],
                "pager": "1,5",
                "sorter": "createdAt=DESC",
                "criteria": {
                }
            }
        }
    }
}
```

当Ajax的定义中使用了`query`作为参数节点时，证明该Ajax触发的是后端的POST查询（HTTP方法必须为POST）。

## 2. 节点说明

这里仅仅解析`query`中的节点信息：

| 节点 | 含义 |
| :--- | :--- |
| projection | 列过滤参数 |
| pager | 分页参数 |
| sorter | 排序参数（支持多字段排序） |
| criteria | 查询树 |

### 2.1. projection

列过滤节点仅支持一种格式，就是数组，数组中的每一个元素表示一个字段名，它有两个基本规则（反向思维，历史原因）：

* 如果projection的值为空数组`[]`，则表示当前查询过程中不过滤任何字段。
* 如果projection的值不为空数组，则表示当前查询中只查询projection中的字段。

### 2.2. pager

分页节点支持两种格式

**标准格式**：

```json
{
    "pager":{
        "page":1,
        "size":10
    }
}
```

* page表示当前所处的自然页码，从`1`开始计算。
* size表示当前页的尺寸，即：每页显示多少条数据。

**简化格式**：

```json
{
    "pager":"1,5"
}
```

上边等价于：page = 1, size = 5。

### 2.3. sorter

排序节点同样支持两种格式：

**标准格式：**

```json
{
    "sorter":[
        "name,DESC",
        "code,ASC"
    ]
}
```

> 上边的排序含义是：先按照name字段降序排列、然后按照code字段升序排列。

**简化格式：**

如果把上边格式改写成简化版如下：

```json
{
    "sorter":"name=DESC,code=ASC"
}
```

### 2.4. criteria

查询树的语法参考Zero UI中的教程：[ZUI-005 查询参数criteria配置解析](/zero-ui/1-zero-ui-guide/zui-005-cha-xun-can-shu-criteria-pei-zhi-jie-xi.html)

## 3. 判断查询请求

Origin X中判断一个请求是否查询请求的关键代码如下（方便开发人员理解内部原理）：

```js
const isIr = (config = {}) => {
    const {method = "GET"} = config;
    if ("POST" === method) {
        const {query = {}} = config;
        return (query.hasOwnProperty("criteria") // 不带分页查询
            || query.hasOwnProperty("pager") // 带分页查询
        )
    } else return false;
};
```



