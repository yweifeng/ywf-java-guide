# hdfs常用命令

### 查看hdfs帮助

```shell
hdfs dfs -help
```

### 查看目录文件

```shell
hdfs dfs -ls /
```

### 上传文件

```shell
# 上传文件，已存在，提示
hdfs dfs -put linux路径 hdfs路径
# -f 覆盖已有文件 
hdfs dfs -put -f linux路径 hdfs路径
```

### 下载文件

```shell
hdfs dfs -get hdfs路径 linux路径
```

### 查看文件

```shell
hdfs dfs -cat hdfs文件路径
```

### 创建文件夹

```shell
hdfs dfs -mkdir hdfs文件夹名称
```

### 创建文件

```shell
hdfs dfs -touchz hdfs文件路径
```

### 向现有文件追加内容

```shell
hdfs dfs -appendToFile 追加文件路径 hdfs文件路径
```

### 合并内容

```shell
hdfs dfs -getmerge hdfs文件路径1 hdfs文件路径2
```

### 删除文件

```shell
# 删除
hdfs dfs -rm hdfs文件路径
# 递归删除
hdfs dfs -rm -R hdfs文件路径
```
