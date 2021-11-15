1. 某一个stage仅有3个mapper，处于running状态一直卡住不动？

   思路： 怀疑mapper的input数据量太大，措施1： 把reduces的数量设置大一点； 措施2： 查看hql中是否有错误导致过大的笛卡尔集

   结果： 不起作用，最终发现是left join 时条件写错了

2. 两个device都做过去重group过的sub-query 在left join阶段出现倾斜

   原因： 因为关联字段是passport，一张表的passport字段有大量NULL值，会导致倾斜。

   方案： 把passport大量为NULL的表中用COALESCE(passport, 100000000000+ rand() * 2000) as joinfield

3. 永丰集群创建外表，alter添加分区，出现权限错误

   原因：/data/bi_uaa/属于另外一个namenode

   方案：

   ```sql
   ALTER TABLE uaa.consumption_features add if not exists partition(dt='2019-02-14') 
   location 'viewfs://hadoop-bd/data/bi_uaa/products/consumption_level/features/2019-02-14';
   -- 必须有前缀 viewfs://hadoop-bd/
   ```

4. 一个表连续join两个表，最终有1个task在66.75%这里卡住了，重现好多次

   参考：

   > Question:
   >
   > Trying to join 6 tables which are having 5 million rows approximately in each table. Trying to join on account number which is sorted in ascending order on all tables. Map tasks are successfully finished and reducers stopped working at 66.68%. Tried options like increasing number of reducers and also tried other options set hive.auto.convert.join = true; and set hive.hashtable.max.memory.usage = 0.9; and set hive.smalltable.filesize = 25000000L; but the result is same. Tried with small number of records (like 5000 rows) and the query works really well.
   >
   > Please suggest what can be done here to make it work.

   > Answer:
   >
   > Reducers at 66% start doing the actual reduce (0-33% is shuffle, 33-66% is sort). In a join with hive, the reducer is performing a Cartesian product between the two data sets.
   >
   > I'm going to guess that there is at least one foreign key that is appearing frequently in all of the data sets. Watch for NULL and default values.
   >
   > For example, in a join, imagine the key "abc" appears ten times in each of the six tables (10^6). That's a million output records for that one key. If "abc" appears 1000 times in one table, 1000 in another, 1000 in another, then twice in the other three tables, you get 8 billion records (1000^3 * 2^3). You can see how this gets out of hand. I'm guessing there is at least one key that is resulting in a massive number of output records.

5.  hive 中文乱码问题

   把join字段的中的特殊字符给replace掉

   ```
   regexp_replace(regexp_replace(b.word,'\\n',''), '\\t', '')
   ```
