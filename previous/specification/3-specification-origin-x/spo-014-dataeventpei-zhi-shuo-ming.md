# DataEvent结构说明

在Origin X中，所有的事件都是以DataEvent对象发送给目标对象的，该对象定义在`src/entity/data/DataEvent.ts`文件中，为组件通信的核心对象。

## 1. 数据结构

关于DataEvent数据结构可以参考下图：

![](/assets/images/spo/014/data-event-structure.png)

## 2. 说明

* 每一个发送的DataEvent都有一个时间名称`name`，可直接通过`key()`的API进行获取。
* 每个DataEvent的名称name必须以`on`开头，其他名字会直接被Origin X忽略掉。
* 同一个事件DataEvent可以进行复用，每一次调用`mount(params)`的时候该事件中的数据会被重置，然后调用链也会被清空。
* DataEvent对象中除了params（每次mount的当前事件的数据）以外，还包含两个核心对象：
  * source：事件源，所有属性都是`$`符号开头。
  * target：事件目标对象，所有的属性都是以`_`符号开头。
* DataEvent对象中包含了一个变量`returns`，它表示每个事件每一步触发的Behavior行为之后的返回结果，其中包含了很多中间结果，方便开发人员调试，这个变量可以理解为调用栈。
* DataEvent对象中还包含了一个state对象，它表示不论事件如何触发，最终会生成一个完整的state对象，只要该对象不为空，那么会调用React组件中的`setState`方法去更新组件状态（该组件一般是根组件：$container）。



