### 一、MySQL中的视图
#### 什么是视图？
    视图是个虚拟的表（非真实存在的）其本质是根据SQL语句获取动态数据集，并为其命名，用户想获取结果只需要使用视图名称来获取结果集，并可以将其当做表来使用
    
1、创建视图

```
### 格式：
create view 视图名称 as SQL语句;

### 示例：
create view vv1 as select students.id,students.name from students where id > 101;
```
2、修改视图

```
### 格式：
alter view 视图名称 as SQL语句;

### 示例：
alter view vv1 as select students.id,students.name,majors.name as majorname from students inner join majors on students.majorid = majors.id;
```
3、使用视图

    使用视图时，将其当作表进行操作即可，由于视图是虚拟表，所以无法使用其对真实表进行创建、更新和删除操作，仅能做查询用.

```
### 格式：
select * from 视图名称;

### 示例：
select * from vv1;
```
4、删除视图

```
### 格式：
drop view 视图名称;

### 示例：
drop view vv1;
```
### 二、触发器
    对某个表进行【增/删/改】操作的前后如果希望触发某个特定的行为时，可以使用触发器，触发器用于定制用户对表的行进行【增/删/改】前后的行为.
1、创建触发器

```
### 格式：
create trigger trigger_name trigger_time trigger_event on tbl_name for each row trigger_stmt;

其中：
trigger_name：标识触发器名称，用户自行指定；
trigger_time：标识触发时机，取值为 BEFORE 或 AFTER；
trigger_event：标识触发事件，取值为 INSERT、UPDATE 或 DELETE；
tbl_name：标识建立触发器的表名，即在哪张表上建立触发器；
trigger_stmt：触发器程序体，可以是一句SQL语句，或者用 BEGIN 和 END 包含的多条语句。

由此可见，可以建立6种触发器，即：BEFORE INSERT、BEFORE UPDATE、BEFORE DELETE、AFTER INSERT、AFTER UPDATE、AFTER DELETE。
另外有一个限制是不能同时在一个表上建立2个相同类型的触发器，因此在一个表上最多建立6个触发器.
```

```
### 创建基础的语法
# 插入前
CREATE TRIGGER tri_before_insert_tb1 BEFORE INSERT ON tb1 FOR EACH ROW
BEGIN
    ...
END

# 插入后
CREATE TRIGGER tri_after_insert_tb1 AFTER INSERT ON tb1 FOR EACH ROW
BEGIN
    ...
END

# 删除前
CREATE TRIGGER tri_before_delete_tb1 BEFORE DELETE ON tb1 FOR EACH ROW
BEGIN
    ...
END

# 删除后
CREATE TRIGGER tri_after_delete_tb1 AFTER DELETE ON tb1 FOR EACH ROW
BEGIN
    ...
END

# 更新前
CREATE TRIGGER tri_before_update_tb1 BEFORE UPDATE ON tb1 FOR EACH ROW
BEGIN
    ...
END

# 更新后
CREATE TRIGGER tri_after_update_tb1 AFTER UPDATE ON tb1 FOR EACH ROW
BEGIN
    ...
END
```
2、触发器实例：
```
创建两个表一个user_info,一个user_infos:

### user_info表创建：
create table user_info(
uid int(5) not null auto_increment,
name varchar(64) not null,
password varchar(255) default null,
email varchar(255) default null,
primary key (uid,name)
)engine=innodb default charset=utf8;

### user_infos表创建：
create table user_infos(
uid int(5) not null auto_increment,
name varchar(64) not null,
password varchar(255) default null,
email varchar(255) default null,
primary key (uid,name)
)engine=innodb default charset=utf8;

### 创建一个插入之前的触发器：
delimiter //
create trigger tri_before_insert_tb1 before insert on user_info for each row
begin
if new.name = "brian" then
insert into user_infos(name,password,email) values (new.name,new.password,new.email);
end if;
end //
delimiter ; (命令和分号间有空格)

### 使用触发器(触发器无法由用户直接调用，而是由于对表的增/删/改操作被动引发的)
往user_info表中插入两条数据:
insert into user_info(name,password,email) values ("jack","jack","jack@126.com"),("brian","brian","brian@126.com");

### 查看表中的内容(对比)
select * from user_info;
select * from user_infos;

```
3、查看触发器(触发器是库级别的所有在查看的时候需要先进入库中)

```
use 数据库名;
show triggers\G;
```

4、删除触发器

```
drop trigger 触发器名称;
```


### 三、存储过程
    我们常用的操作数据库语言SQL语句在执行的时候需要先编译，然后在执行的，而存储过程是一组为了完成特定功能的SQL语句集，经编译后存储在数据库中，用户通过指定存储过程的名字并给定参数(当然也可以不带参数)来调用执行它.
