# node安装

**1.到[node官网](http://nodejs.cn/download/)下载linux版本，有32和64位版本** 

![img](img/node.png)

2. **将文件上传到linux下** 
3. **使用tar -xvf node-v8.9.3-linux-x64.tar.xz 进行解压**
4. **使用mv mv node-v12.13.1-linux-x64 node 修改名称**
5. **建立软连接，变为全局** 

```shell
ln -s /opt/node/bin/npm /usr/local/bin/

ln -s /opt/node/bin/node /usr/local/bin/
```

5. **在Linux命令行node -v 命令会显示nodejs版本**** 
6. **安装cnpm** 
7. **npm install -g cnpm --registry=https://registry.npm.taobao.org** 