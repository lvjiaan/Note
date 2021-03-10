## note

### __SQL SERVER__：

```
日期计算
https://www.cnblogs.com/cjgu/p/5515109.html
SELECT cast(cast(20210101 as varchar) as datetime)
SELECT left(CONVERT(varchar(100),dateadd(month,-1,cast(cast(20210101 as varchar) as datetime)), 112)
```



###### 死锁

```sql
--查询哪些死锁
SELECT request_session_id spid, OBJECT_NAME( resource_associated_entity_id )
tableName FROM sys.dm_tran_locks WHERE resource_type = 'OBJECT'

--杀死进程
kill spid
```

###### 命令

```sql
COALESCE  颗窝赖斯

CONVERT(varchar(8),GETDATE(),112)

row_number()over(partition by cno order by degree desc)
rank()		over(partition by cno order by degree desc)

sqlsever触发器
CREATE TRIGGER trigger_name
 ON dbo.T_Job_Step
after insert
 AS 
INSERT INTO DBO.T_Job_Rule(RuleId) VALUES(dbo.getruleid());
GO

建索引
create index idx_LT_Patent_Main_patentName
    on LT_Patent_Main (patentName, partitionNum)
go

分区表
ON PARTITION_SCHEME_mysqlDB_year ( 分区字段名 )

```

###### 删日志

```sql
--1.修改数据库模式为简单
ALTER DATABASE ResultsLibrary SET RECOVERY SIMPLE;

--2.查看日志名称
SELECT * FROM sysfiles;

--3.缩小日志为1M：
DBCC SHRINKFILE (MultidimensionalData_log, 1);
--或者
DBCC SHRINKFILE (N'MultidimensionalData_log' , TRUNCATEONLY);

--4.修改数据库模式
ALTER DATABASE ResultsLibrary SET RECOVERY FULL;
```

###### sql调优

```sql
查询表条数
SELECT rows FROM sysindexes　WHERE id =OBJECT_ID('dbo.LT_Patent_Main') AND indid <2

先选出需要的字段再进行表关联的速度会快一些

尽量避免在where子句中使用!=或<>操作符，否则将使引擎放弃使用索引而进行全表扫描。

尽量避免在where子句中对字段进行表达式操作，否则将导致全表扫描。 
select id from t where num/2=100 
应改为： 
select id from t wherenum=100*2 

尽量避免在where子句中对字段进行null值判断，否则将导致全表扫描。 
group by 中 where加了 is not null 运行时间：   10 m 35 s 533 ms
					不加去null处理 ： 2 m 1 s 289 ms
```

###### 拆分表中的列

```sql
SELECT t.code,
       v.value unitname
FROM ResultsLibrary.dbo.a_overrule_list t
    CROSS APPLY STRING_SPLIT(t.usedname, '；')v;
```



### LINUX 

```shell
文件授权用户：   chmod -R  andy  wenjianjia
可执行权限：     chmod +x  wenjian 

service postgresql restart

rpm -ivh rpm包名

yum list installed | grep ruby

rpm -qa|grep -i mysql

rpm -e --nodeps

yum -y remove gcc

rpm -qal |grep mysql


vi /etc/sysconfig/network-scripts/ifcfg-eth0
vi /etc/hosts

hostname
hostnamectl set-hostname ***

关闭防火墙
systemctl disable firewalld
systemctl stop firewalld

关闭SELinux
修改配置文件需要重启机器：
vi /etc/sysconfig/selinux
SELINUX=disabled


scp /root/authorized_keys root@192.168.4.112:/root/authorized_keys

source /etc/profile

mysql -u root -p

ALTER USER 'root'@'localhost' IDENTIFIED BY 'Left@zuo123.';

service mysql restart 

tar -xvf

df -h

localhost.localdomain

/var/www/html/

https://blog.csdn.net/qq_36523839/article/details/78885216
ssh连接的基本原理：服务器生成一把密钥(id_rsa)，一把公钥(id_rsa.pub)。将公钥拷贝到客户端的~/.ssh文件中再cat ./id_rsa.pub >> ./authorized_keys，


#全局修改：
　 所有用户都是同一种统一的语言设置　        
　 修改/etc/sysconfig/i18n文件
　 vi /etc/sysconfig/i18n
　 如果你要修改成中文，就改成LANG="zh_CN.UTF-8"
　 如果你要修改成英文，就改成LANG="en_US.UTF-8"
　 保存之后重启电脑生效


echo 'export LANG=en_US.UTF-8' >> /etc/profile
```

