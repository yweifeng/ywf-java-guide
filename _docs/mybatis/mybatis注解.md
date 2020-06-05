---
title: mybatis注解
category: mybatis
order: 3
---



### mybatis注解 CRUD

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



### mybatis注解 一对多

- **IDeptAnnotationMapper.java**

```java
package com.ywf.mybatis.mapper.annotation;

import com.ywf.mybatis.entity.Dept;
import com.ywf.mybatis.entity.Employ;
import org.apache.ibatis.annotations.*;

import java.util.List;

/**
 * @Author:ywf
 */
@Mapper
public interface IDeptAnnotationMapper {

    /**
     * 获取部门信息，包含该部门底下的员工信息
     * @return
     */
    @Select("SELECT * FROM dept")
    @Results({
            @Result(property = "id", column = "id"),
            @Result(property = "deptName", column = "dept_name"),
            @Result(property = "employList", javaType = List.class, column = "id",
                    many = @Many(select = "com.ywf.mybatis.mapper.annotation.IEmployAnnotationMapper.findByDeptId"))
    })
    List<Dept> findAll();
}
```

- **IEmployAnnotationMapper.java**

```java
package com.ywf.mybatis.mapper.annotation;

import com.ywf.mybatis.entity.Employ;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

import java.util.List;

/**
 * @Author:ywf
 */
@Mapper
public interface IEmployAnnotationMapper {
    /**
     * 根据部门ID 查询员工信息
     * @param deptId 部门ID
     * @return
     */
    @Select("SELECT * FROM employ WHERE dept_id = #{deptId}")
    List<Employ> findByDeptId(int deptId);
}
```



### mybatis注解 多对一

- **IEmployAnnotationMapper.java**

```java
package com.ywf.mybatis.mapper.annotation;

import com.ywf.mybatis.entity.Employ;
import org.apache.ibatis.annotations.*;

import java.util.List;

/**
 * @Author:ywf
 */
@Mapper
public interface IEmployAnnotationMapper {

    /**
     * 获取所有员工信息，包含员工的部门信息
     * @return
     */
    @Select("SELECT * FROM employ")
    @Results({
            @Result(column = "dept_id", property = "dept",
                    one = @One(select = "com.ywf.mybatis.mapper.annotation.IDeptAnnotationMapper.getById"))
    })
    List<Employ> findAll();
}
```

- **IDeptAnnotationMapper.java**

```java
package com.ywf.mybatis.mapper.annotation;

import com.ywf.mybatis.entity.Dept;
import com.ywf.mybatis.entity.Employ;
import org.apache.ibatis.annotations.*;

import java.util.List;

/**
 * @Author:ywf
 */
@Mapper
public interface IDeptAnnotationMapper {

    /**
     * 根据ID获取部门信息
     * @param id
     * @return
     */
    @Select("SELECT * FROM dept where id = #{id}")
    Dept getById(int id);
}
```



### mybatis注解 多对多

- **IStudentAnnotationMapper.java**

```java
package com.ywf.mybatis.mapper.annotation;

import com.ywf.mybatis.entity.Student;
import org.apache.ibatis.annotations.*;

import java.util.List;

/**
 * @Author:ywf
 */
@Mapper
public interface IStudentAnnotationMapper {

    /**
     * 获取所有学生信息，包含学生学习的课程信息
     * @return
     */
    @Select("SELECT * FROM student")
    @Results({
            @Result(column = "id", property = "id", id = true),
            @Result(column = "student_name", property = "studentName"),
            @Result(column = "id", property = "courseList",
                many = @Many(select = "com.ywf.mybatis.mapper.annotation.ICourseAnnotationMapper.selectByStudentId"))
    })
    List<Student> findAll();
}
```

- **ICourseAnnotationMapper.java**

```java
package com.ywf.mybatis.mapper.annotation;

import com.ywf.mybatis.entity.Course;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

/**
 * @Author:ywf
 */
@Mapper
public interface ICourseAnnotationMapper {

    /**
     * 根据课程ID获取课程信息
     * @param studentId 学生id
     * @return
     */
    @Select("SELECT * FROM course WHERE id IN (SELECT course_id FROM student_to_course WHERE student_id = #{student_id})")
    Course selectByStudentId(int studentId);
}
```