---
title: node安装
category: linux
order: 4
---



- 到[node官网](http://nodejs.cn/download/)下载linux版本，有32和64位版本

- 将文件上传到linux下

- 使用tar -xvf node-v8.9.3-linux-x64.tar.xz 进行解压
- 使用mv mv node-v12.13.1-linux-x64 node 修改名称
- 建立软连接，变为全局

```shell
ln -s /opt/node/bin/npm /usr/local/bin/

ln -s /opt/node/bin/node /usr/local/bin/
```

- 在Linux命令行node -v 命令会显示nodejs版本** 

- 安装cnpm 

- npm install -g cnpm --registry=https://registry.npm.taobao.org 