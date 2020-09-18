# 通道开发

>  通道开发是主体部分，直接上代码和注释，xx 在文中代表客户名称。

## 1. 接口开发

### 1.1. 标准接口

通道名称：`cn.originx.optic.component.CreateComponent`

```java
package cn.originx.optic.component;

import cn.originx.scaffold.component.AbstractAdaptor;
import io.vertx.core.Future;
import io.vertx.up.commune.ActIn;
import io.vertx.up.commune.ActOut;
import io.vertx.up.commune.Record;

/*
 * 1）传入数据，直接插入
 * 2）请求：POST /api/ox/:identifier
 */
public class CreateComponent extends AbstractAdaptor {
    @Override
    public Future<ActOut> transferAsync(final ActIn request) {
        final Record record = this.activeRecord(request);
        return this.dao().insertAsync(record)
                .compose(ActOut::future);
    }
}
```

其他接口参考：`cn.originx.optic.component`包中的接口开发。

### 1.2. UCMDB通道接口

通道名称：`cn.originx.optic.advanced.CreateComponent`

通道配置：

```json
{
    "configuration.operation": "ADD",
    "plugin.component.before": [
        "cn.originx.uca.plugin.semi.BeforeNumber",
        "cn.originx.uca.plugin.semi.BeforeLife"
    ],
    "plugin.component": "cn.originx.scaffold.plugin.AspectRecord",
    "plugin.component.after": [
        "cn.originx.uca.plugin.semi.AfterEs",
        "cn.originx.itsm.plugin.AfterItsm"
    ],
    "plugin.config": {
        "cn.originx.uca.plugin.semi.BeforeNumber": {
            "field": "code"
        }
    },
    "plugin.activity": "cn.originx.xx.extension.AspectActivity"
}
```

通道代码：

```java
package cn.originx.optic.advanced;

import io.vertx.tp.atom.modeling.data.DataAtom;
import io.vertx.core.Future;
import io.vertx.core.json.JsonObject;
import io.vertx.up.atom.record.Atomy;
import io.vertx.up.commune.ActIn;
import io.vertx.up.eon.em.ChangeFlag;
import io.vertx.up.unity.Ux;

/*
 * 1）传入数据，直接插入
 * 2）请求：POST /api/ox/:identifier
 */
public class CreateComponent extends AbstractUcmdb {
    /*
     * data -> request.getJObject()
     * 传入的 data 为新数据
     */
    @Override
    public Future<JsonObject> transferAsync(final Atomy atomy, final ActIn request,
                                            final DataAtom atom) {
        return Ux.<JsonObject>future(atomy.data())

                /* 1.压缩数据 */
                .compose(data -> Ux.future(this.compress(data, atom)))

                /* 2.UCMDB推送 */
                .compose(compressed -> this.out(atom).procAsync(compressed, ChangeFlag.ADD))

                /* 3.写入数据库 */
                .compose(this.completer(atom)::create)

                /* 4.生成变更历史 */
                .compose(newRecord -> this.trackAsyncC(newRecord, atom));
    }
}
```

### 1.3. ITSM通道接口

通道名称：`cn.originx.itsm.api.OnlineComponent`

通道配置：

```json
{
    "configuration.operation": "UPDATE",
    "plugin.component.before": [
        "cn.originx.ucmdb.plugin.semi.BeforeUcmdb",
        "cn.originx.xx.extension.BeforeOwner"
    ],
    "plugin.component": "cn.originx.scaffold.plugin.AspectBatch",
    "plugin.component.after": [
        "cn.originx.uca.plugin.semi.AfterEs"
    ],
    "plugin.activity": "cn.originx.xx.extension.AspectActivity"
}
```

通道代码：

