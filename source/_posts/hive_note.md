##Hive Note ##

### Hive 实现 WordCount 示例程序 ###

CREATE TABLE docs (lines STRING);
LOAD DATA INPUT 'docs' OVERWRITE INTO TABLE docs;
CREATE TABLE word_counts AS
SELECT word, count(1) As count FROM 
  (SELECT explode(split(line, 's')) AS word FROM docs) w
GROUP BY word
ORDER BY word;

### Hive 中的命名空间 ###

| 命名空间        | 使用权限           | 描述  |
| ------------- |:-------------:| ---------------:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |
