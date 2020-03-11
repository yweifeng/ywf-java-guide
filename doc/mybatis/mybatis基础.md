# mybatis基础

> 备注：demo中，数据库使用mysql、基于springboot集成mybatis

## mybatis 入门

- **创建数据库**

```mysql
CREATE DATABASE demo;
```

- **建表和生成模拟数据**

```mysql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_name` varchar(255) NOT NULL,
  `user_password` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=latin1;

-- ----------------------------
-- Records of user
-- ----------------------------
BEGIN;
INSERT INTO `user` VALUES (1, 'ywf', '13');
INSERT INTO `user` VALUES (2, 'zhangsan', '123');
INSERT INTO `user` VALUES (3, 'lisi', '123');
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```

- **pom.xml**

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.1</version>
</dependency>
```

- **application.yml**

```yaml
mybatis:
  check-config-location: true
  config-location: classpath:mybatis/mybatis-config.xml
  mapper-locations: classpath*:mapper/*Mapper.xml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://192.168.60.15:3306/demo?characterEncoding=utf8&useSSL=false
    username: root
    password: linewell@2016
```

- **mybatis/mybatis-config.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD SQL Map Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 开启下划线转驼峰 -->
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    <typeAliases>
        <typeAlias alias="Integer" type="java.lang.Integer" />
        <typeAlias alias="Long" type="java.lang.Long" />
        <typeAlias alias="HashMap" type="java.util.HashMap" />
        <typeAlias alias="LinkedHashMap" type="java.util.LinkedHashMap" />
        <typeAlias alias="ArrayList" type="java.util.ArrayList" />
        <typeAlias alias="LinkedList" type="java.util.LinkedList" />
        <typeAlias alias="User" type="com.ywf.mybatis.entity.User"/>
    </typeAliases>
</configuration>
```

- **实体类 User.java**

```java
package com.ywf.mybatis.entity;

/**
 * @Author:ywf
 */
public class User{
    private int id;
    private String userName;
    private String userPassword;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getUserPassword() {
        return userPassword;
    }

    public void setUserPassword(String userPassword) {
        this.userPassword = userPassword;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", userName='" + userName + '\'' +
                ", userPassword='" + userPassword + '\'' +
                '}';
    }
}
```

- **IUserService.java**

```java
package com.ywf.mybatis.service;

import com.ywf.mybatis.entity.User;

import java.util.List;

/**
 * @Author:ywf
 */
public interface IUserService {

    /**
     * 查询所有用户
     * @return 用户列表
     */
    List<User> findAll();
}
```

- **UserServiceImpl.java**

```java
package com.ywf.mybatis.service.impl;

import com.ywf.mybatis.entity.User;
import com.ywf.mybatis.mapper.IUserMapper;
import com.ywf.mybatis.service.IUserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @Author:ywf
 */
@Service
public class UserServiceImpl implements IUserService {

    @Autowired
    private IUserMapper userMapper;

    @Override
    public List<User> findAll() {
        return userMapper.findAll();
    }
}
```

- **IUserMapper.java**

```java
package com.ywf.mybatis.mapper;

import com.ywf.mybatis.entity.User;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;

/**
 * @Author:ywf
 */
@Mapper
public interface IUserMapper {
    List<User> findAll();
}
```

- **mapper/UserMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IUserMapper">
    <select id="findAll" resultType="User">
        SELECT * FROM user
    </select>
</mapper>
```

- **MybatisApplicationTests.java**

```java
package com.ywf.mybatis;

import com.ywf.mybatis.entity.User;
import com.ywf.mybatis.service.IUserService;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
class MybatisApplicationTests {
    private Logger log = LoggerFactory.getLogger(this.getClass());

    @Autowired
    private IUserService userService;

    @Test
    void findAll() {
        List<User> userList = userService.findAll();
        log.info(userList.toString());
    }

}
```

- **运行结果**

```
 [User{id=1, userName='ywf', userPassword='13'}, User{id=2, userName='zhangsan', userPassword='123'}, User{id=3, userName='lisi', userPassword='123'}]
```



## mybatis CRUD

- **UserMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IUserMapper">
    <!-- 获取所有用户信息 -->
    <select id="findAll" resultType="User">
        SELECT * FROM user
    </select>

    <!-- 根据id获取用户信息 -->
    <select id="getById" resultType="User" parameterType="Integer">
        SELECT * FROM user WHERE id = #{id}
    </select>

    <!-- 新增用户 -->
    <insert id="insert" parameterType="User">
        insert into user (user_name, user_password) values (#{userName}, #{userPassword})
    </insert>

    <!-- 更新用户-->
    <update id="update" parameterType="User">
        update user
        <set>
            <if test="userName != null">user_name = #{userName}</if>
            <if test="userPassword != null">user_password = #{userPassword}</if>
        </set>
        where id = #{id}
    </update>

    <!-- 删除用户 -->
    <delete id="delete" parameterType="Integer">
        delete from user where id = #{id}
    </delete>

</mapper>
```



## mybatis 注解

- **IUserAnnotationMapper.java**

```java
package com.ywf.mybatis.mapper;

import com.ywf.mybatis.entity.User;
import org.apache.ibatis.annotations.*;

import java.util.List;

/**
 * @Author:ywf
 */
@Mapper
public interface IUserAnnotationMapper {
    /**
     * 查询所有用户
     * @return 用户列表
     */
    @Select("SELECT * FROM user")
    List<User> findAll();

    /**
     * 根据id 获取用户信息
     * @param id 用户id
     * @return 用户信息
     */
    @Select("SELECT * FROM user WHERE id = #{id} ")
    User getById(int id);

    /**
     * 新增用户
     * @param user 用户信息
     * @return
     */
    @Insert("INSERT INTO user(user_name, user_password) VALUES(#{userName}, #{userPassword}) ")
    int insert(User user);

    /**
     * 根据id 删除用户
     * @param id
     * @return
     */
    @Delete("DELETE FROM user WHERE id = #{id}")
    int delete(int id);

    /**
     * 更新用户信息
     * @param user 用户信息
     * @return
     */
    @Update("UPDATE user SET user_name = #{userName}, user_password = #{userPassword} WHERE id = #{id}")
    int update(User user);
}
```



## mybatis 一对多



## mybatis 多对一



## mybatis 多对多