```java
package cn.originx.itsm.api;

import cn.originx.cv.OxCv;
import cn.originx.cv.em.LifeCycle;
import cn.originx.uca.workflow.Commutator;
import io.vertx.core.json.JsonArray;
import io.vertx.core.json.JsonObject;
import io.vertx.tp.ke.cv.KeField;
import io.vertx.up.commune.ActIn;

public class OnlineComponent extends AbstractItsmCi {
    @Override
    public JsonObject toCondition(final JsonObject condition, final ActIn request) {
        /*
         * 查询满足条件的配置项，默认为 []
         * 1. codes 在传入的范围内 []
         * 2. confirmStatus 必须是 confirmed
         * 3. lifecycle 必须满足条件
         */
        condition.put(OxCv.Field.CONFIRM_STATUS, Commutator.UNLOCK_VALUE);

        /*
         * lifecycle 条件设置
         * - INNER（CMDB内部，可以直接升级为 ITSM 工作流）
         * - READY（可上线的状态，直接处理上线
         */
        final JsonArray lifecycle = new JsonArray();
        lifecycle.add(LifeCycle.INNER.name());
        lifecycle.add(LifeCycle.READY.name());

        condition.put(OxCv.Field.LIFE_CYCLE + ",i", lifecycle);
        condition.put(KeField.GLOBAL_ID + ",!n", Boolean.TRUE);
        return condition;
    }

    @Override
    public JsonObject toData(final JsonObject dataBody, final ActIn request) {
        /*
         * 更新数据设置
         * 1）lifecycle 修改成 ONLINE
         * 2）confirmStatus 修改成 confirmed（双重保证）
         */
        dataBody.put(OxCv.Field.LIFE_CYCLE, LifeCycle.ONLINE.name());
        dataBody.put(OxCv.Field.CONFIRM_STATUS, Commutator.UNLOCK_VALUE);
        return dataBody;
    }
}
```

## 2. 任务开发

### 2.1. UCMDB主任务

#### 前置Income

前置组件名称：`cn.originx.ucmdb.component.UcmdbIncome`

前置组件代码：

```java
package cn.originx.ucmdb.component;

import cn.originx.scaffold.component.AbstractIncome;
import cn.originx.ucmdb.atom.UcmdbBuffer;
import cn.originx.ucmdb.atom.UcmdbIntegration;
import cn.originx.ucmdb.refine.Ud;
import cn.originx.ucmdb.service.UcmdbFetcher;
import io.vertx.core.Future;
import io.vertx.core.json.JsonArray;
import io.vertx.tp.atom.modeling.data.DataAtom;
import io.vertx.up.commune.Envelop;
import io.vertx.up.commune.config.DualItem;
import io.vertx.up.unity.Ux;

public class UcmdbIncome extends AbstractIncome {

    @Override
    public Future<Envelop> beforeAsync(final Envelop envelop) {
        /*
         * 接口初始化
         */
        final UcmdbIntegration integration = UcmdbIntegration.createdOn(this.integration());
        final UcmdbFetcher stub = UcmdbFetcher.get(integration);
        /*
         * 先调用标识规则选择器读取 identifier
         */
        final DataAtom atom = this.atom();
        /*
         * 查找 CI 项，之后查找 Relations （配置项、关系）
         */
        final DualItem mapping = this.mapping().child().bind(this.atom().types());
        final UcmdbBuffer buffer = new UcmdbBuffer(this.fabric.createCopy(mapping));
        buffer.connect(integration);

        return stub.bind(buffer).fetchCis(atom.identifier())
                .compose(cis -> stub.fetchRelations(atom.identifier())
                        /*
                         * 合并相关信息
                         */
                        .compose(relations -> {
                            final JsonArray result = Ud.combineCiData(cis, relations, integration.getKeyJoined());
                            /*
                             * 切换后的最终数据处理
                             */
                            Ud.infoJob(this.getClass(), "[ Ud ] 最终数据：{0}", result.encode());
                            return Ux.future(result);
                        })
                )
                .compose(dataArray -> this.output(dataArray, envelop));
    }
}

```

#### 主通道代码

通道名称：`cn.originx.ucmdb.component.UcmdbActor`

通道配置：

```json
{
    "plugin.activity": "cn.originx.xx.extension.AspectActivity",
    "plugin.todo": "cn.originx.xx.extension.AspectTodo"
}
```

通道代码：

