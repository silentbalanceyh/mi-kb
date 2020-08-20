# 接口权限配置

>  所有表结构解析中不包含 zero extension 扩展模块中的系统型字段，主要是：`sigma, active, language, metadata, createdAt, createdBy, updatedAt, updatedBy`八个系统型字段。
> 所有的配置导入都在`ox-driver/ix-atlantic`项目中执行（主程序）

本文介绍以下几个内容：

* 如何配置新权限？
* 如何配置新资源？

## 1. 权限表结构

权限表配置包含了两张主表：`S_PERM（权限），R_ROLE_PERM（连接表）`。

参考下表定义先理解 `S_PERM` 和 `R_ROLE_PERM` 的表结构。

### 1.1. S_PERM

| 字段名 | 列名 | 含义 | 值格式 |
| :--- | :--- | :--- | :--- |
| key | KEY | 权限主键 | 新生成UUID |
| name | NAME | 权限名称 | 中文名称 |
| comment | COMMENT | 权限备注 | 权限的描述信息 |
| code | CODE | 权限编码 | 以`perm`开头带`.`的权限编码 |

### 1.2. R_ROLE_PERM

| 字段名 | 列名 | 含义 | 值格式 |
| :--- | :--- | :--- | :--- |
| roleId | ROLE_ID | 关联角色主键 | UUID格式 |
| permId | PERM_ID | 关联权限主键 | UUID格式 |

## 2. 资源表结构

资源配置包含了三张主表：`S_VIEW（视图）、S_RESOURCE（资源）、S_ACTION（操作）`，在配置接口时，需要先配置最少两张：**资源表和操作表**。视图（`S_VIEW`）表是高级权限配置，在本文中不涉及。

参考下表定义先理解 `S_RESOURCE` 和 `S_ACTION` 的表结构。

### 2.1. S_RESOURCE

| 字段名 | 列名 | 含义 | 值格式 |
| :--- | :--- | :--- | :--- |
| key | KEY | 资源主键 | 新生成UUID |
| name | NAME | 资源名称 | 中文格式 |
| code | CODE | 资源编码 | 以`res`开头带`.`的资源编码 |
| level | LEVEL | 资源级别 | 资源对Action要求的级别（整数）|
| modelRole | MODEL_ROLE | 角色模式 | 四个值：UNION / EAGER / LAZY / INTERSECT |
| modelGroup | MODEL_GROUP | 组模式 | 三个值：HORIZON / CRITICAL / OVERLOOK |
| modelTree | MODEL_TREE | 树模式 | 四个值：PARENT / CHILD / INHERIT / EXTEND |

配置资源的时候，`modelGroup`和`modelTree`是可选的。

### 2.2. S_ACTION

| 字段名 | 列名 | 含义 | 值格式 |
| :--- | :--- | :--- | :--- |
| key | KEY | 操作主键 | 新生成UUID |
| uri | URI | URI地址 | URI路径全地址，从`/api/`开始 |
| method | METHOD | HTTP方法 | 四个值：GET / POST / PUT / DELETE |
| name | NAME | 操作名称 | 中文格式 |
| code | CODE | 操作编码 | 以`act`开头带`.`的操作编码 |
| level | LEVEL | 操作级别 | 当前Action所属级别（整数）|
| renewalCredit | RENEWAL_CREDIT | 副作用模式 | **保留配置** |
| resourceId | RESOURCE_ID | 关联资源ID | 1.1 中配置的资源`KEY` |
| permissionId | PERMISSION_ID | 所属权限ID | 新创建的权限`KEY` |

## 3. 配置步骤

```shell
# 安装UUID生成工具，主键生成使用该配置
npm install -g uuid
```

### 3.1. 创建配置文件

1. 收集访问接口的账号和密码（`S_USER`中的`USERNAME`和`PASSWORD`字段）。
2. 执行下边SQL语句（`DB_ORIGIN_X`元数据库中执行），根据`USERNAME`搜索该用户关联的角色，其中`xxx`就是账号信息：

    ```sql
    SELECT ROLE_ID,PRIORITY FROM R_USER_ROLE 
        WHERE USER_ID IN 
            (SELECT `KEY` FROM S_USER WHERE USERNAME='xxx') 
    ORDER BY PRIORITY DESC;
    ```    
1. 在查询结果中可查询到账号对应的角色ID如：653c2705-5b37-4e66-8085-fc69d076e301
2. 在`src/main/resources/init/oob/`中创建新的API所需`Excel`文件（可参考目前的`xxx-api.xlsx`类型的），文件名尽可能使用可识别的文件前缀，方便单独导入。

### 3.2. 配置规则注意

1. `S_PERM，S_ACTION，S_RESOURCE`遵循统一命名规范编码，编码在系统中不可重复
    1. 权限：`perm.xxx`
    2. 操作：`act.xxx`
    3. 资源：`res.xxx`
2. `S_ACTION`中的 uri 和 method 必须和您将开发的接口匹配，可参考目前的配置，这个配置一定不能出错。
3. `S_ACTION`中的`level`必须大于或等于`S_RESOURCE`中的`level`，推荐使用`等于`的配置。
4. 几个关联关系一定不可以出错
    1. `R_ROLE_PERM`：角色和权限的关联
    2. `S_ACTION`：权限/资源/操作三者之间通过操作进行关联
5. 注意所有表中的`sigma, active, language`都不要填错，参考已有配置

### 3.3. 导入配置

1. 停止容器
2. 修改 `OxDevelop`代码如下，首参不变，第二参使用您创建的文件前缀：

    ```java
    package cn.vertxup;
    
    import io.vertx.tp.ke.booter.Bt;
    
    public class OxDevelop {
    
        public static void main(final String[] args) {
            Bt.doImports("init/oob/", "第二参");
        }
    }
    ```
1. 设置`OxDevelop`的执行配置，参考环境配置，然后执行该程序，将数据重新导入
2. 检查您的权限是否已经进入到系统，查询几张表中的相关信息
3. 重启容器，使用 `Postman` 测试

## 4. 总结

按照上述步骤，接口权限就配置完成了。

