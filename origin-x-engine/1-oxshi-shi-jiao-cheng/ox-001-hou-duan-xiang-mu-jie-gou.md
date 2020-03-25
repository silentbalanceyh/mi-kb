# 项目结构说明

前端项目结构参考：[ZUI-001 Zero UI项目结构说明](/zero-ui/1-zero-ui-guide/zui-001-zero-uixiang-mu-jie-gou-shuo-ming.html)

>  xx 在文中代表客户名称，infix-hy 项目中的内容因客户不同而有所差异

## 1. 数据库

| 数据库名称 | 备注 |
| :--- | :--- |
| DB\_ORIGIN\_X | 元数据仓库 |
| DB\_IOP | 业务数据库 |
| DB\_IOP\_HIS | 业务历史归档库 |

此处说明一下三个数据库的不同使用场景：

* 元数据仓库中不包含配置项数据，但是它包含了业务场景需要使用的
  * 账号/员工数据（对应ITSM中的operator/contacts）
  * 组织架构数据（公司、员工、部门、组、客户、档案信息）
  * 系统配置数据（应用、数据源、模块、字典/分类数据）
* 每次升级的时候，除开上边的这些数据需要执行备份以外，其他的配置信息都可以直接更新，元数据仓库中的内容可以直接通过Excel文件/Json配置重建。
* 业务数据库中存储了客户环境中的配置项数据、关系数据。
* 业务历史归档库是插件`zero-history`提供的功能，用于备份当前系统中的所有删除历史归档信息，`ox-engine`可以通过删除的历史归档数据对业务数据库执行还原操作。

## 2. 项目结构

### 2.1. 基础项目