##### 存储过程优点：
    1. 存储过程增强了SQL语言的功能和灵活性。存储过程可以用流控制语句编写，有很强的灵活性，可以完成复杂的判断和较复杂的运算.
    
    2. 存储过程允许标准组件是编程。存储过程被创建后，可以在程序中被多次调用，而不必重新编写该存储过程的SQL语句。而且数据库专业人员可以随时对存储过程进行修改，对应用程序源代码毫无影响.
    
    3. 存储过程能实现较快的执行速度。如果某一操作包含大量的Transaction-SQL代码或分别被多次执行，那么存储过程要比批处理的执行速度快很多。因为存储过程是预编译的。在首次运行一个存储过程时查询，优化器对其进行分析优化，并且给出最终被存储在系统表中的执行计划。而批处理的Transaction-SQL语句在每次运行时都要进行编译和优化，速度相对要慢一些.
    
    4. 存储过程能过减少网络流量。针对同一个数据库对象的操作（如查询、修改），如果这一操作所涉及的Transaction-SQL语句被组织程存储过程，那么当在客户计算机上调用该存储过程时，网络中传送的只是该调用语句，从而大大增加了网络流量并降低了网络负载.
    
    5. 存储过程可被作为一种安全机制来充分利用。系统管理员通过执行某一存储过程的权限进行限制，能够实现对相应的数据的访问权限的限制，避免了非授权用户对数据的访问，保证了数据的安全.

1、创建存储过程

```
### 格式：
CREATE PROCEDURE 过程名 ([过程参数[,...]])[特性 ...] 过程体

### 示例：
delimiter //
create procedure p1()
begin
select * from user_info;
end //
delimiter ;
```
2、执行存储过程

```
call p1()
```

##### 对于存储过程，可以接收参数，其参数有三类：
    - in    表示该参数的值必须在调用存储过程时指定，在存储过程中修改该参数的值不能被返回，为默认值
    
    - out   该值可在存储过程内部被改变，并可返回
    
    - inout 调用时指定，并且可被改变和返回

in参数示例：

```
delimiter //
create procedure demo_in(in p_in int)   # 定义带参数的存储过程
begin
    select p_in;
    set p_in = 2;
    select p_in;
end //
delimiter ;

### 执行方式：
set @p_in = 1;
call demo_in(@p_in);
```
out参数示例：

```
delimiter //
create procedure demo_out(out p_out int)   # 定义带参数的存储过程
begin
    select p_out;
    set p_out = 2;
    select p_out;
end //
delimiter ;

### 执行方式：
set @p_out = 1;
call demo_out(@p_out);
select @p_out;
```
inout参数示例：

```
delimiter //
create procedure demo_inout(inout p_inout int)   # 定义带参数的存储过程
begin
    select p_inout;
    set p_inout = 2;
    select p_inout;
end //
delimiter ;

### 执行方式：
set @p_inout = 1;
call demo_inout(@p_inout);
select @p_inout;

```
3、变量

```
### 变量定义
declare variable_name [,variable_name...] datatype [declare value];其中，datatype为MySQL的数据类型，如:int, float, date, varchar(length)

### 示例
declare l_int int unsigned default 10000;

### 变量赋值
set 变量名 = 表达式 [,variable_name = expression ...]

### 示例
declare abc int;    # 定义变量
set abc = 123;      # 赋值变量
```

##### 综合示例：

```
delimiter //
create procedure synthesis_demo(
    in i1 int,
    in i2 int,
    inout i3 int,
    out r1 int
)
begin
    declare temp1 int;
    declare temp2 int default 0;
    
    set temp1 = 1;

    set r1 = i1 + i2 + temp1 + temp2;
    
    set i3 = i3 + 100;

end //
delimiter ;

### 执行存储过程
set @t1 =4;
set @t2 = 0;
call synthesis_demo (1, 2 ,@t1, @t2);
select @t1,@t2;
```
4、查询存储过程

```
select name from mysql.proc where db=’数据库名’;

select routine_name from information_schema.routines where routine_schema='数据库名';

show procedure status where db='数据库名';

PS：查看某个存储过程的详细

show create procedure 数据库.存储过程名;
```


5、删除存储过程

```
drop procedure 存储过程名;
```

6、执行存储过程

```
-- 无参数
call 存储过程名()

-- 有参数，全in
call 存储过程名(1,2)

-- 有参数，有in，out，inout
set @t1=0;
set @t2=3;
call 存储过程名(1,2,@t1,@t2)
```

#### 存储过程中的控制语句
##### 变量的作用域
    内部的变量在其作用域范围内享有更高的优先权，当执行到end。变量时，内部变量消失，此时已经在其作用域外，变量不再可见了，应为在存储过程外再也不能找到这个申明的变量，但是你可以通过out参数或者将其值指派给会话变量来保存其值.

