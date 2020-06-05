---
title: scp远程拷贝文件
category: linux
order: 1
---



```shell
复制文件（本地>>远程）：scp /cloud/data/test.txt root@10.21.156.6:/cloud/data/
复制文件（远程>>远程）：scp root@10.21.156.6:/cloud/data/test.txt /cloud/data/
复制目录（本地>>远程）：scp -r /cloud/data root@10.21.156.6:/cloud/data/
复制目录（远程>>本地）：scp -r root@10.21.156.6:/cloud/data/  /cloud/data/	
```

