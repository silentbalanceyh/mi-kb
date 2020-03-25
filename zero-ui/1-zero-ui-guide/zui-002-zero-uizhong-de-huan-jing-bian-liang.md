# Zero UI中的环境变量

Zero UI中使用了一部分系统环境变量用于指定当前Zero UI运行的宿主的一些信息，这些环境变量可以在最终打包成Docker镜像时直接提供，并且可以进行很灵活的全局化配置。

## 1. 配置文件

Zero UI中包含两套环境变量文件：

| 文件名 | 环境变量 |
| :--- | :--- |
| .env.development | 开发环境 Development |
| .env.production | 生产环境 Production |

> Zero UI中提供的所有dg前缀方法都只有在 development 开发环境中生效，也就是说，所有console中的日志信息只有开发环境会被启用，如果是生产环境这些日志将不会被打印，若要在生产环境进行脚本调试可直接使用 console 来打印日志。

## 2. 变量使用

所有Zero UI的环境变量都必须以`Z`开头，并且可以通过 `Ux.Env`直接去掉`Z`过后使用，如定义了环境变量：

```shell
Z_APP=ox.vie.app
```

那么在代码中可直接使用：

```js
import Ux from "ux";
import React from "react";

class A extends React.PureComponent{
    render(){
        const app = Ux.Env.APP; // 直接引用环境变量
        return false;
    }
}
```

## 3. 变量列表

| 外部环境变量 | 脚本中的引用方法 | 备注 |
| :--- | :--- | :--- |
| PORT | 无 | 当前App运行的端口号 |
| HOST | 无 | 当前App运行的host地址 |
| Z\_TITLE | Ux.Env.TITLE | 当前应用运行的标题，HTML中的&lt;header&gt;部分 |
| Z\_APP | Ux.Env.APP | 当前App的名称，该名称为系统唯一标识符，和配置中心可连接 |
| Z\_ENDPOINT | Ux.Env.ENDPOINT | 当前App需要连接的远程后端EndPoint地址，前后端分离架构下专用 |
| Z\_LANGUAGE | Ux.Env.LANGUAGE | 当前App运行的语言标识符，该语言标识符会对应到cab包中的目录，如默认语言为：cn，则所有资源文件目录位于：src/cab/cn/目录下 |
| Z\_ROUTE | Ux.Env.ROUTE | 当前App的动态路由根路径，不同的应用该值应该设置为不同，所有的React Router的路径都是放在该变量下运行 |
| Z\_K\_SESSION | 无（用于构造Session专用） | 当前App在使用SessionStorage时对应的Key前缀，默认使用@@ZUI/，使用前缀可在同一个浏览器中登陆不同的App应用且不会有数据冲突 |
|  | Ux.Env.KEY\_APP | 当前应用保存的SessionStorage |
|  | Ux.Env.KEY\_USER | 当前应用保存的用户专用Session |
| Z\_K\_EVENT | Ux.Env.KEY\_EVENT | 当前App使用的Redux状态时候的事件前缀，用于区分不同Redux行为专用，默认值为@@ZUI-ACT |
| Z\_DEV\_DEBUG | Ux.Env.DEBUG | 是否开启Debug模式，Debug模式中才可看见对应的日志信息 |
| Z\_DEV\_MOCK | Ux.Env.MOCK | 是否打开全局的Mock功能，如果打开就可以支持Mock分离于后端的开发模式 |
|  | Ux.Env.HTTP11 | HTTP1.1的头文件常量 |
|  | Ux.Env.HTTP\_METHOD | Http方法常量 |
|  | Ux.Env.MIMES | 常用MIME映射文件 |
| Z\_DEV\_FORM |  | 监控表单渲染的专用变量，牵涉到表单的布局渲染流程 |
| Z\_DEV\_MONITOR | Ux.Env.MONITOR | 是否启用Zero UI中支持的原生监控工具，会有一个监控工具栏出现在浏览器的下方。 |
| Z\_DEV\_AJAX | Ux.Env.DEBUG\_AJAX | 是否将Ajax请求保存成Json文件格式，如果调试模式打开，Request请求将保存成Json格式，每次请求都会存储一次。 |
| Z\_CORS\_MODE | Ux.Env.CORS\_MODE | 跨域的基本模式，对应到fetch中的选项Option的值：cors，no-cors，或者提供相关的域信息。 |
| Z\_CORS\_CREDENTIALS | Ux.Env.CORS\_CREDENTIALS | 对应Options中的credentials选项，包括include, omit，或者提供相关的域信息。 |
| Z\_SIGN | Ux.Env.SIGN | 是否启用数字签名功能，默认false |
| Z\_CSS\_PREFIX | Ux.Env.CSS\_PREFIX | 当前站点的风格文件专用前缀设置，该设置需要定义Less的全局变量@app |
| Z\_CSS\_COLOR | Ux.Env.CSS\_COLOR | 当前站点的主色调，一旦改动了这个颜色过后，所有按钮边框和效果都会被更改，对应到Ant Design中的@primary-color的值。 |
| Z\_SHARED | Ux.Env.SHARED | 全局Epic和Types共享目录名称，默认值为app，所以共享内容位于src/app/action的目录下 |
| Z\_ENTRY\_LOGIN | Ux.Env.ENTRY\_LOGIN | 当前应用的登录首页 |
| Z\_ENTRY\_ADMIN | Ux.Env.ENTRY\_ADMIN | 当前应用的管理首页【带登录控制】，关于管理首页可以直接将该页面开发成根据不同角色的分离器，这样就可以完成应用入口根据不同授权的分离功能。 |
| Z\_X\_HEADER\_SUPPORT |  | 启用 X-Sigma、X-AppId、X-AppKey和 X-Lang实现多租户、多语言、多应用的平台环境。 |

## 4. 设置例子

下边是环境变量文件`.env.development`的例子：

```properties
PORT=5000
REACT_APP_LESS_MODULES=true

Z_LANGUAGE=cn
Z_ENDPOINT=http://localhost:6083
Z_APP=vie.app.ox
Z_ROUTE=ox
Z_SHARED=app

Z_ENTRY_LOGIN=/login/index
Z_ENTRY_ADMIN=/admin/index
Z_K_SESSION=@@OX/
Z_K_EVENT=@@OX-ACT

Z_DEV_DEBUG=true
Z_DEV_MOCK=true
Z_DEV_AJAX=false
Z_DEV_FORM=true
Z_DEV_MONITOR=true

Z_CSS_PREFIX=ox
Z_CSS_COLOR=#3d8ce7
Z_CORS_CREDENTIALS=include

Z_X_HEADER_SUPPORT=true
```

## 5. 打印结果

![](/assets/images/zui/002/ENV.png)