###### hdfs 命令

```shell
hive表存储情况 
hadoop  fs  -du  -s  -h /warehouse/tablespace/managed/hive
```

###### sqoop demo

```shell
#sqlserver导入hive：
sqoop import --connect 'jdbc:sqlserver://192.168.0.215:1433;database=templibrary' \
--username 'test' --password 'wlzx87811024' \
--query 'select  unitId,[企业名称],[曾用名] from dbo.[20201020unit] where $CONDITIONS' \
--hive-import --hive-database 'achievement' --hive-table 'pt_general_export' -m 1 \
--target-dir '/testDir/sqlserver/pt_general_export' --hs2-url jdbc:hive2://192.168.0.217:10000 \
--fields-terminated-by "~"  --null-string '\\N' --null-non-string '\\N' \
--hive-drop-import-delims --hive-overwrite

sqoop import --connect 'jdbc:sqlserver://192.168.0.209:1433;database=Hadoop' \
--username 'test' --password 'wlzx87811024' \
--query 'select * from pt_patent_old where $CONDITIONS' \
--hive-import --hive-database 'achievement' --hive-table 'a_pt_patent_old' -m 1 \
--target-dir '/testDir/sqlserver/a_pt_patent_old' --hs2-url jdbc:hive2://192.168.0.217:10000 \
--fields-terminated-by "\0x01"  --null-string '\\N' --null-non-string '\\N' \
--hive-drop-import-delims --hive-overwrite

sqoop import --connect 'jdbc:sqlserver://192.168.0.210:1433;database=ResultsLibrary' \
--username 'sa' --password 'wlzx@87811024' \
--query 'select * from dbo.temp_applynumin215 where $CONDITIONS' \
--hive-import --hive-database 'achievement' --hive-table 'a_pt_patent_old' -m 1 \
--target-dir '/testDir/sqlserver/a_pt_patent_old' --hs2-url jdbc:hive2://192.168.0.217:10000 \
--fields-terminated-by "\0x01"  --null-string '\\N' --null-non-string '\\N' \
--hive-drop-import-delims --hive-overwrite
```

```shell
#导出
#方法一：
insert overwrite directory '/testDir/status/test' row format delimited fields terminated by '~' STORED AS textfile select * from achievement.tp_lja111;

sqoop export --connect 'jdbc:sqlserver://192.168.0.210:1433;database=ResultsLibrary' \
--username 'sa' --password 'sa@87811024' \
--table 'pt_general' --fields-terminated-by '~'   --export-dir '/testDir/status/test' --input-null-string '\\N' --input-null-non-string '\\N'

#方法二：
beeline --showHeader=false --outputformat=dsv --delimiterForDSV='≮' -e "select * from achievement.company_status_grantpersonname;" > /usr/test/out11.txt

hdfs dfs -put /usr/test/out11.txt   /testDir/status/

sqoop export --connect 'jdbc:sqlserver://192.168.0.209:1433;database=DM_Address' \
--username 'test' --password 'wlzx87811024' \
--table 'a_address' --export-dir '/testDir/status/test' --fields-terminated-by  '\0x01' \
--null-string '\\N' --null-non-string '\\N'


更新导出（update模式）
HQL示例：insert overwrite directory '/user/root/export/test' row format delimited fields terminated by ',' STORED AS textfile select F1,F2,F3 from <sourceHiveTable> where <condition>;

SQOOP脚本：sqoop export --connect jdbc:mysql://localhost:3306/wht --username root --password cloudera \
--table <targetTable> --fields-terminated-by ','  --columns F1,F2,F3 --update-key F4 --update-mode  updateonly --export-dir /user/root/export/test
```

### hive

