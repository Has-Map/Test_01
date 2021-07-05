#### oracle安装

#####注意事项

- 计算机名尽量避免中文
- 安装路径避免中文和特殊字符
- windows登录用户名避免中文和特殊字符
- **断网**
- 关掉其他软件
- **关掉防火墙和杀毒软件**

#####开始安装

- 按照要求，一路next即可

#### oracle用户

##### oracle自带用户

- sys：超级用户

- system：系统管理员用户

- scot：示范用户（测试用户）


权限：sys > system > scott

安装完成之后scott用户是被锁住的，要想使用scott用户必须先解锁该用户

```sql
--解锁scott用户语句
--首先登录system用户
alter user scott account unlock;
```

##### 常用系统预定义角色

- connect：临时用户
- resource：更为可靠和正式的用户
- dba：数据库管理员角色，拥有数据库管理的最高权限

##### 用户登录方式

- **sqlplus指令**：sqlplus 用户名/密码@数据库实例名（sqlplus workit/workit@oracle10）
- sqlplusw指令：sqlplusw桌面客户端连接oracle服务
- isqlplus指令：浏览器访问方式http://localhost:5560/isqlplus
- **第三方工具**：pl/sql

##### 创建自定义用户

```sql
--创建workit用户
--登录system用户
create user workit			--用户名
identified by workit		--登陆密码
default tablespace users    --默认表空间
temporary tablespace temp;  --临时表空间
--新创建的用户没有任何权限（啥事都干不了），需要对新创建用户进行授权操作
--在学习阶段为了图方便，我们一般对新建用户都是授予dba权限，但是实际工作中是
--干什么操作，就给相应权限。
--授予权限/角色命令 语法：grant 权限/角色名称 to 用户名
grant dba to workit;
--撤销权限/角色命令 语法：revoke 权限/角色名称 from 用户名
revoke dba from workit;
--连接用户命令
conn workit/workit@oracle10;
```

#### oracle表对象

数据：就是现实世界的信息的表征符号（数值，字符，图像，视频，二进制·····）

数据在数据库中存储需要用一个结构来存储

数据有典型性的特征，每个数据都有一些特性。二维的关系来表示，行 表示一个个体（记录）列 表示一个数据的一个特征。

- 表table

  oracle通过表来设计个体的属性，表示属性的二维关系。信息就是对应的记录，一行就是一个个体。

#### oracle处理表对象

- 定义表对象
  1. DDL（Data Define Language）语句sql（Structure Query Language结构化查询语言）的基本数据定义语言
  2. 表事先不存在的时候，需要这个对象，就必须用DDL的**create table**指令来创建表结构（用户必须有create table 权限，且必须有对表对象存储的空间有使用权限-->resource）
  3. 表的结构必须涵盖数据的基本属性的描述。列（字段）数据特征（数据类型）
- 处理数据
  - 对结构下处理信息，需要调用sql的DML（Data Makeup Language）语句来操作数据
    - insert ：插入数据
    - delete：删除数据
    - update：更新数据
    - select ：查询数据

#### oracle数据类型

- 数值型
  - **number[(p[,s])]**：age number(3);	//-999~999		sal number(8,2)	//900000.00

- 字符型

  - char：定长的字符串，非常死不会根据输入的字符串自动适应字符串的长度（资源没有最大化的利用-->占着茅坑不拉屎）	name char(10);	//'文理'	此时name的长度为10
  - varchar：不定长的字符串           name varchar(10);        //'文理'        此时name的长度为2
  - **varchar2**：不定长的字符串        name varchar2(10);        //'文理'        此时name的长度为2

  **总结**：varchar和varchar2之间区别就是最大存储长度不一样；varchar最多可存储2000，varchar2最多可存   储4000；实际使用过程以varchar2为主

- 日期型

  - **data**	精确到毫秒	2021-07-04 19:42:38 103	sysdate
  - timestamp    精确到微秒

- 大对象型

  - blob	二进制大对象
  - **clob**    字符大对象	

- 二进制型

  - binary

#### oracle建表语句

```sql
create table t_student(
stdid number(8) primary key,			--主键ID
stdname varchar2(10) not null,			--姓名
origin varchar2(20) not null,			--籍贯
birthday date,							--出生日期
score number(3) not null);				--成绩
```

#### oracle修改表结构

