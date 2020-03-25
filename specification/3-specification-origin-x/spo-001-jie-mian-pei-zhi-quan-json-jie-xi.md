# 界面配置全Json解析

本文用于定义Origin X中的配置规范，使用更加合理的方式来组织所有的配置信息，并描述各个不同的数据结构，统一完整规范。

## 1. 整体结构

一份完整的界面数据的结构参考：[SPO-002 界面配置全Json示例](/specification/3-specification-origin-x/spo-002-jie-mian-pei-zhi-quan-json-shi-li.html)，整个配置数据的结构如下：

```json
{
    "mock":true,
    "data":{
        "layout":{
        },
        "ajax":{
        },
        "page":{
            "grid":[
            ],
            "container":null
        },
        "control":{
            "<uuid:controlId>":{
                "container":null,
                "component":{
                }
            }
        }
    }
}
```

上述配置结构的顶层是Zero UI中的Mock结构，这里不做说明，主要集中于分析`data`数据节点的内容。

* **layout**：「保留」由于目前的项目中动态渲染（Ox模式）和静态渲染（Zero模式）使用了同一个模板，所以暂时对模板部分的配置保留。
* **ajax**：当前页面需要使用的所有 Ajax 请求数据。
* **page**：当前页面的根节点，根节点主要包含两部分内容：
  * **grid**：当前页面的整体布局，整体布局使用内部的grid来描述，后边会有所说明。
  * **container**：当前页面的外置容器（即顶层容器）——对于页面顶层容器，Ox中只支持单元素容器，即children = 1的容器，其他的容器无法作为顶层容器使用，容器位于：`src/app/web/container`目录中定义的容器组件。
* **control**：页面中某个区域的Web控件节点，该Web控件一般包含两部分内容：container和component。
  * **container**：当前控件需要使用的容器类，可支持 children  &gt; 1的情况，如标签容器，左右分割容器等。
  * **component**：当前控件的核心组件——特殊情况下核心组件也可以是一个容器，这种用于深层嵌套结构。

## 2. 细节规范

细节规范将对整个配置文件进行详细并且严格的解读，以保证实施人员完全理解这套配置的含义。

* 在前期由于Page Designer没有全部完成，所以需要研发人员使用静态Json文件来描述页面，它需要理解规范的核心含义。
* 当你需要在Ox Engine中开发新的自定义组件时，同样需要依靠该规范，才能使得配置被正确解析，才可以对Origin X Engine进行扩展开发。

当前所有需要理解的整体规范如下：

| 文档 | 详细说明 |
| :--- | :--- |
| [SPO-003 页面结构说明](/specification/3-specification-origin-x/spo-003-bu-ju-pei-zhi-gui-fan.html) | 描述一个页面如何构成 |
| [SPO-004 页面布局说明](/specification/3-specification-origin-x/spo-004-ye-mian-bu-ju-shuo-ming.html) | 描述页面布局如何使用Ant Design的Grid布局 |
| [SPO-005 响应式布局](/specification/3-specification-origin-x/spo-005-xiang-ying-shi-bu-ju.html) | 页面响应式布局的说明 |
| [SPO-006 Ajax请求基本说明](/specification/3-specification-origin-x/spo-006-ajaxqing-qiu-ji-ben-shuo-ming.html) | 描述当前页面如何调用远程接口 |
| [SPO-007 Ajax中的查询引擎](/specification/3-specification-origin-x/spo-007-ajaxzhong-de-cha-xun-yin-qing.html) | 描述分页、排序、列过滤、查询专用的远程接口 |
| [SPO-008 关于Ajax中的Lazy模式](/specification/3-specification-origin-x/spo-008-guan-yu-ajax-zhong-de-lazy-mo-shi.html) | 如何使用Lazy和非Lazy模式下的Ajax接口 |
| [SPO-009 Assist辅助数据Ajax](/specification/3-specification-origin-x/spo-009-assistfu-zhu-shu-ju-ajax.html) | 对于辅助依赖数据的Ajax请求说明 |
| [SPO-010 Control的基本结构](/specification/3-specification-origin-x/spo-010-controlde-ji-ben-jie-gou.html) | 描述一个Control（控件）的基本结构 |
| [SPO-011 容器类型的Control](/specification/3-specification-origin-x/spo-011-rong-qi-lei-xing-de-control.html) | 描述一个容器类型的Control如何执行Connect动作 |
| [SPO-012 数据绑定节点data](/specification/3-specification-origin-x/spo-012-shu-ju-bang-ding-jie-dian-data.html) | 如何让component / container和Ajax响应数据绑定 |
| [SPO-013 Act操作组件](/specification/3-specification-origin-x/spo-013-actcao-zuo-zu-jian.html) | 关于操作组件的基本配置说明 |
| [SPO-014 DataEvent结构说明](/specification/3-specification-origin-x/spo-014-dataeventpei-zhi-shuo-ming.html) | 如何封装和触发 DataEvent |
| [SPO-015 不同组件的event节点](/specification/3-specification-origin-x/spo-015-bu-tong-zu-jian-de-event-jie-dian.html) | 不同的组件在使用 event 时构造的 DataEvent的模式说明 |
| [SPO-016 全局覆盖性变量](/specification/3-specification-origin-x/spo-016-quan-ju-fu-gai-xing-bian-liang.html) | 当前Origin X使用的特殊全局变量说明 |
| [SPO-017 五种不同模式的xuiChildren说明](/specification/3-specification-origin-x/spo-017-wu-zhong-bu-tong-mo-shi-de-xuichildren-shuo-ming.html) | 描述 xuiChildren，有关连接点的强化说明 |

## 3. 术语表

| 术语 | 文中说明 |
| :--- | :--- |
| container | 容器 |
| page | 页面 |
| layout | 模板 |
| grid | 布局 |
| control | 控件（控件包含容器 + 组件） |
| component | 组件 |
| event | 事件 |
| config | 配置 |



