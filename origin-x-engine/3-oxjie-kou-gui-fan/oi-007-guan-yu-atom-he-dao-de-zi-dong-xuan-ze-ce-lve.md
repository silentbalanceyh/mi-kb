# 关于Atom和Dao的自动选择策略

## 1. 核心概念

Ox中有两个核心对象：

* DataAtom：`cn.originx.modeling.atom.data.DataAtom`类
* OxDao：`cn.originx.uca.dao.OxDao`类

这两个对象主控了当前系统需要访问的数据库（库级）和访问的模型（表级），往往在扩展的时候，直接书写对应的 `AbstractComponent` 就可以直接实现当前通道了，其中 Atom 和 Dao 会包含两种：

