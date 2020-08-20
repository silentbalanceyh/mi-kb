# GraphicEditor

　　GraphicEditor是g6图编辑器组件，用户绘制复杂图专用，基本属性如下：

| 配置项 | 是否支持 | 说明 |
| :--- | :--- | :--- |
| @zero绑定 | 不支持 | 不和任何生命周期绑定 |
| Mount | 不支持 | 不支持 componentDidMount |
| Update | 不支持 | 不支持 componentDidUpdate |
| Ex | 不支持 | 不支持 $ready，同样不支持 yoRender |

## 1. props属性说明

| 属性 | 说明 |
| :--- | :--- |
| items | 绘制板上拖拽数据 |
| data | 图专用数据，包含了 nodes, edges |
| current | 当前节点专用数据（自由Json结构 ）|
| config | 当前组件专用配置 |
| graphConfig | 图专用配置 |
| submitting | 是否处于提交状态 |
| executor | 和 Editor 绑定的 Command 命令外置函数 |
| context | 右键菜单，node, edge, canvas, group 四种 |

## 2. 数据结构

### 2.1. items 结构

