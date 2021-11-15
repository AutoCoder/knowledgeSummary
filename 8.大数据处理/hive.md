##### Hive join

- Hive Join (内关联)

  仅返回关联上的记录

- Hive LEFT|RIGHT|FULL OUTER JOIN (左关联|右关联|全关联 )

  left 和right 后的 outer 关键字，加不加不影响结果

- Hive LEFT SEMI JOIN (IF/Exists 关联)

  ```SQL
  SELECT a.id, a.name FROM lxw1234_a a LEFT SEMI JOIN lxw1234_b b ON (a.id = b.id);
  
  --执行结果：
  1 zhangsan
  2 lisi
  
  --等价于：
  SELECT a.id, a.name FROM lxw1234_a a WHERE a.id IN (SELECT id FROM lxw1234_b);
  ```

- Full outer join (全连接)

  显示符合条件的数据行，同时显示左右不符合条件的数据行，相应的左右两边显示NULL，即显示左连接、右连接和内连接的并集

- Hive Cross Join  (笛卡尔积关联)

  返回两个表的笛卡尔积结果，不需要指定关联键。 

  ```SQL
  SELECT a.id, a.name, b.age FROM lxw1234_a a CROSS JOIN lxw1234_b b;
       
  --执行结果：
   	1 zhangsan 30
  	1 zhangsan 29
  	1 zhangsan 21
  	2 lisi 30
  	2 lisi 29
  	2 lisi 21
  	3 wangwu 30
  	3 wangwu 29
  	3 wangwu 21
  ```



##### count

```sql
select gender, count(*), count(case when is_vip = 1 then 1 end) from  uaa.tmp_cdum_profile_main_device group by gender;
```

##### COALESCE 

```sql
-- 语法: COALESCE(T v1, T v2, …)  返回参数中的第一个非空值；如果所有值都为NULL，那么返回NULL 
select * from emp;
7369    SMITH   CLERK   7902    1980-12-17      800.0   NULL    20
7499    ALLEN   SALESMAN        7698    1981-2-20       1600.0  300.0   30

select empno,ename,job,mgr,hiredate,sal, COALESCE(comm, 0),deptno from emp;
7369    SMITH   CLERK   7902    1980-12-17      800.0   0.0     20
7499    ALLEN   SALESMAN        7698    1981-2-20       1600.0  300.0   30
```

##### group by

```sql
-- 对雇员中收入大于100的， 用年龄分组，取得每组的人数，最高工资。最后过滤掉人数小于10人的组
select age, max(salary), count(*) from employees where salary > 100 group by age having count(*) > 10;
```

##### limit

```sql
-- limit 语句快速出结果
一般情况下，Limit语句还是需要执行整个查询语句，然后再返回部分结果。
有一个配置属性可以开启，避免这种情况---对数据源进行抽样
hive.limit.optimize.enable=true --- 开启对数据源进行采样的功能
hive.limit.row.max.size --- 设置最小的采样容量
hive.limit.optimize.limit.file --- 设置最大的采样样本数
```

**查看表分区位置**

```sql
describe formatted cdum_predict_pair partition (dt='2018-08-26', type='mobile-mobile');
describe formatted midd_cdum_devices_simple_info partition (dt='2018-08-26');
describe formatted cdum_combined_features partition (dt='2018-08-26', type='mobile-mobile');
```

**添加分区**

```sql
ALTER TABLE cdum_label_pair_set add if not exists partition(dt='2018-09-10') location 'hdfs://hadoop-jy-namenode/data/uaa/commons/cdum/label_pairs/2018-09-10'

ALTER TABLE liveshow_candidates_output add if not exists PARTITION (dt='2019-02-20') LOCATION '/hive/warehouse/uaa.db/liveshow_candidates_output/dt=2019-02-20';

ALTER TABLE ta_static_label_info add if not exists PARTITION (source='mall_id_auth')
LOCATION '/hive/warehouse/uaa.db/ta_static_label_info/source=mall_id_auth';

```

**删除分区**

```
ALTER TABLE uaa.devive_dense_feature_idmapping DROP IF EXISTS PARTITION (dt='2019-10-17');

ALTER TABLE uaa.ta_label_direct_device_label_score DROP IF EXISTS PARTITION (label_name='age', dt='2019-10-24');

ALTER TABLE uaa.ta_label_direct_device_label_score DROP IF EXISTS PARTITION (label_name='gender', dt='2019-10-24');

ALTER TABLE midd_ta_act_relation_month  DROP IF EXISTS PARTITION (dt='2019-08-25',platform=3);

ALTER TABLE uaa.ta_ad_ctr_predict2  DROP IF EXISTS PARTITION (predictname='with_out_age');
```

**变更表分区位置**

