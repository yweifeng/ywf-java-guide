---
title: ehcache基础
category: springboot
order: 1
---



### 1、特性

```
1. 快速
2. 简单
3. 多种缓存策略
4. 缓存数据有两级：内存和磁盘，因此无需担心容量问题
5. 缓存数据会在虚拟机重启的过程中写入磁盘
6. 可以通过RMI、可插入API等方式进行分布式缓存
7. 具有缓存和缓存管理器的侦听接口
8. 支持多缓存管理器实例，以及一个实例的多个缓存区域
9. 提供Hibernate的缓存实现
```

### 2、ehcache 和 redis 比较

| ehcache                          | redis                                     |
| -------------------------------- | ----------------------------------------- |
| 基于内存，速度快，分布式共享麻烦 | 使用socket，效率比ehcache慢，但分布式方便 |

### 3、Spring整合ehcache

#### 3.1 建立spring项目,引入依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ywf</groupId>
    <artifactId>springboot-ehcache</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- Spring Boot启动器父类 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <!-- Spring Boot web启动器 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-json</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!-- fastjson -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        <!-- cache -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
        <dependency>
            <groupId>net.sf.ehcache</groupId>
            <artifactId>ehcache</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

#### 3.2 新建application.yml

```properties
spring:
  cache:
    #ehcache配置文件路径
    ehcache:
      config: classpath:/ehcache/ehcache.xml
    #指定缓存类型，可加可不加
    #type: ehcache
server:
  port: 8080
```

#### 3.3 新建ehcache.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache name="myEncache">

    <!--
        diskStore:为缓存路径，ehcache分为内存和磁盘 2级，此属性定义磁盘的缓存位置
        user.home - 用户主目录
        user.dir - 用户当前工作目录
        java.io.tmpdir - 默认临时文件路径
    -->
    <diskStore path="D:/ehcache/Tmp_Ehcache"/>
    <!--
        name:缓存名称。
        maxElementsInMemory:缓存最大数目
        maxElementsOnDisk：硬盘最大缓存个数。
        eternal:对象是否永久有效，一但设置了，timeout将不起作用。
        overflowToDisk:是否保存到磁盘，当系统宕机时
        timeToIdleSeconds:设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
        timeToLiveSeconds:设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
        diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
        diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
        diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
        memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
        clearOnFlush：内存数量最大时是否清除。
        memoryStoreEvictionPolicy:可选策略有：LRU（最近最少使用，默认策略）、FIFO（先进先出）、LFU（最少访问次数）。
            FIFO，first in first out，这个是大家最熟的，先进先出。
            LFU， Less Frequently Used，就是上面例子中使用的策略，直白一点就是讲一直以来最少被使用的。如上面所讲，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。
            LRU，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。
-->
    <defaultCache
            eternal="false"
            maxElementsInMemory="1000"
            overflowToDisk="false"
            diskPersistent="false"
            timeToIdleSeconds="0"
            timeToLiveSeconds="600"
            memoryStoreEvictionPolicy="LRU"
    />
    <cache
            name="users_test"
            eternal="false"
            maxElementsInMemory="100"
            overflowToDisk="false"
            diskPersistent="false"
            timeToIdleSeconds="0"
            timeToLiveSeconds="300"
            memoryStoreEvictionPolicy="LRU"
    />

</ehcache>
```

#### 3.4 启动类加@EnableCaching

```java
package com.ywf;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@SpringBootApplication
@EnableCaching
public class SpringbootCacheApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootCacheApplication.class, args);
    }
}
```

#### 3.5 代码添加缓存注解

userController:

```java
package com.ywf.controller;

import com.ywf.domain.User;
import com.ywf.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping("/listUser")
    public List<User> listUser() {
        return userService.listUser();
    }

    @RequestMapping("/selectUserById")
    public User selectUserById(@RequestParam Integer userId) {
        return userService.selectUserById(userId);
    }

    @RequestMapping("/delete")
    public void delete(@RequestParam Integer userId) {
        userService.delete(userId);
    }

    @RequestMapping("/update")
    public void update(@RequestParam Integer userId, @RequestParam String userName) {
        User user = new User(userId, userName);
        userService.update(user);
    }
}
```

domain:

```java
package com.ywf.domain;

import java.io.Serializable;

public class User implements Serializable {

    private Integer userId;

    private String userName;

    public User(Integer userId, String userName) {
        this.userId = userId;
        this.userName = userName;
    }

    public Integer getUserId() {
        return userId;
    }

    public void setUserId(Integer userId) {
        this.userId = userId;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }
}
```

userService:

```java
package com.ywf.service;

import com.ywf.domain.User;

import java.util.List;

public interface UserService {
    List<User> listUser();

    User selectUserById(final Integer id);

    void delete(final Integer id);

    void update(final User user);
}
```

userServiceImpl:

```java
package com.ywf.service.impl;

import com.ywf.domain.User;
import com.ywf.service.UserService;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

import java.util.*;

@Service
public class UserServiceImpl implements UserService {

    // 模拟数据库已经有数据
    public static List<User> dbList = new ArrayList<>();

    static {
        dbList.add(new User(1, "ywf"));
        dbList.add(new User(2, "ywf2"));
        dbList.add(new User(3, "ywf3"));
    }

    //使用ehcache配置的缓存名users_test
    private final String CACHE_NAME = "users_test";

    @Override
    @Cacheable(value = CACHE_NAME)
    public List<User> listUser() {
        System.out.println("listUser from db");
        return dbList;
    }

    /**
     * 查询
     *
     * @param userId
     * @return
     */
    @Override
    @Cacheable(value = CACHE_NAME, key = "'user:'+ #userId")
    public User selectUserById(Integer userId) {
        System.out.println("selectUserById from db");
        Optional<User> optional = dbList.stream().filter(p -> (Objects.equals(userId.toString(), p.getUserId().toString()))).findFirst();
        return (optional == null || !optional.isPresent()) ? null : optional.get();
    }

    /**
     * 删除
     *
     * @param userId
     */
    @Override
    @CacheEvict(value = CACHE_NAME, key = "'user:'+ #userId")
    public void delete(Integer userId) {
        System.out.println("delete from db");
        Iterator<User> it = dbList.iterator();
        while (it.hasNext()) {
            User user = (User) it.next();
            if (Objects.equals(userId.toString(), user.getUserId().toString())) {
                it.remove();
            }
        }
    }

    /**
     * 更新，先删除，下次查询再重新读取数据库
     *
     * @param user
     */
    @Override
    @CacheEvict(value = CACHE_NAME, key = "'user:'+ #user.userId")
    public void update(User user) {

        Iterator<User> it = dbList.iterator();
        while (it.hasNext()) {
            User dbUser = (User) it.next();
            if (Objects.equals(dbUser.getUserId().toString(), user.getUserId().toString())) {
                dbUser.setUserName(user.getUserName());
            }
        }
        System.out.println("update " + user.getUserId());
    }
}
```

