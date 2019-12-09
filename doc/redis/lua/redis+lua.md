## Redis中使用Lua的好处

- 减少网络开销。可以将多个请求通过脚本的形式一次发送，减少网络时延
- 原子操作。redis会将整个脚本作为一个整体执行，中间不会被其他命令插入。因此在编写脚本的过程中无需担心会出现竞态条件，无需使用事务。
- 复用。客户端发送的脚步会永久存在redis中，这样，其他客户端可以复用这一脚本而不需要使用代码完成相同的逻辑。



##  redis 脚本常用命令

[Lua官方教程地址](https://www.runoob.com/lua/lua-tutorial.html)

- [ EVAL script numkeys key [key ...\] arg [arg ...]](https://www.w3cschool.cn/redis/scripting-eval.html) 

  执行 Lua 脚本。

- [EVALSHA sha1 numkeys key [key ...\] arg [arg ...]](https://www.w3cschool.cn/redis/scripting-evalsha.html)
  执行 Lua 脚本。

- [SCRIPT EXISTS script script ...](https://www.w3cschool.cn/redis/scripting-script-exists.html)
  查看指定的脚本是否已经被保存在缓存当中。

- [SCRIPT FLUSH](https://www.w3cschool.cn/redis/scripting-script-flush.html)
  从脚本缓存中移除所有脚本。

- [ SCRIPT KILL](https://www.w3cschool.cn/redis/scripting-script-kill.html) 

  杀死当前正在运行的 Lua 脚本。

- [SCRIPT LOAD script](https://www.w3cschool.cn/redis/scripting-script-load.html)
  将脚本 script 添加到脚本缓存中，但并不立即执行这个脚本。



## 编写简单的lua脚本运行

### 方式一 EVAL

```shell
EVAL "return 'hello word'" 0
"hello word"
```



### 方式二 SCRIPT LOAD

```shell
SCRIPT LOAD "return 'hello word'"
"45396e13903003ec8916158e409f5f89dee43ed1"
```



### 方式三 EVALSHA

```shell
EVALSHA 45396e13903003ec8916158e409f5f89dee43ed1 0
"hello word"
```



### 方式四 加载lua脚本文件

```shell
# 与redis-cli同目录创建lua目录

# 创建helloWorld.lua文件

redis-cli --eval lua/helloWorld.lua
"hello world"
```



### 原子性测试

```shell
EVAL "redis.call('mset', KEYS[1],ARGV[1], KEYS[2],ARGV[2]) redis.call('mset', KEYS[3],ARGV[3], KEYS[4],ARGV[4]) return 0" 4 name1 name2 name3 name4 ywf1 ywf2 ywf3 ywf4
```

**返回值**

```shell
mget name1 name2 name3 name4
1) "ywf1"
2) "ywf2"
3) "ywf3"
4) "ywf4"
```

