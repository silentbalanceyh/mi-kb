# 回到始源之地

*人可以一辈子不去登山，可是心里一定要有座山，有了这座山，你才不会迷失自己的方向，任何时候，抬起头来就可以看到自己的希望。  
　　——2002年 随风杨 赠。*

我们总是不知道自己要什么，同样，我们也不清楚生活的目标究竟在哪，于是视之为迷茫。很多时候，在前行途中，我们看多了风景，引起了视觉的疲劳，回过头来却忘记了自己的初衷。开设这个项目的目的只是为了搭建自己的知识库，也可以称为个人知识库，从Zero（[http://www.vertxup.cn](http://www.vertxup.cn)）到Zero UI（[http://www.vertxui.cn](http://www.vertxui.cn)）开源也快有一年的时间了，各种文档错综复杂，少了脉络，使得读者略显疲乏，于是重新整理一份完整的知识库，系统通过整理，将真正有价值的东西呈现出来。

* [Zero教程](https://lang-yu.gitbook.io/zero/)（Gitbook维护中，等结果）
* [Vert.x逐陆记](https://lang-yu.gitbook.io/vert.x/)

> 文档列表参考下方链接。

## 1. 环境准备

Origin X Doc是为Origin X Engine量身打造的文档型项目，使用`gitbook`可直接本地化成电子书方便查阅，不仅仅如此，本文档中收录了更多在开发过程中的心得以及环境搭建过程中遇到的种种问题，希望真正对读者有所帮助。

### 1.1. 本地化

在自己的本机使用下边命令安装一个静态服务器，推荐使用 `serve`

```shell
sudo npm install -g serve
```

然后执行根目录的Shell脚本：

```shell
./book-server.sh
```

用浏览器打开地址：[http://localhost:1231/](http://localhost:1231/)，您就可以访问本地化的 Origin X Doc的主页了。

### 1.2. 在线版

在线的版本发布在新的域名下，地址：[http://www.origin-x.cn](http://www.origin-x.cn)，这里枚举当前系统关联的所有框架主页：

| 地址 | 说明 |
| :--- | :--- |
| [http://www.vertxup.cn](http://www.vertxup.cn) | 「后端」Zero框架主页 |
| [http://www.vertxui.cn](http://www.vertxui.cn) | 「前端」Zero UI框架主页 |
| [http://www.vertxai.cn](http://www.vertxai.cn) | 「工具」Zero 自动化工具主页 

> 由于文档的工作量巨大，大部分文档都是整理的以前的原创，再加上整个几个项目都是由我一个人主体维护，大家都是采取业余时间在处理整个体系的文档以及开源代码，所以有些文档可能会出现残章以及不全的地方，请读者见谅。

## 3. 总结

我希望通过 `Origin X Doc`这个项目将所有目前手中的资料串联起来，统一化管理，主要目标如下：

* Zero的官方文档大部分是代码、缺乏思路，对于“攻击型“的开发者，完全足够用了，但对于”防守型“的开发者，很多时候不太理解里面的一些思路，所以本系列文档希望通过更加原生态的说明来解析开发过程中的一些相关信息。
* Zero UI的官方文档原始版本还比较好，后期采用了所见即所得的版本，对很多开发人员可能比较噩梦，虽然我们已经在至少三个生产系统中使用了两者，但更多缺乏的是教程式的文档。

### 写在篇末

Origin X名字来源于伊苏起源，那是一个让你去追溯真理和源头的故事，跟着所有的线索，冲破重重困难，真正去找寻最终答案的故事。X表示“未知”，整个项目的名字来源于三毛的《梦里花落知多少》的序：“从生的生、到死的死、从已知到未知、从未知到已知、爱的神秘、灵魂的离奇、宇宙中透露着的是层层的迷。”，所以Origin X的名字含义很简单——探索未知世界的起源，是否存在某种可以统一的“理”（并不是脑袋一拍想出来的）。Zero名字来源于高达W中的“飞翼零式”，实际上深层次的含义是：空杯为零，具有一个空杯心态，一切以此为始，真正让系统达到“无胜于有”的境界，让开发人员深度去思考这个世界，看是否可以透过双手，一切从零做起，逐步去影响这个世界。

## 4. 文档列表

### 4.1. 环境搭建部分

#### 1.依赖环境

* [1.1. Neo4j环境](/environment/environment-dependency/001.neo4j.md)
* [1.2. TiDB环境](/environment/environment-dependency/002.tidb.md)
* [1.3. Elastic Search环境](/environment/environment-dependency/003.es.md)
* [1.4. Etcd本地集群搭建](/environment/environment-dependency/004.etcd.md)
* [1.5. Maven环境搭建](/environment/environment-dependency/005.maven.md)
* [1.6. Istio环境搭建](/environment/environment-dependency/006.istio.md)
* [1.7. K8s：HA Etcd集群](/environment/environment-dependency/007.etcd-ha.md)
* [1.8. K8s：HA集群](/environment/environment-dependency/008.k8s.md)
* [1.9. K8s：认证](/environment/environment-dependency/009.k8s-token.md)
* [1.10. K8s：kubeadm配置](/environment/environment-dependency/010.k8s-kubeadm.md)

#### 2.Zero环境

* [2.1. ox-ui环境搭建](/environment/environment-zero/001.ox-ui.md)
* [2.2. 仓库初始化](/environment/environment-zero/002.initialize.md)
* [2.3. zero-ui工程初始化](/environment/environment-zero/003.zero-ui.md)
* [2.4. zero-ai的安装](/environment/environment-zero/004.zero-ai.md)
* [2.5. zero-ui工程初始化（ai命令）](/environment/environment-zero/005.ai-initialize.md)

#### 3.其他环境

* [3.1. Spring Boot中的数据源配置](/environment/environment-thirdpart/001.spring-boot-ds.md)
* [3.2. Karma在ES6中的Webpack配置](/environment/environment-thirdpart/002.karma-es-webpack.md)
* [3.3. Karma的安装教程](/environment/environment-thirdpart/003.karma-installing.md)
* [3.4. Windows环境的Docker安装](/environment/environment-thirdpart/004.docker-windows.md)
* [3.5. 使用Docker环境运行Spring Boot](/environment/environment-thirdpart/005.docker-spring-boot.md)

### 4.2. Origin X Engine产品文档

#### 0. 标准化安全接口

* [1.基本接口说明](/origin-x-engine/3.ox-interface/oi-001-ji-ben-jie-kou-shuo-ming.md)
* [2.认证授权接口](/origin-x-engine/3.ox-interface/oi-002-ren-zheng-shou-quan.md)
* [3.资源请求接口](/origin-x-engine/3.ox-interface/oi-003-zi-yuan-qing-qiu-chu-shi-hua.md)

#### 1. 实施开发（旧版）

* [1.项目结构说明](/origin-x-engine/1.ox-delivery-guide/ox-001-structure-project.md)
* [2.环境搭建](/origin-x-engine/1.ox-delivery-guide/ox-002-environment-backend.md)
* [3.代码结构](/origin-x-engine/1.ox-delivery-guide/ox-003-structure-code.md)
* [4.生产环境升级](/origin-x-engine/1.ox-delivery-guide/ox-004-environment-production.md)
* [5.通道架构](/origin-x-engine/1.ox-delivery-guide/ox-005-structure-channel.md)
* [6.核心数据结构](/origin-x-engine/1.ox-delivery-guide/ox-006-structure-data.md)
* [7.Options配置](/origin-x-engine/1.ox-delivery-guide/ox-007-delivery-options.md)
* [8.任务通道详解](/origin-x-engine/1.ox-delivery-guide/ox-009-channel-task.md)
* [9.第一步：任务和接口配置](/origin-x-engine/1.ox-delivery-guide/ox-009-step1-configuration.md)
* [10.第二步：通道开发](/origin-x-engine/1.ox-delivery-guide/ox-010-step2-channel.md)
* [11.第三步：插件开发](/origin-x-engine/1.ox-delivery-guide/ox-011-step3-plugin.md)
* [12.第四步：最终测试](/origin-x-engine/1.ox-delivery-guide/ox-012-step4-testing.md)
* [13.通道详细说明](/origin-x-engine/1.ox-delivery-guide/ox-013-index.md)
    * [13.1.标准通道](/origin-x-engine/1.ox-delivery-guide/ox-013-channel-standard.md)
    * [13.2.CMDB通道](/origin-x-engine/1.ox-delivery-guide/ox-013-channel-cmdb.md)
    * [13.3.待确认通道](/origin-x-engine/1.ox-delivery-guide/ox-013-channel-confirm.md)
    * [13.4.集成通道](/origin-x-engine/1.ox-delivery-guide/ox-013-channel-integration.md)
* [14.接口权限配置](/origin-x-engine/4.ox-authorization/os-001-authorization.md)
* [15.RBAC模型](/origin-x-engine/4.ox-authorization/os-002-rbac.md)
* [16.数据库列类型](/origin-x-engine/2.ox-modeling/om-001-column-type.md)
    * [16.1.MySQL定义](/origin-x-engine/2.ox-modeling/om-001-database/om-001-1-mysql.md)
    * [16.2.Oracle定义](/origin-x-engine/2.ox-modeling/om-001-database/om-001-2-oracle.md)
* [17.表单录入基本参考](/origin-x-engine/2.ox-modeling/om-002-form-field.md)
* [18.模型检查表](/origin-x-engine/2.ox-modeling/om-003-model-checking.md)
* [19.业务层 Mapping](/origin-x-engine/3.ox-interface/oi-004-ye-wu-ceng-mapping-pei-zhi-shuo-ming.md)
* [20.业务层 Dict](/origin-x-engine/3.ox-interface/oi-005-ye-wu-ceng-dict.md)
* [21.业务层 Identity](/origin-x-engine/3.ox-interface/oi-006-ye-wu-ceng-identity.md)

#### 2. 新版文档（实战为主）

* [1.升级运行要点](/origin-x-engine/5.ox-document/001-ucmdb.md)
* [2.新测试框架的使用](/origin-x-engine/5.ox-document/002-unit-testing.md)
