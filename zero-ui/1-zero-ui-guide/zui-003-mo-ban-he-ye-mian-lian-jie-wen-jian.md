# 模板和页面连接文件

Zero UI中会通过目录来生成固定的路由，并且注入到`react-router`中去，形成固定路由处理。

## 1. 举例

比如现在有一个浏览器中打开的地址：[http://localhost:5000/ox/login/index](http://localhost:5000/ox/login/index)，在Zero中这个URL地址具有不同的意义。

| 项目 | 说明 |
| :--- | :--- |
| localhost | HOST环境变量配置，主机地址 |
| 5000 | PORT环境变量配置，当前应用端口 |
| ox | Z\_ROUTE环境变量配置，不同的应用使用的环境变量不一样，通过这种方式可支持不同的企业用户使用的根路径不同。 |
| login/index | src/components/login/index/UI.js入口文件渲染出来的页面。 |

## 2. 结构

Zero UI中的前端页面主要包含两部分：

* src/container/login/index/UI.js：当前页面使用的模板文件；
* src/components/login/index/UI.js：当前页面核心文件；

在Zero UI中一个页面主要分为两部分：

![](/assets/images/zui/003/container-component.png)

实际上模板是多个页面可共享的文件，而页面文件是每个地址独有的文件，通过这种方式就可以将一个完整的应用呈现出来了。

## 3. 连接

那么如果当前页面需要使用其他模板如何处理，或者说如果一个应用使用了多套模板如何来处理呢？

### 3.1. 配置文件

Zero UI中只支持二级目录结构的页面，也就是说：`/ox/`下边只有两级，分别对应module和page两个基本概念，模板和页面的关系定义在下边文件中：

```shell
src/route.json
```

上述文件内容如下：

```json
{
    "defined": "_module_page",
    "special": {
        "_login_index": [
            "_login_index"
        ],
        "_module_page": [
            "_module_page"
        ]
    }
}
```

### 3.2. 约定说明

上述`route.json`中有两部分内容

* defined：定义了当前应用的默认模板，上边的定义片段模板位置存储于：`src/container/module/page/`目录中。
* special：定义当前应用的扩版模板，这个数据是一个JsonObject对象，key是模板名称，而value是一个数组，包含了当前模板下运行的所有页面。

所以上述的Json代码其实可以简化成：

```json
{
    "defined": "_module_page",
    "special": {
        "_login_index": [
            "_login_index"
        ]
    }
}
```

由于/module/page这个页面同样适用了/module/page这个模板，而默认模板使用的是/module/page，所以可直接简化，也就是说模板名称只是一种约定，/login/index这个页面也可以使用其他的模板如/module/page，最终看开发人员如何设计。

## 4. 自动路由规则

那么最后讲解一下最终的约定说明：

| 约定 | 说明 |
| :--- | :--- |
| 模板目录约定 | 所有的模板文件放在src/container/下 |
| 页面目录约定 | 所有的页面文件放在src/components/下 |
| 入口文件名约定 | 不论模板还是页面文件名固定为UI.js |
| 目录层级约定 | 在src/container和src/components下的页面文件必须是二级路径，二级路径足够表达出很多不同种类的模块和页面了 |
| 变量名约定 | 书写route.json中的模板和页面名称时，一律使用：\_&lt;module&gt;\_&lt;page&gt;的语法，如上边所示 |



