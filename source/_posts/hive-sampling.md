title: HIVE 表随机抽样
date: 2019/12/10
tags: [hive, Sample]

-------
如何在 hive 表中抽样尽可能随机的数据？
<!-- more -->
## Hive 数据抽样

当前希望能在hive表中抽样 10w 条数据，抽取的数据是随机的。

### 方法一（错误❌）

这种方式每次抽取的数据都是一样的，是按照文件的固定顺序抽取的。

```sql
select * from my_table limit 100000;
```

### 方法二（不推荐）

通过 `order by` 和 `rand` 结合来抽取数据：

```sql
select * from my_table
order by rand()
limit 100000;
```

这种方式抽取的数据是随机的，但是当数据量较大时会存在性能问题。`order by` 会将所有的数据放在一个 `reducer` 里面进行全局排序。

### 方法三

使用 `sort by` 进行筛选。`sort by` 会保证数据在 `reducer` 里面是有序的，不能保证全局有序。

```sql
select * from my_table
sort by rand()
limit 100000;
```

> This is much better, but I’m not convinced it’s truly random. The issue is that Hive’s method of <font color=red>splitting data into multiple reducers is undefined</font>. It might be truly random, it might be **based on file order**, it might be **based on some value in the data**. How Hive implements the **limit clause across reducers is also undefined**. Maybe it takes data from the reducers in order- i.e., all data from reducer 0, then all from reducer 1, etc. Maybe it round-robins through them and mixes everything together.
>
> In the worst case, let’s say the reduce key is based on a column of the data, and the limit clause is in order of reducers. Then the sample will be extremely skewed.



### 方法四

使用 `distribute by` 和 `sort by` 抽取随机数据。

`distribute by` 会把数据按照后面的 `key` 进行分组，相同的 `key` 合并到同一个 `reduce` 中进行处理，然后 `sort by` 对同一个 `reduce` 里面的数据进行排序，从而实现真正的随机采样。

```sql
select * from my_table
distribute by rand()
sort by rand()
limit 100000;
```

当然为了进一步提高执行效率，可以在 map 进行数据筛选：

```sql
select * from my_table
where rand() < 0.0001
distribute by rand()
sort by rand()
limit 100000;
```

where 后面的限制条件取值一般是计算方式：

10 * 抽样数据量 / 总样本数

> In this case, since the total size is ten billion, and the sample size is ten thousand, I can easily calculate that’s exactly 0.000001 of the total data. However, if the where clause was “rand() < 0.000001”, it would be possible to end up with fewer than 10000 rows output. “rand() < 0.000002” would probably work, but that’s really relying on a very good implementation of rand(). Safer to fudge it by a bit more. In the end it doesn’t matter much since the bottleneck quickly becomes the simple full table scan to start the job, and not anything based on the volume sent to reducers.



## 参考文献

[random sampling in hive](http://www.joefkelley.com/736/)



