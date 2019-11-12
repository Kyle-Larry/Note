### 一、Hive基本概念
1. 什么是Hive
    ~~~
    Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能。
    本质是将HQL/SQL转换为MapReduce程序。
        1）Hive处理的数据存储在HDFS
        2）Hive分析数据底层的实现是MapReduce
        3)执行程序运行在Yarn上

1. Hive的优缺点
     ~~~
    优点：
        1）操作接口采用类SQL语法，避免写MapReduce
        2）支持用户自定义函数
        3）优势在于处理大数据，进行离线数据分析，因为Hive的执行延迟比较高
    缺点：
        1）HQL表达能力有限，无法表达迭代式算法，不擅长数据挖掘方面
        2）效率比较低，自动生成的MapReduce Job通常不够智能化，调优比较困难，粗度较大

1. Hive架构原理

   ![hive](picture/hive/Hive_architecture.jpg)
     ~~~
    组成及作用：
        用户接口：ClientCLI（hive shell）、JDBC/ODBC(java访问hive)、WEBUI（浏览器访问hive）
        
        元数据(Metastore)：包括表名、表所属的数据库（默认是default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等；默认存储在自带的derby数据库中，推荐使用MySQL存储Metastore
        Hadoop：使用HDFS进行存储，使用MapReduce进行计算
        
        驱动器（Driver）：
        （1）解析器（SQL Parser）：将SQL字符串转换成抽象语法树AST，这一步一般都用第三方工具库完成，比如antlr；对AST进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误。
        （2）编译器（Physical Plan）：将AST编译生成逻辑执行计划。
        （3）优化器（Query Optimizer）：对逻辑执行计划进行优化。
        （4）执行器（Execution）：把逻辑执行计划转换成可以运行的物理计划。对于Hive来说，就是MR/Spark。

    工作原理：
        用户创建数据库、表信息，存储在hive的元数据库中；
        向表中加载数据，元数据记录hdfs文件路径与表之间的映射关系；
        执行查询语句，首先经过解析器、编译器、优化器、执行器，将指令翻译成MapReduce，提交到Yarn上执行，最后将执行返回的结果输出到用户交互接口。

1. Hive和传统关系型数据库(RMDB)比较

   ![hive](picture/hive/hive-compare-common-db.jpg)
     ~~~
    数据库的事务、索引以及更新都是传统数据库的重要特性。但是Hive到目前也不支持更新（这里说的是对行级别的数据进行更新），不支持事务；
    虽然Hive支持建立索引，但是它还不能提升数据的查询速度：

1. 参数配置方式
     ~~~
    1)配置文件
        优先级：Hadoop配置 < 默认配置hive-default.xml < 用户自定义配置hive-site.xml ;
        配置文件的设定对本机启动的所有Hive进程都有效。
    2）命令行参数
        bin/hive -hiveconf mapred.reduce.tasks=10;
        仅对本次hive启动有效。
    3）参数声明
        hive (default)> set mapred.reduce.tasks=10;
        仅对本次hive启动有效
        
     注：
        查看配置信息：
            hive>set;
            hive (default)> set mapred.reduce.tasks;
        
        上述三种设定方式的优先级依次递增,即配置文件<命令行参数<参数声明。
        注意某些系统级的参数，例如log4j相关的设定，必须用前两种方式设定，因为那些参数的读取在会话建立以前已经完成了。

1. Hive数据类型
     ~~~
    基本类型：TINYINT、SMALINT、INT、BIGINT、BOOLEAN、FLOAT、DOUBLE、STRING、TIMESTAMP、BINARY
    集合类型：STRUCT、MAP、ARRAY
    
    类型转化：
        1）隐式类型转换：
            a.任何整数类型都可以隐式地转换为一个范围更广的类型
            b.所有整数类型、FLOAT和STRING类型都可以隐式地转换成DOUBLE
            c.TINYINT、SMALLINT、INT都可以转换为FLOAT
            d.BOOLEAN类型不可以转换为任何其它的类型
        2)CAST显示转换：
            // a为binary（字节数组），a代表数字时
            // 如果强制类型转换失败，表达式返回空值 NULL
            SELECT (cast(cast(a as string) as double)) from src;
      
