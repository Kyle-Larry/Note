### 一、vi编辑命令
command | operation
:-: | :-: 
yy  | 复制
yNy | 复制N行
p  |  粘贴
u  |  撤销
dd |  删除一行
dNd | 删除N行
shift + ^ | 移动到行头
shift + $  | 移动到行尾
N + shift + g  |跳到第N行
o | 进入下一行的编辑模式

### 二、文件目录
command | operation
:-: | :-: 
touch	| 创建空文件
echo	|	追加文件
mkdir -p | 递归创建
cp -r |	递归复制
ln -s	[原文件][目标文件] |	软连接
history |	历史服务器

### 三、搜索查找
command | operation
:-: | :-: 
find /opt -name *.jar | 在/opt目录下查找jar文件
find /opt -user root | 在/opt目录下查找root用户的文件
find /opt -size +1024 | 在/opt目录下查找大于1M的文件

### 四、压缩和解压缩
command | operation
:-: | :-: 
tar -zcvf archive.tar.gz dir1 | 创建一个gzip格式的压缩包
tar -zxvf archive.tar.gz | 解压一个gzip格式的压缩包
zip a.zip a | 把a压缩成zip格式的文件
unzip + *.zip | 解压文件

### 五、磁盘分区
command | operation
:-: | :-: 
fdisk -l | 查看分区
df | 查看硬盘
mount | 挂载
unmount | 卸载 

### 六、进程线程
command | operation
:-: | :-: 
ps -aux | 查看系统中的进程
top |  查看系统的健康状态

### 七、RPM包
command | operation
:-: | :-: 
rpm -qa \| grep mysql	| 查询是否具有mysql的RPM包
rpm -e --nodeps [包名] | 强制卸载此包
rpm -ivh --nodeps [包名] | 不检测依赖进度

### 八、定时任务Crontab
command | operation
:-: | :-: 
crontab -e | 编辑定时任务
crontab -l | 查询定时任务
crontab -r | 删除定时任务