```
delimiter //
create procedure proc3()
begin
    declare x1 varchar(5) default 'outer';
begin
    declare x1 varchar(5) default 'inner';
    select x1;
end;
    select x1;
end //
delimiter ;

### 执行
call proc3();
```

##### 条件语句

```
delimiter //
create procedure proc_if ()
begin
    declare i int default 0;
    if i = 1 then
        select 1;
    elseif i = 2 then
        select 2;
    else
        select 7;
    end if;

end //
delimiter ;

### 执行
call proc_if();
```

##### 循环语句

###### while循环
    while ···· end while
```

delimiter //
create procedure proc_while ()
begin
    declare num int ;
    set num = 0 ;
    while num < 10 do
        select
            num ;
        set num = num + 1 ;
    end while ;

end //
delimiter ;

### 执行
call proc_while();

```
###### repeat循环
    repeat···· end repeat
    它在执行操作后检查结果，而while则是执行前进行检查

```
delimiter //
create procedure proc_repeat ()
begin
    declare i int ;
    set i = 0 ;
    repeat
        select i;
        set i = i + 1;
        until i >= 5
    end repeat;
end //
delimiter ;

### 执行
call proc_repeat();
```
###### loop ·····end loop
    loop循环不需要初始条件，这点和while 循环相似，同时和repeat循环一样不需要结束条件, leave语句的意义是离开循环
    
```
delimiter //
create procedure proc_loop ()
begin
    declare i int default 0;
    loop_label: loop
        
        set i=i+1;
        if i<8 then
            iterate loop_label;
        end if;
        if i>=10 then
            leave loop_label;
        end if;
        select i;
    end loop loop_label;
end //
delimiter ;

### 执行
call proc_loop()
```
###### iterate迭代
    通过引用复合语句的标号,来从新开始复合语句

```
delimiter //  
create procedure proc_iterate ()
begin 
	declare v int;  
	set v=0;  
	loop_lable:loop  
	if v=3 then   
	set v=v+1;  
	iterate loop_lable;  
	end if;  
	select v;
	set v=v+1;  
	if v>=5 then 
	leave loop_lable;  
	end if;  
	end loop;  
end //
delimiter ;

### 执行
call proc_iterate()
```

###### case语句

```
delimiter //  
create procedure proc_case (in parameter int)  
begin 
	declare var int;  
	set var=parameter+1;  
	case var  
	when 0 then   
	select  var; 
	when 1 then   
	select  var;   
	else   
	select  var;  
	end case;  
end // 
delimiter ;

### 执行
set @proc = 1;  # 当你修改@proc的值的时候每次调用proc_case输出的都会和当前@proc自身加1
call proc_case(@proc)
```

### 四、mysql存储过程的基本函数
#### 字符串类

```
CHARSET(str) //返回字串字符集
CONCAT (string2 [,... ]) //连接字串
INSTR (string ,substring ) //返回substring首次在string中出现的位置,不存在返回0
LCASE (string2 ) //转换成小写
LEFT (string2 ,length ) //从string2中的左边起取length个字符
LENGTH (string ) //string长度
LOAD_FILE (file_name ) //从文件读取内容
LOCATE (substring , string [,start_position ] ) 同INSTR,但可指定开始位置
LPAD (string2 ,length ,pad ) //重复用pad加在string开头,直到字串长度为length
LTRIM (string2 ) //去除前端空格
REPEAT (string2 ,count ) //重复count次
REPLACE (str ,search_str ,replace_str ) //在str中用replace_str替换search_str
RPAD (string2 ,length ,pad) //在str后用pad补充,直到长度为length
RTRIM (string2 ) //去除后端空格
STRCMP (string1 ,string2 ) //逐字符比较两字串大小,
SUBSTRING (str , position [,length ]) //从str的position开始,取length个字符,
```
#### 数字类

```
ABS (number2 ) //绝对值
BIN (decimal_number ) //十进制转二进制
CEILING (number2 ) //向上取整
CONV(number2,from_base,to_base) //进制转换
FLOOR (number2 ) //向下取整
FORMAT (number,decimal_places ) //保留小数位数
HEX (DecimalNumber ) //转十六进制
注：HEX()中可传入字符串，则返回其ASC-11码，如HEX('DEF')返回4142143
也可以传入十进制整数，返回其十六进制编码，如HEX(25)返回19
LEAST (number , number2 [,..]) //求最小值
MOD (numerator ,denominator ) //求余
POWER (number ,power ) //求指数
RAND([seed]) //随机数
ROUND (number [,decimals ]) //四舍五入,decimals为小数位数]
```

