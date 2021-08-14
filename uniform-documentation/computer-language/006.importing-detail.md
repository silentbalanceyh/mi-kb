# MySQL导入数据详解

本文记录MySQL 5.x和8.x平台的导入过程，仅做记录，参考下边的SQL代码：

* MySQL 客户端 Workbench 8.x 版本
* MySQL 命令行
* MySQL Server 5.x版本

## 0. 表准备

```sql
create table emp(
    empid int primary key auto_increment,
    ename varchar(10) unique,
    job varchar(10) not null,
    mgr int,
    hiredate date,
    sal float default 0,
    comm float,
    deptno int,
    foreign key(deptno) references dept(deptno)
);
```

数据准备：



## 1. SQL脚本如下

```sql

```



