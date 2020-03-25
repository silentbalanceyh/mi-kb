# 全局覆盖性变量

全局覆盖性变量主要用于穿透整个Control的树形结构，使得特殊的数据可以直接传入到不同的组件中执行分发，并且这些变量有别于Zero UI中本身存在的变量信息。

## 1. 变量说明

| 变量名称.属性名 | 说明 |
| :--- | :--- |
| $metadata.control | 当前配置中的control节点，处理DataEvent构造target事件目标专用 |
| $metadata.ajax | 当前配置中的ajax节点，Lazy模式下构造Promise专用 |
| $container | 顶层父组件引用，所有的组件能够拿到的$container都是顶层引用 |
| $datum | 所有Ajax（非Lazy模式）加载的数据信息，使用ajaxKey = 响应数据 |
| $query | 当前组件使用的查询专用，controlId = 查询条件 |
| $generator | Lazy模式下的Promise生成器，ajaxKey = Function的模式，参数为该Promise的入参 |
| $connect | ajaxKey = controlId，记录了Control组件消费ajax的核心连接表，该连接表只针对component中的data节点中的配置，即为主数据连接表 |

## 2. 组件/容器核心变量

| 变量名.属性名 | 说明 |
| :--- | :--- |
| id | 当前组件的UUID |
| name | 当前组件的名称，如OxCard，标识当前组件类型 |
| event | 当前组件的事件配置对象 |
| config | 当前组件所需的配置信息（根据不同的组件配置有所区别） |
| config.mapping | 当前组件使用的数据转换节点 |

## 3. 协变变量

协变变量是在第一部分的基础之上进行过处理的变量，这些变量在进入component组件过后，会变成component的专用变量。

| 变量名称 | 协变规则 |
| :--- | :--- |
| $query | 使用当前组件的id在上层$query中进行定位，查询出当前组件专用的初始化条件（以及变化过）的条件。 |
| $generator | 当前组件绑定的data节点，先从$connect中照当当前组件使用的\_dataKey，这个\_dataKey可以直接从$datum变量中去捕捉当前组件绑定的数据信息，实际上用于处理DataEvent发送给目标组件引起目标组件改变的情况。 |