```java
package cn.originx.ucmdb.component;

import cn.originx.scaffold.component.AbstractActor;
import cn.originx.uca.commerce.Linker;
import cn.originx.ucmdb.refine.Ud;
import io.vertx.core.Future;
import io.vertx.core.json.JsonArray;
import io.vertx.core.json.JsonObject;
import io.vertx.tp.atom.modeling.data.DataAtom;
import io.vertx.tp.modular.dao.AoDao;
import io.vertx.up.commune.ActIn;
import io.vertx.up.commune.ActOut;
import io.vertx.up.commune.config.Identity;
import io.vertx.up.util.Ut;

/*
 * Ucmdb 主动同步任务（定义配置处理）
 */
public class UcmdbActor extends AbstractActor {
    @Override
    public Future<ActOut> transferAsync(final ActIn request) {
        final JsonArray data = request.getJArray();
        if (!data.isEmpty()) {
            /*
             * 读取当前系统里所有这一类的数据相关信息
             * 这里需要特殊说明，数据本身格式如：
             * [] - JsonArray
             * 这里不使用 atom() / dao() 两个核心方法
             */
            final DataAtom atom = this.atom();
            final String identifier = atom.identifier();
            Ud.infoJob(this.getClass(), "[ Ud ] 执行内部逻辑！{0}，数据：{1}", 
                    identifier, data.encode());
            final Linker linker = Linker.get(identifier);
            Ut.contract(linker, DataAtom.class, this.atom());
            Ut.contract(linker, AoDao.class, this.dao());
            Ut.contract(linker, JsonObject.class, this.options());
            Ut.contract(linker, Identity.class, this.identity());
            return linker.saveAsync(data).compose(ActOut::future);
        } else {
            /*
             * 没有读取到任何数据信息
             */
            Ud.infoJob(this.getClass(), "[ Ud ] 没有任何数据信息！");
            return ActOut.future(new JsonArray());
        }
    }
}
```

### 2.2. ITSM同步任务

通道名称：`cn.originx.itsm.component.AccountActor`

通道代码：

```java
package cn.originx.itsm.component;


import cn.originx.itsm.service.AccountRunner;
import cn.originx.itsm.service.Coordinator;
import cn.originx.itsm.service.PreIn;
import cn.originx.refine.Ox;
import cn.vertxup.erp.service.EmployeeService;
import cn.vertxup.erp.service.EmployeeStub;
import io.vertx.core.Future;
import io.vertx.core.json.JsonArray;
import io.vertx.core.json.JsonObject;
import io.vertx.tp.ke.cv.KeField;
import io.vertx.up.atom.Refer;
import io.vertx.up.commune.ActIn;
import io.vertx.up.commune.ActOut;
import io.vertx.up.unity.Ux;
import io.vertx.up.util.Ut;

import java.util.Objects;
import java.util.Set;

public class AccountActor extends AbstractItsmJob {
    private transient final EmployeeStub employeeStub =
            Ut.singleton(EmployeeService.class);
    private transient Coordinator coordinator;

    @Override
    protected Coordinator stub() {
        if (Objects.isNull(this.coordinator)) {
            this.coordinator = new AccountRunner(this.integration());
        }
        return this.coordinator;
    }

    @Override
    public Future<ActOut> transferAsync(final ActIn request) {
        /*
         * 1. PreIn 专用
         */
        final PreIn pre = this.pre();
        Ox.Log.infoUca(this.getClass(), "准备员工数据：type = {0}", 
                                        pre.getClass().getName());
        /*
         * 2. 读取新数据
         */
        final Refer current = new Refer();
        return this.stub().fetchAsync()
                /*
                 * 3. 准备数据处理
                 */
                .compose(pre::readyAsync)
                .compose(current::future)
                /*
                 * 4. 新数据
                 */
                .compose(this::fetchOriginal)
                .compose(original -> this.runAsync(original, current.get(), KeField.WORK_NUMBER))
                .compose(ActOut::future);
    }

    private Future<JsonArray> fetchOriginal(final JsonArray input) {
        final Set<String> numbers = Ut.mapString(input, KeField.WORK_NUMBER);
        final JsonObject condition = new JsonObject();
        condition.put("workNumber,i", Ut.toJArray(numbers));
        return this.employeeStub.fetchAsync(condition)
                .otherwise(Ux.otherwise(JsonArray::new));
    }
}

```