#### 时间日期类

```
ADDTIME (date2 ,time_interval ) //将time_interval加到date2
CONVERT_TZ (datetime2 ,fromTZ ,toTZ ) //转换时区
CURRENT_DATE ( ) //当前日期
CURRENT_TIME ( ) //当前时间
CURRENT_TIMESTAMP ( ) //当前时间戳
DATE (datetime ) //返回datetime的日期部分
DATE_ADD (date2 , INTERVAL d_value d_type ) //在date2中加上日期或时间
DATE_FORMAT (datetime ,FormatCodes ) //使用formatcodes格式显示datetime
DATE_SUB (date2 , INTERVAL d_value d_type ) //在date2上减去一个时间
DATEDIFF (date1 ,date2 ) //两个日期差
DAY (date ) //返回日期的天
DAYNAME (date ) //英文星期
DAYOFWEEK (date ) //星期(1-7) ,1为星期天
DAYOFYEAR (date ) //一年中的第几天
EXTRACT (interval_name FROM date ) //从date中提取日期的指定部分
MAKEDATE (year ,day ) //给出年及年中的第几天,生成日期串
MAKETIME (hour ,minute ,second ) //生成时间串
MONTHNAME (date ) //英文月份名
NOW ( ) //当前时间
SEC_TO_TIME (seconds ) //秒数转成时间
STR_TO_DATE (string ,format ) //字串转成时间,以format格式显示
TIMEDIFF (datetime1 ,datetime2 ) //两个时间差
TIME_TO_SEC (time ) //时间转秒数]
WEEK (date_time [,start_of_week ]) //第几周
YEAR (datetime ) //年份
DAYOFMONTH(datetime) //月的第几天
HOUR(datetime) //小时
LAST_DAY(date) //date的月的最后日期
MICROSECOND(datetime) //微秒
MONTH(datetime) //月
MINUTE(datetime) //分返回符号,正负或0
SQRT(number2) //开平方
```

### 五、索引
    索引，是数据库中专门用于帮助用户快速查询数据的一种数据结构。类似于字典中的目录，查找字典内容时可以根据目录查找到数据的存放位置，然后直接获取即可.
##### 索引的工作方式是：
    在索引创建之后，它记录与被索引字段相关联的位置值。当表里添加新数据时，索引里也会添加新项，当数据库执行查询，而且where条件里指定的字段已经设置了索引时，数据库会首先在索引里搜索where子句指定的值，如果在索引里找到了这个值，索引就可以返回被搜索数据在表里的实际位置.
##### mysql中常见的索引有:
- 普通索引
- 唯一索引
- 主键索引
- 组合索引

1、普通索引(普通索引仅有一个功能：加速查询)

```
### 格式
create index index_name on table_name(column_name)

### 创建普通索引+表 示例：
create table in1(
    nid int not null auto_increment primary key,
    name varchar(32) not null,
    email varchar(64) not null,
    extra text,
    index ix_name (name)
)

### 查看索引
show index from 表名;

### 删除索引
drop 索引名 on 表名;
```
2、唯一索引（唯一索引有两个功能：加速查询 和 唯一约束（可含null））

```
### 格式
create unique index 索引名 on 表名(列名)

### 创建唯一索引+表 示例：
create table in1(
    nid int not null auto_increment primary key,
    name varchar(32) not null,
    email varchar(64) not null,
    extra text,
    unique ix_name (name)
)

### 查看唯一索引
show index from 表名;

### 删除唯一索引
drop unique index 索引名 on 表名
```

3、主键索引(主键有两个功能：加速查询 和 唯一约束（不可含null）)

```
### 格式
alter table 表名 add primary key(列名);

### 创建表 + 创建主键 示例：
create table in1(
    nid int not null auto_increment primary key,
    name varchar(32) not null,
    email varchar(64) not null,
    extra text,
    index ix_name (name)
)

OR

create table in1(
    nid int not null auto_increment,
    name varchar(32) not null,
    email varchar(64) not null,
    extra text,
    primary key(ni1),
    index ix_name (name)
)

### 查看主键索引
show index from 表名;

### 删除主键索引
alter table 表名 drop primary key;
alter table 表名  modify  列名 int, drop primary key;

```
4、组合索引(组合索引是将n个列组合成一个索引)

```
### 创建表
create table in3(
    nid int not null auto_increment primary key,
    name varchar(32) not null,
    email varchar(64) not null,
    extra text
)

### 创建组合索引
create index ix_name_email on in3(name,email);

### 查看组合索引
show index from 表名;

### 删除组合索引
对于组合索引，如从表中删除了某列，则索引会受到影响。则该列也会从索引中删除。如果删除组成索引的所有列，则整个索引将被删除
```