```sql
ALTER TABLE cdum_label_pair_set PARTITION (dt='2018-09-10') SET LOCATION 'hdfs://hadoop-jy-namenode/data/uaa/commons/cdum/label_pairs/2018-09-10';


```

**聚合操作**

```sql
--data
Danniel,20,male
Alice,NULL,female
Lily,21,female
Bob,22,male

select max(age),gender from cdum_hive_test group by gender;
22, male
21, female

--data
Danniel,20,male
Alice,NULL,female
Lily,NULL,female
Bob,22,male

select max(age),gender from cdum_hive_test group by gender;
22, male
NULL, female
```

**sort by 和 order by**

> Hive sort by的排序发生在每个reduce里，order by和sort by之间的不同点是前者保证在全局进行排序，而后者仅保证在每个reduce内排序，如果有超过1个reduce，sort by可能有部分结果有序。

**Cluster By 和Distribute By**

> Hive使用distribute by中的列去分发行到每个reduce中，所有同样的distribute by列的行将发送到同一个reduce。
>
> cluster by 是 distribute by + sort by, 不仅分发，还保证每个reduce内按照一定列排序

**修改表结构**

修改表结构后，增加的列通过hql查询为NULL

```sql
-- 解决： 添加CASCADE

alter table uaa.um_rec_i2i_predict_table add columns (sense string) CASCADE;

alter table uaa.midd_ta_filtered_tv add columns (item_id bigint) CASCADE;

alter table uaa.ta_widedeep_predict_result add columns (probs string) CASCADE;

alter table uaa.device_relation_wifi_features add columns (cluster_flag string) CASCADE; ;

-- 修改字段类型

ALTER TABLE uaa.um_ppc_category_tag_avg_ratio CHANGE COLUMN category_id category_id string;
ALTER TABLE uaa.family_structure_from_lianpu add columns (family_members array<string>) CASCADE;

alter table uaa.um_deivce_ppc_tag_merge_binomial CHANGE COLUMN user_id key_id string;
alter table uaa.um_woi_wifi_features_total CHANGE COLUMN scan_stat#avg_freq scan_stat__avg_freq string;
scan_stat#avg_freq      string
conn_stat#avg_freq      string
scan_stat#avg_holidays  string
conn_stat#avg_holidays  string
scan_stat#avg_leisure_hours     string
conn_stat#avg_leisure_hours     string
conn_stat#avg_work_hours        string
scan_stat#avg_work_hours        string
scan_stat#avg_workdays  string
conn_stat#avg_workdays  string
conn_stat#device_cnt    string
scan_stat#device_cnt    string
mac_ssid_cnt#mac_cnt    string
scan_stat#max_freq      string
conn_stat#max_freq      string
conn_stat#max_holidays  string
scan_stat#max_holidays  string
scan_stat#max_leisure_hours     string
conn_stat#max_leisure_hours     string
scan_stat#max_work_hours        string
conn_stat#max_work_hours        string
scan_stat#max_workdays  string
conn_stat#max_workdays  string
conn_stat#repeat_device_cnt     string
scan_stat#repeat_device_cnt     string
mac_ssid_cnt#ssid_cnt   string



ALTER TABLE uaa.um_user_embedding_merge add columns (baidu_feed_vec string) CASCADE;
ALTER TABLE uaa.bi_new_user_realtimefeature add columns (devApplist string) CASCADE;

ALTER TABLE uaa.bi_questionnaire_label_details add columns (marriaged_label string, lianpu_marriage_status string) CASCADE;

ALTER TABLE uaa.midd_device_sparse_bisdk_app add columns (install_years double) CASCADE;

ALTER TABLE uaa.student_feature_id_mapping CHANGE COLUMN device_id feature_name string;

ALTER TABLE midd_vip_dense_feature_base CHANGE COLUMN pay_year order_year string;

-- 修改字段名
ALTER TABLE midd_cdum_device_activeness CHANGE COLUMN vv_count platform_di int;
```

**动态分区**

```sql
-- 不用分多个端各写一份代码
insert overwrite table uaa.device_wifiap_feature_backlist partition (dt='${target_date}', platform)
select adsid, platform from uaa.base_mobile_pingback_action_wifiap where dt='${target_date}' and adsid is not NULL and ap_mac is not NULL group by adsid,platform having count(*) < 1000;
```

**多层partition**

