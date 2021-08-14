# 页面结构说明

## 1. 基本

页面结构说明主要位于基本节点`page`下，它的格式如下：

```json
{
    "page":{
        "grid":[]
        "container":{}
    }
}
```

| 节点 | 含义 |
| :--- | :--- |
| grid | 描述当前页面的布局信息 |
| container | 顶层页面容器 |

在Origin X Engine中，一个页面对应后端的`UI_PAGE`表，里面会存储当前页面的所有配置信息，页面本身包含两部分配置内容：布局和容器，页面布局的信息可参考：[SPO-004 页面布局说明](/specification/3-specification-origin-x/spo-004-ye-mian-bu-ju-shuo-ming.html)。

## 2. Origin X页面设计原则

页面本身的布局可以直接使用容器类型的组件来完成，参考下图：

![](/assets/images/spo/003/page-layout.png)

左边是目前Origin X使用的模式，直接在上边的配置节点中引入“顶层容器”的概念，然后`grid`直接进行页面的第一次布局，使得整个页面具有容器和布局两个概念。

右边是一种协变模式，也就是说，页面可以不使用布局，而直接使用一个容器类型的control（参考：[SPO-011 容器类型的Control](/specification/3-specification-origin-x/spo-011-rong-qi-lei-xing-de-control.html)），然后通过`xuiChildren`渲染这个Control中的子控件，同样也是可以完成的。那么为什么不推荐使用右边这种呢？对于一个Web页面而言，Origin X的设计遵循两个基本原则：

* 一个页面尽可能由多个控件（Control）构成，而不是由一个单控件构成。
* 尽可能少地使用容器型控件，造成控件的多层嵌套。

从这点意义上讲，右边的这种方式不是不能在Origin X中配置，而是不太符合Origin X推荐的页面设计原则。

## 3. 限制

由于使用了图上左边这种模式来进行整体页面布局，所以顶层容器会受到一定的限制，从Origin X的规范上看，容器类型的Control分很多种，主要按照子控件的数量分成：

* 一个子控件的容器：children 数量 = 1。
* 多个子控件的容器：children 数量 &gt; 1。

对于顶层容器，Origin X中只允许使用children数量 = 1的容器，这种情况下像`OxTabContainer`就是不合适的，那么如果整个页面的初始容器是`OxTabContainer`呢，这种模式下的需求将在未来的Origin X版本中放开，如果目前遇到这种需求，可以考虑直接使用右图的这种模式来处理，而不去配置`container`节点。

