# 代码结构

> 跨项目的包表示在另外的项目中包含了同名包，Zero 框架的包结构就不在本章说明。xx 在文中代表客户名称，infix-hy 项目中的内容因客户不同而有所差异

## 1.ox-platform资源平台包

| 包名 | 说明 |
| :--- | :--- |
| cn.originx.cv | Ox 中的常量包 |
| cn.originx.cv.em | Ox 中的枚举常量包 |
| cn.originx.modeling.shell | （模型）Ox 中命令行专用模型 |
| cn.originx.optic.component | 通道标准组件包 |
| cn.originx.refine | Ox 中的工具包 |
| cn.originx.scaffold | Ox 资源平台脚手架 |
| cn.originx.scaffold.component | （抽象层）通道组件抽象层 |
| cn.originx.scaffold.plugin | （抽象层）插件抽象层 |
| cn.originx.scaffold.shell | （抽象层）命令行工具抽象层 |
| cn.originx.scaffold.testing | （抽象层）测试框架抽象层 |

## 2. ox-cmdb配置管理包

| 包名 | 说明 |
| :--- | :--- |
| cn.originx.channel | Zero Extension模块通道实现层 |
| cn.originx.cv | 常量包 |
| cn.originx.cv.em | 枚举常量包 |
| cn.originx.migration | 一键升级程序专用包 |
| cn.originx.modeling.atom.data | （模型）Ox中的记录模型（做过重构，单文件空实现） |
| cn.originx.optic.advanced | （抽象层）强化组件抽象层 |
| cn.originx.quiz | 测试框架 |
| cn.originx.uca | 自定义组件包（核心业务逻辑） |
| cn.originx.uca.ambient | 「UCA」环境初始化组件 |
| cn.originx.uca.calculator | 「UCA」核心计算包，组件选择、树选择、待办选择 |
| cn.originx.uca.calendar | 「UCA」Auditor责任人组件 |
| cn.originx.uca.cancel | 「UCA」回滚专用组件 |
| cn.originx.uca.code | 「UCA」序号生成器 |
| cn.originx.uca.codex | 「UCA」标识规则处理器 |
| cn.originx.uca.commerce | 「UCA」配置项核心业务层 |
| cn.originx.uca.concrete | 「UCA」配置项原子业务层 |
| cn.originx.uca.confirm | 「UCA」确认/拒绝组件 |
| cn.originx.uca.connector | 「UCA」关系处理器 |
| cn.originx.uca.elasticsearch | 「UCA」全文检索ES组件 |
| cn.originx.uca.log | 「UCA」日志记录器 |
| cn.originx.uca.plugin | 「UCA」通道插件实现层 |
| cn.originx.uca.ticket | 「UCA」确认单服务组件 |
| cn.originx.uca.workflow | 「UCA」工作流切换器（CMDB/ITSM） |
| cn.vertxup.atom | 建模RESTful接口 |
| cn.vertxup.cv | zero 中的 Agent/Worker地址常量 |
| cn.vertxup.es | 全文检索RESTful接口 |
| cn.vertxup.graphic | 图引擎RESTful接口 |
| cn.vertxup.ox | Ox引擎专用接口 |

## 3. 模块包结构

所有模块的核心包结构：

* `cn.originx`：Ox Engine模块根包。
* `cn.vertxup`：RESTful接口根包。

### 3.1. xx 专用包

xx包为当前客户专用模块和接口包，位于项目：

* `infix-hy/hy-xx-up`：上层接口包
* `infix-hy/hy-xx-down`：下层领域模型包

| 包名 | 说明 |
| :--- | :--- |
| cn.originx.xx.domain | 领域模型包 |
| cn.originx.channel | Zero Extension模块通道实现层 |
| cn.originx.xx.extension | 插件实现包 |
| cn.originx.xx.migration | 一键更新组件插件包 |
| cn.vertxup.xx.core | 核心业务RESTful接口 |
| cn.vertxup.xx.cv | zero 中的 Agent/Worker地址常量 |
| cn.vertxup.xx.modular | 我的圈子RESTful接口 |
| cn.vertxup.xx.oob | 变更单RESTful接口 |

### 3.2. 插件模块包

> 模块包的根包会带上模块名称，不同名称模块包名会有所区别，假设名称为 &lt;name&gt; 。

| 包名称 | 说明 |
| :--- | :--- |
| cn.originx.&lt;name&gt;.api | RESTful开放接口专用包 |
| cn.originx.&lt;name&gt;.extension | 插件实现包 |
| cn.originx.&lt;name&gt;.refine | 模块工具包 |
| cn.originx.&lt;name&gt;.atom | 模块模型专用包 |
| cn.originx.&lt;name&gt;.component | 组件实现专用包 |
| cn.originx.&lt;name&gt;.cv | 常量专用包 |
| cn.originx.&lt;name&gt;.input | 输入专用包 |
| cn.originx.&lt;name&gt;.output | 输出专用包 |
| cn.originx.&lt;name&gt;.service | 模块服务层 |
| cn.originx.&lt;name&gt;.quiz | 测试框架专用包 |
| cn.originx.&lt;name&gt;.error | 异常定义包 |
| cn.originx.optic.advanced | infix-ucmdb 中存在，组件抽象和实现包 |
| cn.originx.&lt;name&gt;.photon | UCMDB专用服务组件包 |