```sql
--更改t_student表的score为number(5,2)
alter table t_student modify score number(5,2);
--添加sex列
alter table t_student add sex char(2) not null;
--删除sex列
alter table t_student drop column sex;
```

#### oracle表结构DDL指令小结

```sql
create table						--创建新的表
alter table 表名 modify 字段··		 --修改列
alter table 表名 add 字段··			 --添加列
alter table 表名 drop column		   --删除列
drop table 表名	--删除表（灾难性：结构和数据都被删除)
--删除表后需要用purge recyclebin来清空回收站
purge recyclebin;
```

####oracle表数据处理小结（DML）

- **insert**
  - insert into 表名 values(·····);
  - insert into 表名(字段名列表,····) values(val1,val2,···);  /字段名列表与插入的数据要一一对应
  - not null非空字段，必须要插入数据；

- **delete**
  - delete 表名 [条件]
  - delete 语句谨慎使用（执行delete语句后，若没有执行事务控制语句（TCL）那么删除的数据还可以恢复，否则无法恢复）；
- **update**
  - update 表名 set 字段 = value [条件]
  - update 表名 set 字段1 = value1,字段2 = value2,····[条件;is null（空值判断）]
- **select**
  - select *from 表名;	//全字段全表检索查询

#### oracle克隆表

- **克隆表**

  ```sql
  --必须有访问scott用户表对象的权限
  create table t_dept
  as
  select *from scott.dept;
  ```

- 克隆表结构，不要表数据

  ```sql
  create table t_dept
  as
  select *from scott.dept
  where 1 = 2;
  ```

#### oracle中sql语句类型

- DDL——Data Define Language数据定义语言
  - create新建对象：create user;	create table;	create or replace view····
  - alter修改对象：alter user;    alter table;······
  - drop删除对象：drop user;    drop table;·····
- DML——Data Makeup Language数据操作语言
  - insert添加数据
  - delete删除数据
  - update修改数据
  - select查询数据
- TCL——Transaction Control Language事务控制语言
  - commit提交
  - rollback取消回滚
  - savepoint设置回滚点
- DCL——Data Control Language数据控制语言
  - grant····to 授权
  - revoke·····from 撤销

#### 经典面试题

**oracle中TCL（事务控制语言）语句**

- 概念：将对oracle的每条语句都当做一个原子事务，oracle编译引擎会自动执行，但是不会最终与oracle明确（确认，回退、回滚、取消）
- 问题：insert，delete，update操作（sql语句），对数据库表有影响。数据库退出后，再次登陆访问，数据没有影响。
- 对事务控制
  - 确定提交：commit
  - 回滚取消：rollback
  - 设置回滚点：savepoint
- **注意**
  - DDL语句是自动事务处理，无需人为干涉。
  - 只有影响**记录结果集的DML语句**（insert，delete，update）才需要人为的事务控制。

**oracle中DCL（数据控制语言）语句**

- 概念：针对不同的用户，反馈其所有权限的数据访问；可以一定限度保障数据的安全；
- 常见的DCL指令：
  - 授予权限：grant 权限 to 用户
  - 撤销权限：revoke 权限 from 用户
- **注意**
  - 角色权限高的用户可以直接访问角色权限低的用户数据
  - 角色权限低的用户访问角色权限高的用户数据的时候，需要数据控制

#### oracle中的delete、truncate、drop的区别

- delete：**删除表记录**，可以被回滚取消
- truncate：**截断表**，属于DDL语句，保留表结构，去掉表中所有数据，不可以被回滚取消
- drop：**删除表**，属于DDL语句，结构和数据都会被删除，且不能被回滚

#### oracle中建表约束

- 常规建表语句（熟练）

- 现实建表，考虑具体的业务逻辑要求，需要根据业务逻辑要求，创建相应的约束

  - **not null** 字段非空

  - **default** 指定默认值

    ```sql
    pwd varchar2(20) default 'workit',
    ```

  - **unique** 唯一值

  - **check** 检查值满足条件

    ```sql
    sex char(2) check (sex in ('男'，'女')),
    ```

  - **primary key** 主键

  - **foreign key** 外键

    ```sql
    tipid number(8) not null,
    foreign key(tipid) references 表名(tipid)
    --上述表名为从哪个表中引用的外键的表名
    ```

