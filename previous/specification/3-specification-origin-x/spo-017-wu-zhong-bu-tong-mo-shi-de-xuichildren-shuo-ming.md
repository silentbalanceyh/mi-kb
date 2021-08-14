# 五种不同模式的xuiChildren

本文是渲染的重点，因为里面包含了整个Origin X在传数据过程中的核心继承概念，所以会通过最简化的代码来描述整个继承过程中的区别。

## 1. 分类

先看看基本分类

| 分类 | 结构 | 说明 |
| :--- | :--- | :--- |
| 标准单控件 | container + component x 1 | xuiControl的标准结构，一个container中包含了一个component |
| 标准多控件 | container + component x n | xuiControl的标准结构，一个container中包含了多个component |
| 容器子控件 | component = control | component位置上配置的是一个容器行的组件，子组件充当了容器作用 |
| 状态容器 | container使用了state | container会将计算的一些state状态结果传递给component |
| 状态容器子组件 | component使用了state | component会将state的计算结果传递给它的子控件 |

## 2. 五种模式说明

本章节主要讲解五种模式的详细内容，本文以下边两个控件为例（一个是容器组件，一个是组件）：

**Outer**：容器组件

```jsx
class Outer extends React.PureComponent{
    render(){
        const { children } = this.props;
        return (
            <div>
                Container
            </div>
        )
    }
}
```

**Inner**：普通组件

```jsx
class Inner extends React.PureComponent{
    render(){
        return (
            <div>
                Component
            </div>
        )
    }
}
```

### 2.1. 标准单控件

标准单控件模式是最常用的模式，实际上只需要将上边的Outer的render方法改成下边这种就可以了：

```jsx
render(){
    const { children } = this.props;
    return (
        <div>
            Container
            {children}
            {/** 上边代码就是直接渲染组件的代码 **/}
        </div>
    )
}
```

外围调用代码类似于：

```jsx
return (
    <Outer>
        <Inner/>
    </Outer>
)
```

也就是说，它遵循下边几个规则：

* 在Outer的Jsx渲染中，必须使用代码将 children 变量渲染，若不使用它，即使是最后这种模式，子组件也是不会被渲染出来的。
* 直接渲染模式中，Outer内部的变量是无法传递给 children 的，也就是说 Outer 和 Inner 不存在任何数据传递和交互。
* Inner组件的属性只能在调用代码外部进行设置，而不能进入到Outer容器中设置。

### 2.2. 标准多控件

标准多控件模式也是常用的一种模式，和上边模式不同的是在外围调用代码中，渲染的是多个子控件。外围的调用代码改成：

```jsx
return (
    <Outer>
        <Inner/>
        <Inner/>
    </Outer>
)
```

这里调用代码内部使用了两个Inner组件，也就是说Outer的内部 children 在这个时候是一个数组，除了使用上边的`{children}`以外，还可以使用下边这种模式渲染：

```jsx
return (
    <div>
        {children.map(child => child)}
    </div>
)
```

> 由于React默认可以渲染集合，所以上边代码和直接使用 {children} 是等价的。

### 2.3. 容器子控件

容器子控件实际上增加了整个组件的层级，先修改Inner的代码为：

```jsx
class Inner extends React.PureComponent{
    render(){
        const { children } = this.props;
        return (
            <div>
                Component
                {children}
            </div>
        )
    }
}
```

在这种模式下，其实Inner和Outer的用法就一致了，那么外围调用代码有可能如：

```jsx
return (
    <Outer>
        <Inner>
            <OtherComponent/>
        </Inner>
    </Outer>
)
```

### 2.4. 状态容器

状态容器的外围调用代码实际上和上边的代码没什么区别，唯一的区别在内部实现部分，这种情况下，容器内部会有状态信息，然后在渲染 children 时将状态信息设置到 children 中去。

```jsx
class Outer extends React.PureComponent{
    state = {}
    componentDidMount(){
        this.setState({
            $user:{
                username:"Lang",
                password:"lang.lang"
            }
        }}
    }
    render(){
        const { $user = {} } = this.state;
        const { children } = this.props;
        return (
            <div>
                Container,
                {React.cloneElement(children,{
                    $username: $user.username
                })}
            </div>
        )
    }
}
```

从上边的代码可以知道，Outer组件将自己的状态中的`$user`变量的username字段数据传递给Inner组件了，它们之间存在内部通信，而最外层是感知不了这种传递的，如`OxAssist`就是采取的这种模式。