| 项目名称 | 备注 |
| :--- | :--- |
| vertx-zero | Zero Framework基础框架：[http://www.vertxup.cn](http://www.vertxup.cn) |
| ox-engine | Ox Engine核心项目 |

### 2.2. Ox项目

| 父项目 | 子项目 | 备注 |
| :--- | :--- | :--- |
| ox-core | ox-cmdb | CMDB配置管理平台 |
|  | ox-platform | ox-engine通用资源管理平台 |
| ox-driver | ix-atlantic | 客户应用入口 |
| ox-plugin/infix-db | infix-mysql | MySQL数据库实现 |
|  | infix-oracle | Oracle数据库实现 |
| ox-plugin/infix-hy | hy-xx-down | xx下层模型层 |
|  | hy-xx-up | xx上层插件、业务层 |
| ox-plugin/infix-tp | infix-aiops | 统一事件管理 |
|  | infix-automate | 自动化巡检 |
|  | infix-bastion | 堡垒机 |
|  | infix-cmdb-v1 | CMDB V1 对接模块 |
|  | infix-ddi | DDI |
|  | infix-itsm | （标准化）ITSM对接模块 |
|  | infix-srm | SRM存储管理 |
|  | infix-ucmdb | （标准化）UCMDB对接模块 |

### 2.3. Zero Extension扩展模块

| 项目名称 | 备注 | 是否使用 |
| :--- | :--- | :--- |
| zero-ambient | X\_ 全局环境（应用、数据源、字典） | Yes |
| zero-atom | M\_ 动态建模（模型、实体、关系） | Yes |
| zero-crud | 无表（通用CRUD模板） | Yes |
| zero-erp | E\_ 组织架构 | Yes |
| zero-graphic | 图引擎，可能采用neo4j数据库 | Yes |
| zero-jet | I\_ 动态路由（接口、任务、服务） | Yes |
| zero-ke | **【底层】**所有 Extension 的底层项目 | Yes |
| zero-lbs | L\_ Location专用 | No |
| zero-quiz | 测试框架 | No |
| zero-rbac | 安全管理 | Yes |
| zero-router | 微服务 Api Gateway | No |
| zero-ui | UI\_ 界面配置模块 | Yes |

## 3. 配置文件说明

> 配置文件主管整个应用的所有配置信息，由于是配置和数据驱动，所以配置文件部分属于最核心的部分，配置文件全部位于主项目的 resources 文件夹中：ix-atlantic/src/main/resources

### 3.1. 主配置

> 主配置为 zero 核心框架的相关配置！

| 配置文件 | 备注 |
| :--- | :--- |
| vertx.yml | zero 框架主配置 |
| vertx-elasticsearch.yml | ES主配置 |
| vertx-excel.yml | Excel导入/导出主配置 |
| vertx-extension.yml | zero 扩展模块主配置 |
| vertx-inject.yml | zero 依赖注入配置 |
| vertx-jet.yml | 动态路由主配置 |
| vertx-job.yml | 任务管理主配置 |
| vertx-jooq.yml | 数据库主配置 |
| vertx-readible.yml | 可视化容错主配置 |
| vertx-secure.yml | 安全管理主配置 |
| vertx-shared.yml | 缓存主配置 |
| logback.xml | logback 日志主配置 |
| keys/keystore.jceks | 安全管理专用证书配置 |
| codex | 前端接口验证规则配置目录 |

### 3.2. 扩展配置

> 扩展配置为 zero-extension 的主配置

| 目录 | 子目录/文件 | 备注 |
| :--- | :--- | :--- |
| plugin | channel.json | zero-extension 中的核心通道专用配置，用于底层连接各个模块之间的松散插件结构。 |
| plugin/ambient | configuration.json | zero-ambient 系统环境主配置 |
|  | todo | 待办事项主配置 |
| plugin/atom | configuration.json | zero-atom 动态建模主配置 |
| plugin/crud | configuration.json | zero-crud 增删查改主配置 |
|  | module | 各个模块的crud模块配置 |
| plugin/rbac | configuration.json | zero-rbac 安全管理设置主配置 |
| plugin/ui | configuration.json | zero-ui 界面程序主配置 |
|  | column | zero-ui 中的"动态列"专用配置 |
| cab/cn | components | vertx-ui 中 X\_MODULE 主配置 |
|  | extension | vertx-ui 中 extension 扩展模块核心前端配置 |

### 3.3. Ox核心配置

| 目录 | 子目录/文件 | 备注 |
| :--- | :--- | :--- |
| migration | XXXXXXXX（日期格式） | 一键升级程序中的备份主目录，存在于 .gitignore 文件配置中。 |
| init | job | 多环境任务专用配置 |
|  | oob | 元数据目录（初始化专用） |
|  | database.yml | 元数据库建库 liquibase 配置 |
|  | liquibase.properties | 元数据库数据库配置 |
| runtime | configuration.json | CMDB二期平台主配置 |
|  | adjuster | CMDB二期数据修正专用配置，UCMDB字段特殊处理，建模特殊处理。 |
|  | console | 命令行工具专用配置 |
|  | cmdb-v2 | UCMDB核心通道统一配置 |
|  | excel/schema | 动态建模源文件 |
|  | external | UCMDB核心任务/接口专用配置 |
|  | imported | 初始化导入文件 |
|  | json/model | 建模模型Json源文件 |
|  | json/schema | 建模实体Json源文件 |

### 3.4. Ox插件配置

> 注：所有分开模块中的 cmdb-v2 目录是这个模块对接 CMDB 二期平台的专用对接配置，每个模块对应的配置会有所不同。

多环境集成配置中目前主要包含：

* zw：招为专用外网环境
* dev：开发测试环境
* home：本地调试环境（个人专用）
* prod：生产环境

| 项目 | 配置目录/文件 | 备注 |
| :--- | :--- | :--- |
| infix-mysql | engine/database/sql/MYSQL5 | 动态建模DDL层核心配置：MySql |
| infix-oracle | engine/database/sql/ORACLE12 | 动态建模DDL层核心配置：Oracle |
| hy-xx-up | init | xx 专用数 liquibase 配置 |
|  | plugin/xx/annal | logback 日志管理接入配置，xx表示客户项目名称 |
| infix-aiops | plugin/aiops/annal | logback 事件工单管理日志配置 |
|  | plugin/aiops/cmdb-v2/ | CMDB二期对接配置 |
|  | plugin/aiops/data | 事件工单管理平台辅助配置 |
| infix-automate | plugin/automate/annal | logback 自动化巡检日志配置 |
|  | plugin/automate/cmdb-v2/ | CMDB二期对接配置 |
| infix-bastion | plugin/bastion/annal | logback 堡垒机日志配置 |
|  | plugin/bastion/cmdb-v2/ | CMDB二期对接配置 |
|  | plugin/bastion/data | 堡垒机辅助配置 |
|  | plugin/bastion/configuration.json | 堡垒机主配置 |
|  | plugin/bastion/integration\*.json | 多环境集成配置 |
| infix-cmdb-v1 | plugin/cmdb-v1/annal | logback CMDB一期日志配置 |
|  | plugin/cmdb-v1/integration\*.json | CMDB一期集成配置 |
| infix-ddi | plugin/ddi/annal | logback DDI日志配置 |
|  | plugin/ddi/configuration.json | DDI主配置 |
|  | plugin/ddi/integration\*.json | 多环境集成配置 |
| infix-itsm | plugin/itsm/annal | logback ITSM日志配置 |
|  | plugin/itsm/cmdb-v2 | CMDB二期对接配置 |
|  | plugin/itsm/data/ | ITSM辅助配置 |
|  | plugin/itsm/configuration.json | ITSM主配置 |
|  | plugin/itsm/integration\*.json | 多环境集成配置 |
| infix-srm | plugin/srm/annal | logback SRM存储管理日志配置 |
|  | plugin/srm/cmdb-v2 | CMDB二期对接配置 |
|  | plugin/srm/configuration.json | SRM主配置 |
|  | plugin/srm/integration\*.json | 多环境集成配置 |
| infix-ucmdb | plugin/ucmdb/annal | logback UCMDB日志配置 |
|  | plugin/ucmdb/cmdb-v2/ | CMDB二期对接配置 |