```sql
--创建表
create table  IF NOT EXISTS achievement.lvtest(
	id		 		    string,
	address				string,
	processcreatetime	 string
) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\0x01' STORED AS TEXTFILE;
格式
1.TEXTFILE 
2.SEQUENCEFILE 
3.RCFILE 

--查看表结构（看分隔符）
show create table ee;

--增加/修改字段
Alter table pt_Patent_Person_Type add columns(personId string);
Alter table lt_patent_pledge change column pledgeid  pledgeid  string

--复制表结构
CREATE TABLE t_old LIKE t_new;

--清空表
truncate table t1;

--删除表
drop table if exists achievement.lvtest;	

--时间格式转换（2009.9.29转成20090929）
select date_format(to_date(from_unixtime(UNIX_TIMESTAMP('2009.9.29','yyyy.MM.dd'))),'yyyyMMdd') ;

--hive 竖表转横
select applynum,concat_ws(',',collect_set(publicnum)) from rt_patent_public_num group by applynum;

if(条件，值1，值2) 条件为真：值1，否则值2

#hive表目录
hadoop fs -ls  /warehouse/tablespace/managed/hive/achievement.db
#hive表占用空间大小查看
hadoop fs -du /warehouse/tablespace/managed/hive/achievement.db/ft_patent_unit_summary_lin|awk '{ SUM += $1 } END { print SUM/(1024*1024*1024)}'

df -h #查看容量使用情况

hive -e "sql str;"

hive -f all.sql


udf
CREATE FUNCTION IntegerFunc AS 'udf.IntegerValueUDF' USING JAR 'hdfs:///dir/*.jar';

list jar;

delete jar /home/hive/hive-andyUDF.jar;
add jar /home/hive/hive-andyUDF.jar;

drop function split_match;
create function split_match as 'cn.andy.hive.split_match';

CREATE FUNCTION split_match as 'cn.andy.hive.split_match' USING JAR 'hdfs:///lvja/hive-andyUDF.jar' ;

```

```sql
insert overwrite table achievement.PT_Patent_Person_Type
select
       distinct '',replace(name,'"',''),replace(type,'"',''),null,null,null,null
from (
        select split(applicant,',') as sss1,split(apporganizationtype,',')as sss2
        --遍历四个表的数据fmzl、fmsq、wgzl、syxx
        from achievement.pt_patent_original_merge_fmzl
     )t
lateral view outer explode(`map`(sss1[0],sss2[0],sss1[1],sss2[1],sss1[2],sss2[2],sss1[3],sss2[3],sss1[4],sss2[4]
    ,sss1[5],sss2[5],sss1[6],sss2[6],sss1[7],sss2[7],sss1[8],sss2[8],sss1[9],sss2[9],sss1[10],sss2[10],sss1[11],sss2[11],sss1[12],sss2[12],
    sss1[13],sss2[13],sss1[14],sss2[14],sss1[15],sss2[15],sss1[16],sss2[16],sss1[17],sss2[17],sss1[18],sss2[18],sss1[19],sss2[19],sss1[20],sss2[20]
    ,sss1[21],sss2[21],sss1[22],sss2[22],sss1[23],sss2[23],sss1[24],sss2[24],sss1[25],sss2[25],sss1[26],sss2[26],sss1[27],sss2[27],sss1[28],sss2[28],
    sss1[29],sss2[29],sss1[30],sss2[30],sss1[31],sss2[31],sss1[32],sss2[32],sss1[33],sss2[33],sss1[34],sss2[34],sss1[35],sss2[35]
    )) info as name,type  where  name is not null and type is not null;
    
    
    
```



###### 调优

```shell
#linux内存清理
sync; echo 2 > /proc/sys/vm/drop_caches

#hive mr调优
SET mapred.reduce.tasks=50;
set hive.exec.parallel=true;
set hive.exec.parallel.thread.number=10;
SET mapreduce.reduce.memory.mb=5000;
SET mapreduce.reduce.shuffle.memory.limit.percent=0.06;
set hive.map.aggr=true;

# hadoop 杀死job
hadoop job -list
hadoop job -kill job_1546932571

#删除临时文件
rm -rf /usr/test/*
hadoop fs -rm -r /testDir/status/*

#回收站清空
hdfs dfs -expunge
```

