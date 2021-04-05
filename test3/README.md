## 实验三 创建分区表

#### 姓名：喻丹  学号：201810414203   班级：18级软工2班

#### 用户名：yudan

### 实验目的

掌握分区表的创建方法，掌握各种分区方式的使用场景。

### 实验内容

<b>·</b>本实验使用3个表空间：USERS,USERS02,USERS03。在表空间中创建两张表：订单表(orders)与订单详表(order_details)。

<b>·</b>使用你自己的账号创建本实验的表，表创建在上述3个分区，自定义分区策略。

<b>·</b>你需要使用system用户给你自己的账号分配上述分区的使用权限。你需要使用system用户给你的用户分配可以查询执行计划的权限。

<b>·</b>表创建成功后，插入数据，数据能并平均分布到各个分区。每个表的数据都应该大于1万行，对表进行联合查询。

<b>·</b>写出插入数据的语句和查询数据的语句，并分析语句的执行计划。

<b>·</b>进行分区与不分区的对比实验。

### 实验步骤:

#### 步骤一：

创建自己的账号yudan，然后以system身份登录,用户授权三个表空间所分配的空间没有限制:

SQL>ALTER USER yudan QUOTA UNLIMITED ON USERS;

SQL>ALTER USER yudan QUOTA UNLIMITED ON USERS02;

SQL>ALTER USER yudan QUOTA UNLIMITED ON USERS03;

SQL>exit

![](picture1.png)

#### 步骤二：

以自己的账号yudan身份登录，并运行脚本文件test3.sql

[student@deep02 ~]$ sqlplus yudan/123@localhost/pdborcl

SQL> @test3.sql

