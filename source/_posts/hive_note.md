##Hive Note ##

### Hive 实现 WordCount 示例程序 ###

```
CREATE TABLE docs (lines STRING);
LOAD DATA INPUT 'docs' OVERWRITE INTO TABLE docs;
CREATE TABLE word_counts AS
SELECT word, count(1) As count FROM 
  (SELECT explode(split(line, 's')) AS word FROM docs) w
GROUP BY word
ORDER BY word;
```

### Hive 中的命名空间 ###

| 命名空间        | 使用权限           | 描述  |
| ------------- |:-------------:| ---------------:|
| hivevar      | 可读/可写 | (Hive v0.8.0 以及以后的版本) 用户自定义变量 |
| hiveconf      | 可读/可写      |  Hive 相关的配置属性 |
| system | 可读/可写  |    Java 定义的配置属性 |
| env    |  只可读           |   Shell 环境定义的环境变量       |

用户可以自定义变量以便在 Hive 脚本中使用。
