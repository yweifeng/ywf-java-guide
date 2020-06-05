---
title: mybatis基础
category: mybatis
order: 2
---



> 备注：demo中，数据库使用mysql、基于springboot集成mybatis
>
> 项目代码：https://github.com/yweifeng/ywf-mybatis.git



### mybatis 入门

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
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=utf8mb4;

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



### mybatis CRUD

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



### mybatis 一对多

- **模拟数据**

```mysql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

DROP TABLE IF EXISTS `dept`;
CREATE TABLE `dept` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `dept_name` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=utf8mb4;

CREATE TABLE `employ` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `employ_name` varchar(255) NOT NULL,
  `dept_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=utf8mb4;

BEGIN;
INSERT INTO `dept` VALUES (1, '研发部');
INSERT INTO `dept` VALUES (2, '人事部');
INSERT INTO `employ` VALUES (1, 'ywf', '1');
INSERT INTO `employ` VALUES (2, 'zhangsan', '1');
INSERT INTO `employ` VALUES (3, 'lisi', '2');
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```



- **Dept.java**

```java
package com.ywf.mybatis.entity;

import java.util.ArrayList;
import java.util.List;

/**
 * @Author:ywf
 */
public class Dept {
    private int id;
    private String deptName;
    List<Employ> employList = new ArrayList<>();

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getDeptName() {
        return deptName;
    }

    public void setDeptName(String deptName) {
        this.deptName = deptName;
    }

    public List<Employ> getEmployList() {
        return employList;
    }

    public void setEmployList(List<Employ> employList) {
        this.employList = employList;
    }

    @Override
    public String toString() {
        return "Dept{" +
                "id=" + id +
                ", deptName='" + deptName + '\'' +
                ", employList=" + employList +
                '}';
    }
}
```

- **Employ.java**

```java
package com.ywf.mybatis.entity;

/**
 * @Author:ywf
 */
public class Employ {
    private int id;
    private String employName;
    private Dept dept;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getEmployName() {
        return employName;
    }

    public void setEmployName(String employName) {
        this.employName = employName;
    }

    public Dept getDept() {
        return dept;
    }

    public void setDept(Dept dept) {
        this.dept = dept;
    }

    @Override
    public String toString() {
        return "Employ{" +
                "id=" + id +
                ", employName='" + employName + '\'' +
                ", dept=" + dept +
                '}';
    }
}
```

- **mybatis-config.xml**

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
        <typeAlias alias="Dept" type="com.ywf.mybatis.entity.Dept"/>
        <typeAlias alias="Employ" type="com.ywf.mybatis.entity.Employ"/>
    </typeAliases>
</configuration>
```

- **DeptMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IDeptMapper">

    <resultMap type="Dept" id="deptResultMap">
        <result property="id" column="dept_id"/>
        <result property="deptName" column="dept_name"/>
        <collection property="employList" ofType="Employ" column="dept_id">
            <id property="id" column="employ_id"></id>
            <result property="employName" column="employ_name"/>
        </collection>
    </resultMap>
    <!-- 获取所有部门信息，包含部门底下的员工 -->
    <select id="findAll" resultMap="deptResultMap">
        SELECT d.*,e.id as employ_id, e.employ_name FROM dept d INNER JOIN employ e on e.dept_id = d.id
    </select>
</mapper>
```



### mybatis 多对一

- **EmployMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IEmployMapper">

    <resultMap type="Employ" id="employResultMap">
        <result property="id" column="employ_id"/>
        <result property="employName" column="employ_name"/>
        <association property="dept" javaType="Dept">
            <id property="id" column="dept_id"></id>
            <result property="deptName" column="dept_name"/>
        </association>
    </resultMap>
    <!-- 获取所有员工信息，并关联部门 -->
    <select id="findAll" resultMap="employResultMap">
        SELECT e.id as employ_id, e.employ_name, d.* FROM employ e LEFT JOIN dept d ON e.dept_id = d.id
    </select>
</mapper>
```



### mybatis 多对多

- 模拟数据

```mysql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

DROP TABLE IF EXISTS `student`;
CREATE TABLE `student` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `student_name` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=utf8mb4;

CREATE TABLE `course` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `course_name` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=utf8mb4;

CREATE TABLE `student_to_course` (
  `student_id` int(11) NOT NULL,
  `course_id` int(11)  NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

BEGIN;
INSERT INTO `student` VALUES (1, 'ywf');
INSERT INTO `student` VALUES (2, 'zhangsan');
INSERT INTO `course` VALUES (1, '语文课');
INSERT INTO `course` VALUES (2, '体育课');
INSERT INTO `student_to_course` VALUES (1, 1);
INSERT INTO `student_to_course` VALUES (2, 1);
INSERT INTO `student_to_course` VALUES (2, 2);
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```

- **Student.java**

```java
package com.ywf.mybatis.entity;

import java.util.ArrayList;
import java.util.List;

/**
 * @Author:ywf
 */
public class Student {
    private int id;
    private String studentName;
    List<Course> courseList = new ArrayList<>();

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getStudentName() {
        return studentName;
    }

    public void setStudentName(String studentName) {
        this.studentName = studentName;
    }

    public List<Course> getCourseList() {
        return courseList;
    }

    public void setCourseList(List<Course> courseList) {
        this.courseList = courseList;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", studentName='" + studentName + '\'' +
                ", courseList=" + courseList +
                '}';
    }
}
```

- **Course.java**

```java
package com.ywf.mybatis.entity;

import java.util.ArrayList;
import java.util.List;

/**
 * @Author:ywf
 */
public class Course {
    private int id;
    private String courseName;
    List<Student> studentList = new ArrayList<>();

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getCourseName() {
        return courseName;
    }

    public void setCourseName(String courseName) {
        this.courseName = courseName;
    }

    public List<Student> getStudentList() {
        return studentList;
    }

    public void setStudentList(List<Student> studentList) {
        this.studentList = studentList;
    }

    @Override
    public String toString() {
        return "Course{" +
                "id=" + id +
                ", courseName='" + courseName + '\'' +
                ", studentList=" + studentList +
                '}';
    }
}
```

- **mybatis-config.xml**

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
        <typeAlias alias="Student" type="com.ywf.mybatis.entity.Student"/>
        <typeAlias alias="Course" type="com.ywf.mybatis.entity.Course"/>
    </typeAliases>
</configuration>
```

- **StudentMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IStudentMapper">

    <resultMap type="Student" id="studentResultMap">
        <result property="id" column="s_id"/>
        <result property="studentName" column="student_name"/>
        <collection property="courseList" ofType="Course" column="c_id">
            <id property="id" column="c_id"></id>
            <result property="courseName" column="course_name"/>
        </collection>
    </resultMap>
    <!-- 获取所有学生信息，包含学生学习的课程信息 -->
    <select id="findAll" resultMap="studentResultMap">
        SELECT s.*, s.id as s_id,c.*, c.id as c_id FROM student s, course c, student_to_course sc
        where sc.student_id = s.id and sc.course_id = c.id
    </select>
</mapper>
```