1. 实操：导入文件数据到表中
    ~~~
    样例json:
        {
            "name": "songsong",
            "friends": ["bingbing" , "lili"] ,       //列表Array, 
            "children": {                      //键值Map,
                "xiao song": 18 ,
                "xiaoxiao song": 17  
          }
            "address": {                      //结构Struct,
                "street": "hai dian qu" ,
                "city": "beijing" 
            }
        }
    
    样例文本：
        songsong,bingbing_lili,xiao song:18_xiaoxiao song:17,hai dian qu_beijing
        yangyang,caicai_susu,xiao yang:18_xiaoxiao yang:19,chao yang_beijing 
    
    1）建表HQL
        create table test(
            name string,
            friends array<string>,
            children map<string, int>,
            address struct<street:string, city:string>
        )
        row format delimited fields terminated by ','
        collection items terminated by '_'
        map keys terminated by ':'
        lines terminated by '\n';
        
    2）导入数据
        hive (default)> load data local inpath '/test.txt' into table test;
        
    3）访问集合中的数据，以下分别是ARRAY，MAP，STRUCT的访问方式
        hive (default)> select friends[1]as`朋友`,children['xiao song']as`年龄`,address.city from test where name="songsong";
        
### 二、DDL数据定义
   数据库模式定义语言DDL(Data Definition Language)，是用于描述数据库中要存储的现实世界实体的语言。
   1. 数据库操作
        ~~~
        1)创建数据库
            hive (default)> create database if not exists db_hive location '/db_hive.db';
            hive> desc database extended db_hive;
            
        2）修改数据库
            hive (default)> alter database db_hive set dbproperties('createtime'='20190506');
            // 数据库的其他元数据信息都是不可更改的，包括数据库名和数据库所在的目录位置。
            
        3）查看数据库
            hive> show databases like 'db_hive*';
            
        4)删除数据库
            hive> drop database if exists db_hive cascade; 
   
   1. 表操作
        ~~~
        1）表的类型
           Table Type: MANAGED_TABLE、EXTERNAL_TABLE
           管理表（内部表）：
               默认创建的表就是管理表，当我们删除一个管理表时，Hive也会删除这个表中数据。管理表不适合和其他工具共享数据。
           外部表：
               删除该表并不会删除掉这份数据，不过描述表的元数据信息会被删除掉。  
        
        2）创建表
            a.普通创建表
                create (external) table if not exists student2(id int, name string)
                row format delimited fields terminated by '\t'
                stored as textfile
                location '/user/hive/warehouse/student2';
                
            b.根据查询结果创建表（查询的结果会添加到新创建的表中）
                create (external) table if not exists student3 as select id, name from student;
            
            c.根据已经存在的表结构创建表
                create (external) table if not exists student4 like student;       
   
        3）修改表
            a.重命名
                alter table student2 rename to student1;
            b.添加列
                 alter table student1 add columns(desc string);
                 // ADD是代表新增一字段，字段位置在所有列后面(partition列前)
            c.更新列
                 alter table student1 change column desc addr string;
            d.替换列
                alter table student1 replace columns(id int, name string);
                // REPLACE则是表示替换表中所有字段
        
        4）删除表
            drop table if exists student1;
   
   1. 分区表
        ~~~
        分区表实际上就是对应一个HDFS文件系统上的独立的文件夹，该文件夹下是该分区所有的数据文件。
        在查询时通过WHERE子句中的表达式选择查询所需要的指定的分区，这样的查询效率会提高很多。
        
        1）创建分区表
            create table dept_partition(
                deptno int, dname string, loc string
            )
            // 二级分区表
            partitioned by (month string, day string)
            row format delimited fields terminated by '\t';
        
        2）查询分区表中数据
            // 单分区查询
            select * from dept_partition where month='201911' and  day='11';    
            
            // 多分区联合查询
            select * from dept_partition where month='201908'
            union
            select * from dept_partition where month='201907'
            union
            select * from dept_partition where month='201909';
        
        3）增加分区
            alter table dept_partition add partition(month='201905') partition(month='201904');
            // 增加多个分区之间用空格" "隔开，删除多个分区用","隔开
       
        4）删除分区
            alter table dept_partition drop partition (month='201905'), partition (month='201904');
        
        5）查看分区
            show partitions dept_partition;
  
        6）把数据直接上传到分区目录上，让分区表和数据产生关联的两种方式
            a.上传数据后修复
                dfs -mkdir -p /hive/dept_partition/month=201911/day=11;
                dfs -put /dept.txt /hive/dept_partition/month=201911/day=11;
                // 修复命令
                msck repair table dept_partition;
                
            b.上传数据后添加分区
                dfs -mkdir -p /hive/dept_partition/month=201911/day=12;
                dfs -put /dept.txt /hive/dept_partition/month=201911/day=12;
                // 执行添加分区
                alter table dept_partition add partition(month='201911', day='12');
            
            c.上传数据后load数据到分区
                dfs -mkdir -p /hive/dept_partition/month=201911/day=13;
                load data local inpath '/dept.txt' into table dept_partition partition(month='201911',day='13');

### Hive常用命令

1. “-e”不进入hive的交互窗口执行sql语句
   ~~~  
   bin/hive -e "select id from student;"
   
1. “-f”执行脚本中sql语句
     ~~~
     bin/hive -f ./data/student.sql > ./data/student.txt
     
     