```sql
/data/uaa/commons/cdum/features/dev_feature_vec/2019-02-17/mobile/android/IP/IP_LIST
/data/uaa/commons/cdum/features/dev_feature_vec/2019-02-17/mobile/ios/IP/IP_LIST
/data/uaa/commons/cdum/features/dev_feature_vec/2019-02-17/pc/pc/IP/IP_LIST
以上数据如果用
/data/uaa/commons/cdum/features/dev_feature_vec/${dt}/${platform}/${subplatform}/${category}/${subcategory}
多层动态分区

那么如下sql 将不会
select device_id, feature_vec from uaa.midd_cdum_dev_feature_vec
where dt = '2019-02-17' and platform = 'mobile' and category = 'IP' and subcategory = 'IP_LIST';
--有条件中缺少subplatform， 所以会从/data/uaa/commons/cdum/features/dev_feature_vec/${dt}/${platform} 这个目录下把数据全部加载进来

```

**移动任务队列**

```shell
yarn application -movetoqueue application_1628490930009_3861728 -queue root.bi_uaa.cdum

yarn application -movetoqueue application_1624481688026_11625116 -queue root.bi_uaa.rtta
```

**倾斜检测**

```sql
--取30M数据采样查看key的分布
select passport_id, cnt from (SELECT passport_id, count(*) as cnt FROM midd_cdum_startup_vip_passport_device TABLE (30M) where dt='2019-02-17' group by passport_id)a order by cnt desc limit 100;

select device_id, cnt from (SELECT device_id, count(*) as cnt FROM uaa.device_dense_feature TABLESAMPLE (30M) where dt='2019-10-19' and platform='android' group by device_id)a order by cnt desc limit 100;
```

**倾斜优化设置**

```sql
SET hive.optimize.skewjoin=true;
SET hive.groupby.skewindata=true;
```

**建立临时中间表**

```sql
SET hive.groupby.skewindata=true;
with tmp_distinct_connect as (
select mac, ssid, device_id from uaa.cdum_device_wifi_relation_daily_clean where dt='2019-05-14'
and source in ('bisdk_mobile', 'vv') and is_connected = 1 group by mac, ssid, device_id
)
insert overwrite table uaa.device_relation_daily_merge partition (dt='2019-05-14')
select a.mac, a.ssid, a.device_id, b.device_cnt from 
tmp_distinct_connect a
join
(
select mac, ssid, count(*) as device_cnt from tmp_distinct_connect group by mac, ssid
) b
on a.mac = b.mac and a.ssid = b.ssid;
```


```sql
select passport_a, passport_b, max(prob) as prob from 
(
select passport_a, passport_b, prob from uaa.device_relation_passport_pair_output where dt='${dt}'
union all
select passport_b as passport_a, passport_a as passport_b, prob from uaa.device_relation_passport_pair_output where dt='${dt}'
) a 
group by passport_a, passport_b;
```

**MapJoin**

```sql
set hive.auto.convert.join=true;
set hive.mapjoin.smalltable.filesize=25000000;
set hive.auto.convert.join.noconditionaltask=true;
set hive.auto.convert.join.noconditionaltask.size=100000000;
```

如果出现如下报错，一定要检查小表的行数是否过大

如果想调整maximum memory 需要在命令行中设置 export HADOOP_CLIENT_OPTS="-Xmx2048m";

但是这里调大以后，可能会map运算中可能会出现Error: Java heap space

![hive_mapjoin](D:\工作相关\技术积累\knowledgeSummary\6.大数据处理\hive_mapjoin.jpeg)

**窗口函数**

```sql
-- row_number 用于组内排序
Row_Number() OVER (partition by geohash5 ORDER BY count(1) desc

-- sum 用于求窗口内的累计值
sum(dev_cnt) over (partition by city_id order by dev_cnt desc) as accum
-- for example 源数据
-- city_id | geohash | dev_cnt
863100  wt031 100
863100  wt032 200                   
863100  wt232 300
863100  wt332 400
861100  wx111 100  
861100  wx131 150
861100  wx032 200                   
861100  wx222 250
861100  wx122 300
                   
=> 
863100 wt332  0.4
863100 wt232  0.7  
863100 wt032  0.9
863100 wt031  1.0         
861100  wx122 0.3
861100  wx222 0.55
861100  wx032 0.75
861100  wx131 0.9 
861100  wx111 1.0
```

hive 运行慢，reduce报错143

一般怀疑是内存不够

```sql
set mapreduce.map.memory.mb=16384;
set mapreduce.map.java.opts=-Xmx13106M;   
set mapred.map.child.java.opts=-Xmx13106M;
set mapreduce.reduce.memory.mb=16384;
set mapreduce.reduce.java.opts=-Xmx13106M; --reduce.memory*0.8
set mapreduce.task.io.sort.mb=512;

set mapreduce.job.reduces=500;
set mapreduce.reduce.tasks = 800;
```

**HIVE OOM**

https://blog.csdn.net/yisun123456/article/details/81327372

**增加Map数量**
```sql
set mapred.max.split.size = 64000000;
```