### 2.5. 状态容器子组件

这种模式就不重复了，结合2.3和2.4，读者可以理解了，在Origin X中，这种模式下的component本身是类似2.4定义的容器组件。

## 3. 设计理念

实际上React本身没有这么复杂，为什么要在Origin X中设计这么复杂的结构呢，主要原因如下：

* Origin X中每个控件包含两部分，Container/Component，其目的是在使用控件渲染的时候使得一些和HTML盒模型相关的属性直接使用容器来完成，这样容器可以处理掉类似外边距、内边距、边、头部、尾部这些和业务本身不相关的任务。
* React在使用 children 变量的过程中，本来就存在两种方式：
  * 是否要将当前组件的计算型数据结果传递给children，若传递则需要调用`cloneElement`的API来实现。
  * 如果不需要给children传递任何数据，那么就可以直接使用。
* 很多时候，使用Container/Component的两层架构来渲染一个组件时，Container可以在Component变得更加纯，因为像`OxAssist`和`OxBoundary`这种不渲染界面的虚拟容器在使用的时候，可以在真正的Component渲染之前将所有界面的数据准备好。

## 4. xuiChildren源代码

下边是 xuiChildren 这个函数的核心源代码，研发人员可以参考：

```js
/**
 * 子组件渲染，用于中间层容器，中间层容器的渲染主要分为两种模式
 * 1.「标准」（单组件）
 * -- xuiControl直接渲染：父容器 + 子组件：
 *    container + component 模式
 * 2.「标准」（多组件）
 * -- xuiControl直接渲染：父容器 + n * 子组件：
 *    container + components 模式
 * 3.「协变」（容器子组件模式）
 * 配置的 component 位置上的组件是一个 container 类型的组件，此时子组件充当了
 * 容器的作用，这种模式下，如果当前 control 原始包含了容器，那么会形成一个：
 * container -> component -> control 的三级结构
 * 4.「协变」（状态容器模式）
 * 配置的 container 位置上的组件本身是一个状态组件，需要将 state 变量中的一些计算
 * 出来的结果数据放到 component 中执行渲染
 * container -> component ( component 部分数据来源于 container ）
 * 5.「协变」（状态容器子组件模式）
 * 配置的 component 位置上的组件本身是一个状态组件，需要将 state 变量中的一些计算
 * 出来的结果数据放到它所关联的外置的 control 中执行第二次渲染
 * container -> component -> control 结构（ control 部分数据来源于 component ）
 */
const xuiChildren = (reference, children) => {
    const {config = {}, name = "", data} = reference.props;
    const state = reference.state;
    /**
     * 读取 config 中的 control
     * 如果 control 是一个 String，直接链接
     */
    let grid = config.grid;
    if (undefined === grid) {
        /**
         * 不使用 xuiControl
         */
        if (state) {
            /**
             * 带状态的容器
             */
            const props = _xuiInherit(reference);
            Ux.dgDebug(props, "[Ox] Container, 纯渲染，带状态，传入Props：", "#4666e4");
            return _xuiUniform(children, props);
        } else {
            /**
             * 不带状态的容器
             * 最简单的模式
             * 1 和 2
             */
            Ux.dgDebug({}, "[Ox] Container, 纯渲染，不带状态，传入Props：", "#4666e4");
            return _xuiUniform(children);
        }
    } else {
        /**
         * 使用 xuiControl 方式渲染，如果不是数组的时候需要转换
         */
        if (!U.isArray(grid)) {
            grid = [grid];
        }
        const {$metadata = {}, id, $container, data} = reference.props;
        const {control = {}} = $metadata;
        const prefix = `ox-grid-${id}`;
        const ui = Fn.toGrid(grid, prefix, control);
        /**
         * 容器属性处理
         */
        const inherit = Fn.inheritContainer($container);
        if (state) {
            /**
             * 带状态的容器 或 组件
             */
            const state = _xuiInherit(reference);
            Object.assign(inherit, state);
            Ux.dgDebug({
                name,
                config,
                data,
                inherit
            }, "[Ox] Container, 连接Control渲染，带状态。", "#4666e4");
        } else {
            /**
             * 不带状态的容器或组件
             */
            Ux.dgDebug({
                name,
                config,
                data
            }, "[Ox] Container, 连接Control渲染，不带状态。", "#4666e4");
        }
        return Rdr.xuiGrid(ui, UI, {
            $data: data,
            inherit
        })
    }
};
```