脚本文件建表分区内容：

       CREATE TABLE ORDERS
       (
       ORDER_ID NUMBER(10, 0) NOT NULL
       , CUSTOMER_NAME VARCHAR2(40 BYTE) NOT NULL
       , CUSTOMER_TEL VARCHAR2(40 BYTE) NOT NULL
       , ORDER_DATE DATE NOT NULL
       , EMPLOYEE_ID NUMBER(6, 0) NOT NULL
       , DISCOUNT NUMBER(8, 2) DEFAULT 0
       , TRADE_RECEIVABLE NUMBER(8, 2) DEFAULT 0
       , CONSTRAINT ORDERS_PK PRIMARY KEY
       (
       ORDER_ID
       )
       USING INDEX
       (
       CREATE UNIQUE INDEX ORDERS_PK ON ORDERS (ORDER_ID ASC)
       LOGGING
       TABLESPACE USERS
       PCTFREE 10
       INITRANS 2
       STORAGE
       (
              BUFFER_POOL DEFAULT
       )
       NOPARALLEL
       )
       ENABLE
       )
       TABLESPACE USERS
       PCTFREE 10
       INITRANS 1
       STORAGE
       (
       BUFFER_POOL DEFAULT
       )
       NOCOMPRESS
       NOPARALLEL
       PARTITION BY RANGE (ORDER_DATE)
       (
       PARTITION PARTITION_2015 VALUES LESS THAN (TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
       NOLOGGING
       TABLESPACE USERS
       PCTFREE 10
       INITRANS 1
       STORAGE
       (
       INITIAL 8388608
       NEXT 1048576
       MINEXTENTS 1
       MAXEXTENTS UNLIMITED
       BUFFER_POOL DEFAULT
       )
       NOCOMPRESS NO INMEMORY
       , PARTITION PARTITION_2016 VALUES LESS THAN (TO_DATE(' 2017-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
       NOLOGGING
       TABLESPACE USERS
       PCTFREE 10
       INITRANS 1
       STORAGE
       (
       BUFFER_POOL DEFAULT
       )
       NOCOMPRESS NO INMEMORY
       , PARTITION PARTITION_2017 VALUES LESS THAN (TO_DATE(' 2018-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
       NOLOGGING
       TABLESPACE USERS
       PCTFREE 10
       INITRANS 1
       STORAGE
       (
       BUFFER_POOL DEFAULT
       )
       NOCOMPRESS NO INMEMORY
       , PARTITION PARTITION_2018 VALUES LESS THAN (TO_DATE(' 2019-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
       NOLOGGING
       TABLESPACE USERS02
       PCTFREE 10
       INITRANS 1
       STORAGE
       (
       BUFFER_POOL DEFAULT
       )
       NOCOMPRESS NO INMEMORY
       , PARTITION PARTITION_2019 VALUES LESS THAN (TO_DATE(' 2020-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
       NOLOGGING
       TABLESPACE USERS02
       PCTFREE 10
       INITRANS 1
       STORAGE
       (
       BUFFER_POOL DEFAULT
       )
       NOCOMPRESS NO INMEMORY
       , PARTITION PARTITION_2020 VALUES LESS THAN (TO_DATE(' 2021-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
       NOLOGGING
       TABLESPACE USERS02
       PCTFREE 10
       INITRANS 1
       STORAGE
       (
       BUFFER_POOL DEFAULT
       )
       NOCOMPRESS NO INMEMORY
       , PARTITION PARTITION_2021 VALUES LESS THAN (TO_DATE(' 2022-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
       NOLOGGING
       TABLESPACE USERS03
       PCTFREE 10
       INITRANS 1
       STORAGE
       (
       BUFFER_POOL DEFAULT
       )
       NOCOMPRESS NO INMEMORY
       );

       CREATE TABLE order_details
       (
       id NUMBER(10, 0) NOT NULL
       , order_id NUMBER(10, 0) NOT NULL
       , product_name VARCHAR2(40 BYTE) NOT NULL
       , product_num NUMBER(8, 2) NOT NULL
       , product_price NUMBER(8, 2) NOT NULL
       , CONSTRAINT order_details_fk1 FOREIGN KEY  (order_id)
       REFERENCES orders  (  order_id   )
       ENABLE
       )
       TABLESPACE USERS
       PCTFREE 10 INITRANS 1
       STORAGE (BUFFER_POOL DEFAULT )
       NOCOMPRESS NOPARALLEL
       PARTITION BY REFERENCE (order_details_fk1);

![](picture2.png)

#### 步骤三：

运行脚本文件之后，我们通过sqldeveloper可以看到两个表已经创建完成，数据也已经插入好了。

插入数据的语句：

       declare
              dt date;
              m number(8,2);
              V_EMPLOYEE_ID NUMBER(6);
              v_order_id number(10);
              v_name varchar2(100);
              v_tel varchar2(100);
              v number(10,2);
              v_order_detail_id number;
       begin
       /*
       system login:
       ALTER USER "TEACHER" QUOTA UNLIMITED ON USERS;
       ALTER USER "TEACHER" QUOTA UNLIMITED ON USERS02;
       ALTER USER "TEACHER" QUOTA UNLIMITED ON USERS03;
       */
              v_order_detail_id:=1;
              delete from order_details;
              delete from orders;
              for i in 1..10000
              loop
              if i mod 6 =0 then
              dt:=to_date('2015-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2015
              elsif i mod 6 =1 then
              dt:=to_date('2016-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2016
              elsif i mod 6 =2 then
              dt:=to_date('2017-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2017
              elsif i mod 6 =3 then
              dt:=to_date('2018-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2018
              elsif i mod 6 =4 then
              dt:=to_date('2019-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2019
              else
              dt:=to_date('2020-3-2','yyyy-mm-dd')+(i mod 60); --PARTITION_2020
              end if;
              V_EMPLOYEE_ID:=CASE I MOD 6 WHEN 0 THEN 11 WHEN 1 THEN 111 WHEN 2 THEN 112
                                          WHEN 3 THEN 12 WHEN 4 THEN 121 ELSE 122 END;
              --插入订单
              v_order_id:=i;
              v_name := 'aa'|| 'aa';
              v_name := 'zhang' || i;
              v_tel := '139888883' || i;
              insert /*+append*/ into ORDERS (ORDER_ID,CUSTOMER_NAME,CUSTOMER_TEL,ORDER_DATE,EMPLOYEE_ID,DISCOUNT)
              values (v_order_id,v_name,v_tel,dt,V_EMPLOYEE_ID,dbms_random.value(100,0));
              --插入订单y一个订单包括3个产品
              v:=dbms_random.value(10000,4000);
              v_name:='computer'|| (i mod 3 + 1);
              insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
              values (v_order_detail_id,v_order_id,v_name,2,v);
              v:=dbms_random.value(1000,50);
              v_name:='paper'|| (i mod 3 + 1);
              v_order_detail_id:=v_order_detail_id+1;
              insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
              values (v_order_detail_id,v_order_id,v_name,3,v);
              v:=dbms_random.value(9000,2000);
              v_name:='phone'|| (i mod 3 + 1);

              v_order_detail_id:=v_order_detail_id+1;
              insert /*+append*/ into ORDER_DETAILS(ID,ORDER_ID,PRODUCT_NAME,PRODUCT_NUM,PRODUCT_PRICE)
              values (v_order_detail_id,v_order_id,v_name,1,v);
              --在触发器关闭的情况下，需要手工计算每个订单的应收金额：
              select sum(PRODUCT_NUM*PRODUCT_PRICE) into m from ORDER_DETAILS where ORDER_ID=v_order_id;
              if m is null then
              m:=0;
              end if;
              UPDATE ORDERS SET TRADE_RECEIVABLE = m - discount WHERE ORDER_ID=v_order_id;
              IF I MOD 1000 =0 THEN
              commit; --每次提交会加快插入数据的速度
              END IF;
              end loop;
       end;

![](picture3.1.png)

![](picture3.2.png)

#### 步骤四：

写查询数据的语句，并分析语句的执行计划。

       set autotrace on

       select * from yudan.orders where order_date
       between to_date('2017-1-1','yyyy-mm-dd') and to_date('2018-6-1','yyyy-mm-dd');

![](picture4.1.png)

![](picture4.2.png)

![](picture4.3.png)

       select a.ORDER_ID,a.CUSTOMER_NAME,
       b.product_name,b.product_num,b.product_price
       from yudan.orders a,yudan.order_details b where
       a.ORDER_ID=b.order_id and
       a.order_date between to_date('2017-1-1','yyyy-mm-dd') and to_date('2018-6-1','yyyy-mm-dd');

![](picture4.4.png)

![](picture4.5.png)

#### 步骤五：

查看数据库的使用情况，查看表空间的数据库文件，以及每个文件的磁盘占用情况。

       SQL>SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';

       SQL>SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
       free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
       Round(( total - free )/ total,4)* 100 "使用率%"
       from (SELECT tablespace_name,Sum(bytes)free
              FROM   dba_free_space group  BY tablespace_name)a,
              (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
              group  BY tablespace_name)b
       where  a.tablespace_name = b.tablespace_name;

![](picture5.png)

#### 实验分析总结

通过操作步骤可以知道，orders表按照范围分区，通过订单日期进行分区。而orders与order_details表主外键关联，所以在order_details表使用引用分区进行存储。
通过对执行计划分析，我们可以知道有分区查找数据的优势更大一些，速度还是要快一些的。所以如果数据量小，有分区与无分区差别不是很大，甚至无分区可能更快。如果数据量大，有分区的表优势明显加大。
查看了表空间的数据库文件，我们可以看到orders表按范围在USERS、USERS02、USERS03空间存储，USERS、USERS02表空间使用率高些，USERS03使用率低些。order_details表则全存储在USERS空间。