### JAVA

```java
  ExecutorService 
  ·多线程更新一个表里面的不同行也可能会死锁 
  		解决：表加索引（主键）。
 java 之 AtomicReference
AtomicReference类提供了一个可以原子读写的对象引用变量。 原子意味着尝试更改相同AtomicReference的多个线程（例如，使用比较和交换操作）不会使AtomicReference最终达到不一致的状态。 AtomicReference甚至有一个先进的compareAndSet（）方法，它可以将引用与预期值（引用）进行比较，如果它们相等，则在AtomicReference对象内设置一个新的引用。   
   
      
  \r   回车不换行（进度条）
  
  
  maven打包跳过测试命令
  mvn clean install -Dmaven.test.skip=true
  	
  	
  
```

### Python

```
列表解析

　　根据已有列表，高效创建新列表的方式。

　　列表解析是Python迭代机制的一种应用，它常用于实现创建新的列表，因此用在[]中。

语法：

　　[expression for iter_val in iterable]
　　[expression for iter_val in iterable if cond_expr]
　　
　　
　　
　　
打包exe
pyinstaller -F xxx.py
```

爬虫

```

```

### GIT

```
git clone + 你的仓库地址

进入仓库目录
git init
git add .
git commit -m “你的提交信息”
git push
```



## 备忘

##### 跑专利价值分数：

公开号插入到	[209].DW_ResultLibrary.Patents.T_Publisher   分数 score
		  			   [209].DW_ResultLibrary.Patents.T_PatSnap     价值 PatentValue

 状态置0跑数据

```sql
insert into [209].DW_ResultLibrary.Patents.T_PatSnap(PubNumber,Status)
select distinct publicNum,0 FROM  nbResultsLibrary.dbo.LT_PatentMain t1 where not EXISTS(select 1 from [209].DW_ResultLibrary.Patents.T_PatSnap where pubnumber=t1.publicNum);
```

##### 密码

```
192.168.4.69  admin 1   （XML下载）
sl   zscq123
XMSBTEST   Yq123456
192.168.0.215:1433    test  wlzx87811024
192.168.0.216 root nbsti

190 token http://api.sti.gov.cn/oauth/token?username=jeecg&grant_type=saml_auth&client_id=client_3&client_secret=123456

长三角 cbs  Sai670123124

```

##### 代码

###### 区域常用代码

```
FA33020601	大榭开发区
FA33020602	保税区
FA33021201	高新区
FA33021202	东钱湖旅游度假区
FA33028201	杭州湾新区

望春工业园 专利考核街道代码   330203300

FA33020601,FA33020602,FA33021201,FA33021202,FA33028201

330203	海曙区
330205	江北区
330206	北仑区
330211	镇海区
330212	鄞州区
330213	奉化区
330225	象山县
330226	宁海县
330281	余姚市
330282	慈溪市

P310000,P320000,P330000,P340000

浙江省  P330000
江苏省
安徽省
上海市
```

###### 常用表状态码

```

专利类型
	1 发明专利    4
	2 实用新型    5
	3 外观专利

专利状态：10（有效）
		20（失效）
		21（届满）
		30（在审）
		
地址 最准确字段：province  City citycode district   adcode TownShip TownCode
状态 3：错误   2正确

地址表清洗
9、3下载失败


认领类型（1确认2跨区申请3区内调整）
```

###### other

```
各种加工日期表： dm.dbo.T_Jobdata_log

动态参数名：	dynamicParameter

每月专利统计格式：
对平台数据资源的完整性、准确性、有效性进行全面的检查梳理，通过FTP下载全国专利XML数据531.1GB，现已下载至今年的12月11日（全国专利数量：34289717条，解析更新到2020.11.28；法律状态数量：63376488条，解析更新到2020.11.28；宁波专利数量：937147条，解析更新到2020.11.28），入库商标数据6706029条，入库软件著作数据138741条



届满   发明 20年  其他10年

git地址
http://192.168.4.185/databatch/datasynchronization.git
```

##### 东钱湖接口









































