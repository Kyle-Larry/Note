#### HDFS架构概述
- Namenode：存储元数据
- Datanode：存储数据的节点，会对数据块进行校验
- Secondarynamenode：监控namenode 的元数据，每隔一定的时间进行元数据的合并

#### YARN架构概述
- ResourceManager(rm)：
处理客户端请求、启动/监控ApplicationMaster、监控NodeManager、资源分配与调度
- NodeManager(nm)：
单个节点上的资源管理、处理来自ResourceManager、ApplicationMaster的命令
- ApplicationMaster：
数据切分、为应用程序申请资源，并分配给内部任务、任务监控与容错
- Container：
对任务运行环境的抽象，封装了CPU、内存等多维资源以及环境变量、启动命令等任务运行相关的信息

#### MapReduce架构概述
- MapReduce将计算过程分为两个阶段：Map和Reduce
- Map阶段并行处理输入数据
- Reduce阶段对Map结果进行汇总

#### 配置文件
- core-site.xml、hdfs-site.xml、yarn-site.xml、mapred-site.xml、
- hadoop-env.sh、yarn-env.sh、mapred-env.sh分别添加jdk环境变量
- slaves配置主机名

##### 1、为什么要格式化？（hdfs namenode -format）
NameNode主要被用来管理整个分布式文件系统的命名空间(实际上就是目录和文件)的元数据信息，同时为了保证数据的可靠性，还加入了操作日志，所以，NameNode会持久化这些数据(保存到本地的文件系统中)。对于第一次使用HDFS，在启动NameNode时，需要先执行-format命令，然后才能正常启动NameNode节点的服务。
##### 2、格式化做了哪些事情？
在NameNode节点上，有两个最重要的路径，分别被用来存储元数据信息和操作日志，而这两个路径来自于配置文件，它们对应的属性分别是dfs.name.dir和dfs.name.edits.dir，同时，它们默认的路径均是/tmp/hadoop/dfs/name。格式化时，NameNode会清空两个目录下的所有文件，之后，会在目录dfs.name.dir下创建文件
hadoop.tmp.dir 这个配置，会让dfs.name.dir和dfs.name.edits.dir会让两个目录的文件生成在一个目录里