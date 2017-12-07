### 一、数据库操作
1、查看所有数据库

```
show databases;
```
2、创建数据库

```
### utf-8 字符集数据库
create database 数据库名称 default charset utf8 collate utf8_general_ci;

### gdk 字符集数据库
create database 数据库名称 default character set gbk collate gbk_chinese_ci;
```
3、使用数据库（切换到想使用的数据库）

```
use 数据库名称;

### 查看当前使用数据库的所有表
show tables;

### 查看当前正在使用的数据库
show tables; 
select database(); 
status; 
```
4、用户管理

```
### 创建用户
create user '用户名'@'IP地址' identified by '密码';

### 删除用户
drop user '用户名'@;'IP地址';

### 修改用户名
rename user '用户名'@'IP地址' to '新用户名'@'IP地址';

### 修改密码
set password for '用户名'@'IP地址' = password('新密码');

### 查询所有的用户信息
select * from mysql.user;

PS:所有的用户信息都保存在mysql库中的user表中
```
5、用户授权管理

```
### 查看用户权限
show grants for '用户'@'IP地址';

### 对用户进行授权
grant 权限 on 数据库.表 to '用户'@'IP地址';

### 取消用户权限
revoke 权限 on 数据库.表 from '用户'@'IP地址';

```
#### 具体权限介绍

权限 | 权限级别 | 权限说明
------|-------|-------
all privileges | 数据库和表 | 除grant外所有的权限
create | 数据库、表或者索引 | 创建数据库、表或者索引权限
drop   | 数据库和表         | 删除数据库和表权限
grant option | 数据库、表或者保存过程 | 赋予权限选项
reference | 数据库和表
alter  | 表 | 更改表，比如添加字段、索引等
delete | 表 | 删除数据权限
index  | 表 | 索引权限
insert | 表 | 插入权限
select | 表 | 查询权限
update | 表 | 更新权限
create view | 视图 | 创建视图权限
show view | 视图 | 查看视图权限
alter routine | 存储过程 | 更改存储过程权限
create routine | 存储过程 | 创建存储过程权限
execute | 存储过程 | 执行存储过程权限
file    | 服务器主机上的文件访问 | 文件访问权限
create temporary tables | 服务器管理 | 创建临时表权限
lock tables | 服务器管理 | 锁表权限
create user | 服务器管理 | 创建用户权限
process     | 服务器管理 | 查看进程权限
reload      | 服务器管理 | 执行flush-hosts, flush-logs, flush-privileges, flush-status, flush-tables, flush-threads, refresh, reload等命令的权限
repliaction client | 服务器管理 | 服务器位置的访问权限
replication slave  | 服务器管理 | 由复制从属使用权限
show databases | 服务器管理 | 查看数据库权限
shutdown | 服务器管理 | 关闭数据库权限
super    | 服务器管理 | 执行kill线程权限

##### MYSQL的权限如何分布的呢，就是针对表可以设置什么权限，针对列可以设置什么权限等......

权限分布 | 可能的设置的权限
---|---
表权限 | 'select','insert','update','delete','create','drop','crant','references','index','alter'
列权限 | 'select','insert','update','references'
过程权限 | 'execute','alter','routine','grant'

##### 对数据库和用户名权限

权限 | 权限说明
---|---
数据库名.* | 数据库中的所有
数据库名.表 | 指定数据库中的某张表
数据库名.存储过程 | 指定数据库中的存储过程
* . * | 所有数据库
用户名@IP地址  | 用户只能在改IP下才能访问
用户名@192.168.1.%  |  用户只能在改IP段下才能访问(通配符%表示任意)
用户名@%     | 用户可以再任意IP下访问(默认IP地址为%)


##### MySQL权限经验原则:
权限控制主要是出于安全因素，因此需要遵循一下几个经验原则：
```
1、只授予能满足需要的最小权限，防止用户干坏事。比如用户只是需要查询，那就只给select权限就可以了，不要给用户赋予update、insert或者delete权限。

2、创建用户的时候限制用户的登录主机，一般是限制成指定IP或者内网IP段。

3、初始化数据库的时候删除没有密码的用户。安装完数据库的时候会自动创建一些用户，这些用户默认没有密码。

4、为每个用户设置满足密码复杂度的密码。

5、定期清理不需要的用户。回收权限或者删除用户。
```
##### 做一些关于上面权限的示例

```
grant all privileges on db1.tb1 TO '用户名'@'IP'

grant select on db1.* TO '用户名'@'IP'

grant select,insert on *.* TO '用户名'@'IP'

revoke select on db1.tb1 from '用户名'@'IP'

### 如果想要把所执行的sql语句立即生效需要执行下面的语句(将数据读取到内存中，从而立即生效)
flush privileges;
```

##### MySQL忘记密码解决方法：

