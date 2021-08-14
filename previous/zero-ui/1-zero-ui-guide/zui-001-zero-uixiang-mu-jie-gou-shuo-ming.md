# Zero UI项目结构说明

本文对Zero UI脚手架进行拆解说明，让开发人员清楚该脚手架的内容以及相关细节，Zero UI脚手架采取了“组件化”方式对项目文件进行组织，使用了Ant-Design-Pro/Ant-Design中的组件，但并没使用分层结构，所有的`Reducer, Action`全部内置在当前组件自身的目录中。目前尚未升级到最新版的依赖库：

* antd，由于 antd 从 3.0 到 4.0 存在很大程度的界面修改，目前最新版本使用：`3.26.12`。
* @antv/g6，编辑器修改于 gg-editor 库中的核心源代码，暂时不升级最新，目前使用：`3.2.10`。

## 1. 目录结构

> 只有标记为【开发】的目录会被开发人员自我管理，其他目录基本不用改变，标记为【Zero】部分为Zero UI提供的常用组件以及相关开发包，不提供给开发人员扩展。

| 主目录 / 子目录 / ... | 特殊文件 | 说明 |
| :--- | :--- | :--- |
| .zero |  | Zero AI默认配置文件目录，和 [http://www.vertxai.cn](http://www.vertxai.cn) 结合使用时的配置工作目录，自动化开发核心目录。 |
| build |  | 生产环境build的输出目录，可配置在服务器中运行，不提交到 github，发布时的输出目录。 |
| config | variables.js | 环境变量连接文件，用于连接不同环境变量用于docker容器化和k8s集群专用。 |
| config | modules.js | 模块相对路径处理专用导入脚本，解决不同模块之间的import相对路径导入问题。 |
| config | webpack.config.dev.js | 开发环境webpack配置 |
| config | webpack.config.prod.js | 生产环境webpack配置 |
| public |  | 静态资源文件包，publicUrl 地址 |
| scripts |  | 原生启动脚本 |
| shell |  | Zero专用启动脚本：环境变量初始化，代码链接，脚本执行，路由生成，容器配置检查，启动 |
| shell / tpl / route | routes.zt | Zero UI中的路由模板，路由连接器依赖该模板生成详细的react-router路由连接文件 |
| shell | run-package.json | 依赖库升级限制专用包管理配置文件 |
| src / app |  | 【开发】App专用目录 |
| src / cab |  | 【开发】多国语言包资源文件 |
| src / components |  | 【开发】Page页面组件 |
| src / container |  | 【开发】Layout模板组件 |
| src / econnomy |  | 【Zero】可重用组件 |
| src / editor |  | 【Zero】g6 编辑器专用目录 |
| src / entity |  | 【Zero】TypeScript数据对象 |
| src / environment |  | 核心环境文件 |
| src / environment | actions.js | Redux中的Action连接文件 |
| src / environment | combiner.js | Redux中的Reducer合并器 |
| src / environment | datum.js | 【生成】该文件运行时会由 run-zero.sh 脚本生成，主要用于链接组件（components）和容器（container）自身的 Epic/Types 定义 |
| src / environment | reducers.js | 统一调用的抽象Reducer方法，一般会映射到核心修改状态树的dispatcher中。 |
| src / environment | routes.js | 【生成】系统会自动生成该路由文件，整个应用的路由连接器就是从这里作为起点链接 react-router的路由信息。 |
| src / environment | zero.js | @zero注解的定义文件 |
| src / extension |  | Zero Extension 扩展模块（包含了大部分企业应用的 OOB 形态） |
| src / mock |  | 新版 Zero UI 中的 mock 数据文件 |
| scr / plugin |  | 【开发】插件专用目录 |
| src / style |  | css相关风格文件 |
| src / ux |  | 【Zero】Utility X包，纯函数主入口 |
| stories |  | Storybook专用包 |
| .babelrc |  | Babel配置文件 |
| .env.development |  | 开发环境环境变量文件 |
| .env.production |  | 生产环境环境变量文件 |
| .eslintrc.js |  | Eslint配置文件 |
| .gitignore |  | Git Ignore专用文件 |
| package.json |  | NPM包配置 |
| package-lock.json |  | （略） |
| README.md |  | （略） |
| run.sh |  | 生产环境运行脚本 |
| run-deploy.sh |  | 打包部署脚本 |
| run-update.sh |  | 依赖库升级脚本 |
| run-ux.sh |  | 环境升级脚本 |
| run-ux-current.sh |  | 环境升级主执行脚本 |
| run-zero.bat、run-zero.sh |  | Zero UI启动脚本 |
| SUMMARY.md |  | （略） |
| tsconfig.json |  | TypeScript配置文件 |
| yarn.lock |  | （略） |
| yuidoc.json |  | YUIDOC配置文件 |

虽然上边文件和目录结构复杂，但实际上开发人员仅需要关注标记了【开发】的目录即可，不同项目的【开发】部分略有差异。

## 2. Zero UI框架的设计理念

看过《人月神话》的软件工程师心里都清楚，一个软件项目的成败除了人本身的用心，更多的时候在于一个“节奏”感，疯狂的加班不一定导致最终高质量的成品，一个人的英雄主义也拯救不了太复杂的工程粒度，而我们往往过于乐观，很多时候觉得只要有技术，好像技术就是一切，无所不能，并且错误地以为甲方所要求的都是追求高精尖的东西。这么多年以来，我细细思索，其实不是这个道理——技术最好的方式是点到即止恰如其分地描述了现实世界，更多的时候我们不是在开发一个完美的东西，相反，我们做的是一个瑕不掩瑜的八十分的作品。

Zero UI的出现不在我的意料之中，原本打算开发了Zero（[http://www.vertxup.cn](http://www.vertxup.cn)）过后就休息一段时间，但随着项目的压力，铺面而来的项目需求如同寒风中的冷刀，一刀刀割着肌肤，才发现原来仅仅有后端是不够，还需要一个完整的“外壳“，因为我们的用户只能直观去理解“看得见”的东西。既然Zero可以从某种层面上解决工程师的效率、犯错的问题，那么Zero UI的基本设计同样如此。