```
### 启动免授权服务端
mysqld --skip-grant-tables

### 重新打开个shell终端执行
mysql -uroot -p

### 修改用户名密码
update mysql.user set authentication_string=password('brian') where user='root';
flush privileges;
```

### 二、数据表操作
1、创建表

```
### 首先要先切进创建表的库中
use 数据库名;

### 创建表
create table 表名(
    列名  类型  是否可以为空,
    列名  类型  是否可以为空
)engine=innodb default charset=utf8;
```

##### 我们接下来就来具体的举例创建表时候的类型和一个相关参数
MySQL中的数据类型大概分为：数值、时间、字符串三种

```
bit[(M)]  二进制位(101001),m表示二进制位的长度(1-64) ,默认m=1

tinyint[(M)] [unsigned] [zerofill] 小整数:数据类型用于保存一些范围的整数数值范围(有符号：-128~127，无符号：0~255，特别的：MySQL没有布尔值，使用tinyint(1)构造)

int[(M)] [unsigned] [zerofill] 整数:数据类型用于保存一些范围的整数数值范围(有符号:-2147483648~2147483647,无符号:0~4294967295,特别的：整数类型中的m仅用于显示，对存储范围无限制。例如： int(5),当插入数据2时，select 时数据显示为： 00002)

bigint[(M)] [unsigned] [zerofill] 大整数，数据类型用于保存一些范围的整数数值范围(有符号:-9223372036854775808~9223372036854775807,无符号:0~18446744073709551615)

decimal[(M[,D])] [unsigned] [zerofill] 准确的小数值，m是数字总个数（负号不算），d是小数点后个数。 m最大值为65，d最大值为30(特别的：对于精确数值计算时需要用此类型decaimal能够存储精确值的原因在于其内部按照字符串存储)

float[(M[,D])] [unsigned] [zerofill] 单精度浮点数（非准确小数值），m是数字总个数，d是小数点后个数(无符号:-3.402823466E+38 to -1.175494351E-38,0,1.175494351E-38 to 3.402823466E+38,有符号:0,1.175494351E-38 to 3.402823466E+38)

double[(M[,D])] [unsigned] [zerofill] 双精度浮点数（非准确小数值），m是数字总个数，d是小数点后个数(无符号:-1.7976931348623157E+308 to -2.2250738585072014E-308,0,2.2250738585072014E-308 to 1.7976931348623157E+308 有符号:0,2.2250738585072014E-308 to 1.7976931348623157E+308)

PS:**** 数值越大，越不准确 ****

```


```
char(M) char数据类型用于表示固定长度的字符串，可以包含最多达255个字符。其中m代表字符串的长度(PS: 即使数据小于m长度，也会占用m长度)

varchar(M) varchars数据类型用于变长的字符串，可以包含最多达255个字符。其中m代表该数据类型所允许保存的字符串的最大长度，只要长度小于该最大值的字符串都可以被保存在该数据类型中(注：虽然varchar使用起来较为灵活，但是从整个系统的性能角度来说，char数据类型的处理速度更快，有时甚至可以超出varchar处理速度的50%。因此，用户在设计数据库时应当综合考虑各方面的因素，以求达到最佳的平衡)

text text数据类型用于保存变长的大字符串，可以组多到65535 (2**16 − 1)个字符

```

```
enum 枚举类型:一个ENUM列最多可以有65535个不同的元素。 （实际限制不足3000条）

set 集合类型:SET列最多可以有64个不同的成员

date YYYY-MM-DD（1000-01-01/9999-12-31）

time HH:MM:SS（'-838:59:59'/'838:59:59'）

year YYYY（1901/2155）

datetime   YYYY-MM-DD HH:MM:SS（1000-01-01 00:00:00/9999-12-31 23:59:59）

timestamp YYYYMMDD HHMMSS（1970-01-01 00:00:00/2037 年某时）

```
以上就是MySQL中的所有数据类型了，下面来说一下具体的一些相关参数


```
### 是否为空(非字符串)
null --可以为空
not null --不可以为空

### 默认值(创建列时可以指定默认值，当插入数据时如果没有主动设置，则自动添加默认值)
示例：
create table 表名(
    nid int not null defalut 2,
    num int not null
);

### 自增(如果为某列设置自增列，插入数据时无需设置此列，默认将自增（表中只能有一个自增列）)
示例：
create table 表名(
    nid int not null auto_increment primary key,
    num int null
);
或者
create table 表名(
    nid int not null auto_increment,
    num int null,
    index(nid)
);
PS:1、对于自增列，必须是索引(包含主键)
   2、对于自增可以设置步长和起始值(语句如下)
        show session variables like 'auto_inc%';  # 查询
        set session auto_increment_increment=2; # 修改
        set session auto_increment_offset=10;   # 修改
        
        show global variables like 'auto_inc%';  # 查询
        set global auto_increment_increment=2; # 修改
        set global auto_increment_offset=10;   # 修改
        
### 主键(一种特殊的唯一索引，不允许有空值，如果主键使用单个列，则它的值必须唯一，如果是多列，则其组合必须唯一)
示例:
create table 表名(
    nid int not null auto_increment primary key,
    num int null
);
或者
create table 表名(
    nid int not null,
    num int null,
    primary key(nid,num)
);

### 外键(一个特殊的索引，只能是指定内容)
示例:
create table color(
    nid int not null primary key,
    name char(16) not null
);


create table fruit(
    nid int not null primary key,
    smt char(64) null,
    color_id int not null,
    constraint fk_cc foreign key (color_id) references color(nid)
);

```

2、删除表

```
drop table 表名;
```

3、清空表

```
delete from 表名;
truncate table 表名;
```

4、修改表


```
### 添加列
alter table 表名 add 列名 类型;

### 删除列
alter table 表名 drop column 列名;

### 修改列
alter table 表名 modify column 列名 类型;   # 修改类型
alter table 表名 change 原列名 新列名 类型; # 修改列名、类型

### 添加主键
alter table 表名 add primary key(列名);

### 删除主键
alter table 表名 drop primary key;
alter table 表名  modify  列名 int, drop primary key;

### 添加外键
alter table 从表 add constraint 外键名称（形如：FK_从表_主表） foreign key 从表(外键字段) references 主表(主键字段);

### 删除外键
alter table 表名 drop foreign key 外键名称；

### 修改默认值
alter table 表名 alter i set default 1000;

### 删除默认值
alter table 表名 alter i drop default;
```



### 三、表内容的操作：
在MySQL中对表内容的操作无非就是增、删、改、查、其他等操作方式，下面就做下具体的示例介绍(PS：前提是要先进入数据库创建完数据表才能做下面的操作)

1、增
```
### 插入单条数据 列名和值是对应的
insert into 表名 (列名，列名....) values (值，值....);

### 插入多条数据 列名和值是对应的
insert into 表名 (列名，列名....) values (值，值....)(值，值....);

### 复制表数据
insert into 表名 (列名，列名....) select (列名，列名....) from 表名;

```
2、删

```
### 删除表里面的所有数据
delete from 表名;

### 删除表中的指定数据
delete from 表名 where id＝1 and name＝'brian';
```
3、改

```
update 表名 set name ＝ 'brian' where id>1;
```
4、查

```
### 查整个表
select * from 表名;

### 查表中id大于1的
select * from 表名 where id > 1;

### 查询表中列中id大于1
select nid,name,gender as gg from 表 where id > 1;
```

5、其他

```
- 通过条件判断查询数据
    select * from 表名 where id > 1 and name != 'brian' and num = 12;
    select * from 表名 where id between 5 and 16;
    select * from 表名 where id in (11,22,33);
    select * from 表名 where id not in (11,22,33);
    select * from 表名 where id in (select nid from 表名);
    
- 通配符查询数据
    select * from 表名 where name like 'bri%';  - bri开头的所有（多个字符串）
    select * from 表名 where name like 'bri_';  - bri开头的所有（一个字符）
    
- 对查询数据进行限制
    select * from 表名 limit 5;            - 前5行
    select * from 表名 limit 4,5;          - 从第4行开始的5行
    select * from 表名 limit 5 offset 4;    - 从第4行开始的5行
    
- 对查询数据进行排序
    select * from 表名 order by 列 asc;              - 根据 “列” 从小到大排列
    select * from 表名 order by 列 desc;             - 根据 “列” 从大到小排列
    select * from 表名 order by 列1 desc,列2 asc;    - 根据 “列1” 从大到小排列，如果相同则按列2从小到大排序
    
- 分组查询数据
    select num from 表名 group by num;
    select num,nid from 表名 group by num,nid;
    select num,nid from 表名  where nid > 10 group by num,nid order nid desc;
    select num,nid,count(*),sum(score),max(score),min(score) from 表名 group by num,nid;
    select num from 表名 group by num having max(id) > 10;
    
    PS：特别的：group by 必须在where之后，order by之前
    
- 连表操作
    ### 无对应关系则不显示
    select A.num, A.name, B.name from A,B Where A.nid = B.nid;
    
    ### 无对应关系则不显示
    select A.num, A.name, B.name from A inner join B on A.nid = B.nid;
    
    ### A表所有显示，如果B中无对应关系，则值为null
    select A.num, A.name, B.name from A left join B on A.nid = B.nid;
    
    ### B表所有显示，如果B中无对应关系，则值为null
    select A.num, A.name, B.name from A right join B on A.nid = B.nid;
    
- 组合操作
    ### 组合，自动处理重合
    select nickname
    from A
    union
    select name 
    from B
    
    ### 组合，不处理重合
    select nickname
    from A
    union all
    select name
    from B
```
[具体连表实例](http://www.cnblogs.com/brianzhu/)