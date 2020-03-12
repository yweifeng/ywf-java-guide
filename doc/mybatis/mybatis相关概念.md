# mybatis相关概念

## mybatis 事务管理

**@Transactional注解属性如下**

![img](img/mybatis1.png)



### 隔离级别

隔离级别是指若干个并发的事务之间的隔离程度，与我们开发时候主要相关的场景包括：脏读取、重复读、幻读。

我们可以看`org.springframework.transaction.annotation.Isolation`枚举类中定义了五个表示隔离级别的值：

```java
public enum Isolation {
    DEFAULT(-1),
    READ_UNCOMMITTED(1),
    READ_COMMITTED(2),
    REPEATABLE_READ(4),
    SERIALIZABLE(8);
}
```

- `DEFAULT`：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是：`READ_COMMITTED`。
- `READ_UNCOMMITTED`：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读和不可重复读，因此很少使用该隔离级别。
- `READ_COMMITTED`：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
- `REPEATABLE_READ`：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。即使在多次查询之间有新增的数据满足该查询，这些新增的记录也会被忽略。该级别可以防止脏读和不可重复读。
- `SERIALIZABLE`：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

指定方法：通过使用**isolation**属性设置，例如：

```java
@Transactional(isolation = Isolation.DEFAULT)
```



### 传播行为

所谓事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。

我们可以看`org.springframework.transaction.annotation.Propagation`枚举类中定义了6个表示传播行为的枚举值：

```java
public enum Propagation {
    REQUIRED(0),
    SUPPORTS(1),
    MANDATORY(2),
    REQUIRES_NEW(3),
    NOT_SUPPORTED(4),
    NEVER(5),
    NESTED(6);
}
```

- REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
- SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
- REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
- NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于REQUIRED。

指定方法：通过使用**propagation**属性设置，例如：

```java
@Transactional(propagation = Propagation.REQUIRED)
```



### 用法

@Transactional 可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 **public** 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。

虽然 @Transactional 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。另外， @Transactional 注解应该只被**应用到 public 方法上**，这是由 Spring AOP 的本质决定的。如果你在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，这将被忽略，也不会抛出任何异常。

默认情况下，只有来自外部的方法调用才会被AOP代理捕获，也就是，类内部方法调用本类内部的其他方法并不会引起事务行为，即使被调用方法使@Transactional注解进行修饰。

在使用@Transactional注解前，请在**启动类**上加上注解@**EnableTransactionManagement**来开启事务。

```java
@EnableTransactionManagement  //开启事务
public class AdminApplication {

    public static void main(String[] args) {
        SpringApplication.run(AdminApplication.class, args);
    }

}
```

使用示例：

```java
@Transactional(isolation = Isolation.REPEATABLE_READ,propagation = Propagation.REQUIRED,rollbackFor = Exception.class)
```



## mybatis 延迟加载

###  什么是延迟加载

- 就是在需要用到数据的时候才进行加载，不需要用到数据的时候就不加载数据。延迟加载也称为懒加载。 

- **优点：**先从单表查询，需要时再从关联表去关联查询，大大提高数据库的性能，因为查询单表要比关联查询多张表的速度快很多。

- **缺点：**因为只有当需要用到数据时，才会进行数据库查询，这样在大批量数据查询时，因为查询工作也需要耗费时间，所以可能造成用户等待时间变长，造成用户体验下降。



### 如何实现延迟加载

#### 一对多实现延迟加载

> 场景：打印出所有部门的部门名称，此时不想加载用户信息

- **方式一：配置延迟加载的全局开关,参考配置**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD SQL Map Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- 延迟加载的全局开关 -->
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>
</configuration>
```

- **DeptMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IDeptMapper">
    <resultMap id="lazyDeptResultMap" type="Dept">
        <result column="d_id" property="id" />
        <result column="dept_name" property="deptName" />
        <collection property="employList" ofType="Employ" column="d_id"
                    select="com.ywf.mybatis.mapper.IEmployMapper.findByDeptId"
        />
    </resultMap>

    <!-- 获取所有部门信息，延迟加载部门底下的员工信息 -->
    <select id="lazyFindAll" resultMap="lazyDeptResultMap">
        SELECT d.id as d_id, d.dept_name FROM dept d
    </select>
</mapper>
```

- **IEmployMapper.java**

```java
package com.ywf.mybatis.mapper;

import com.ywf.mybatis.entity.Employ;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import java.util.List;

/**
 * @Author:ywf
 */
@Mapper
public interface IEmployMapper {
    /**
     * 根据部门Id 获取员工信息
     * @param deptId 部门ID
     * @return
     */
    List<Employ> findByDeptId(@Param("deptId") int deptId);
}

```

- **EmployMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IEmployMapper">

    <!-- 根据部门ID获取员工信息 -->
    <select id="findByDeptId" resultType="Employ" parameterType="Integer">
        SELECT * FROM employ where dept_id = #{deptId}
    </select>
</mapper>
```

- **调用**

```java
/**
  * 一对多 延迟加载
  */
@Test
void lazyOne2Many() {
    List<Dept> deptList = deptService.lazyFindAll();
    for (Dept dept : deptList) {
        log.info(dept.getDeptName());
    }
}
```



- **方式二：fetchType="lazy"**

- **DeptMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IDeptMapper">
    <resultMap id="lazyDeptResultMap" type="Dept">
        <result column="d_id" property="id" />
        <result column="dept_name" property="deptName" />
        <collection property="employList" ofType="Employ" column="d_id"
                    select="com.ywf.mybatis.mapper.IEmployMapper.findByDeptId"
                    fetchType="lazy"
        />
    </resultMap>

    <!-- 获取所有部门信息，延迟加载部门底下的员工新 -->
    <select id="lazyFindAll" resultMap="lazyDeptResultMap">
        SELECT d.id as d_id, d.dept_name FROM dept d
    </select>
</mapper>
```



#### 多对一实现延迟加载

> 场景：打印出所有员工名称，此时不想加载部门信息

- **方式一：配置延迟加载的全局开关,参考配置**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD SQL Map Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- 延迟加载的全局开关 -->
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>
</configuration>
```

- **EmloyMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IEmployMapper">
    <resultMap id="lazyEmployResultMap" type="Employ">
        <result property="id" column="e_id" />
        <result property="employName" column="employ_name" />
        <association property="dept" column="dept_id"
            select="com.ywf.mybatis.mapper.IDeptMapper.getById"
        />
    </resultMap>

    <!--获取所有员工信息， 懒加载部门信息 -->
    <select id="lazyFindAll" resultMap="lazyEmployResultMap">
        SELECT * FROM employ e
    </select>
</mapper>
```

- **IDeptMapper.java**

```java
package com.ywf.mybatis.mapper;

import com.ywf.mybatis.entity.Dept;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import java.util.List;
import java.util.Map;

/**
 * @Author:ywf
 */
@Mapper
public interface IDeptMapper {
    /**
     * 根据部门id获取部门详情
     * @param id
     * @return
     */
    Dept getById(@Param("id") int id);
}
```

- **DeptMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IDeptMapper">
    <!-- 根据ID获取部门信息 -->
    <select id="getById" resultType="Dept">
        SELECT * FROM dept WHERE id = #{id}
    </select>
</mapper>
```

- **调用**

```java
/**
 * 多对一 延迟加载
 */
@Test
void lazyMany2One() {
    List<Employ> employList = employService.lazyFindAll();
    for (Employ employ : employList) {
        log.info(employ.getEmployName());
    }
}
```

**方式二：fetchType="lazy"**

- **EmployMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IEmployMapper">
    <resultMap id="lazyEmployResultMap" type="Employ">
        <result property="id" column="e_id" />
        <result property="employName" column="employ_name" />
        <association property="dept" column="dept_id" fetchType="lazy"
            select="com.ywf.mybatis.mapper.IDeptMapper.getById"
        />
    </resultMap>

    <!--获取所有员工信息， 懒加载部门信息 -->
    <select id="lazyFindAll" resultMap="lazyEmployResultMap">
        SELECT * FROM employ e
    </select>
</mapper>
```



#### 多对多实现延迟加载

> 场景：打印出所有学生名称，此时不想关联课程信息

**方式一：配置延迟加载的全局开关,参考配置**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD SQL Map Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- 延迟加载的全局开关 -->
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>
</configuration>
```

- **StudentMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IStudentMapper">
    <resultMap type="Student" id="lazyStudentResultMap">
        <result property="id" column="student_id"/>
        <result property="studentName" column="student_name"/>
        <collection property="courseList" ofType="Course" column="course_id"
            select="com.ywf.mybatis.mapper.ICourseMapper.getById"
        />
    </resultMap>
    <!-- 获取所有学生信息，懒加载学生学习的课程信息 -->
    <select id="lazyFindAll" resultMap="lazyStudentResultMap">
        SELECT s.*, sc.course_id FROM student s, student_to_course sc
    </select>
</mapper>
```

- **ICourseMapper.java**

```java
package com.ywf.mybatis.mapper;

import com.ywf.mybatis.entity.Course;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

/**
 * @Author:ywf
 */
@Mapper
public interface ICourseMapper {

    /**
     * 根据ID获取课程详情
     * @param id 课程ID
     * @return
     */
    Course getById(@Param("id") int id);
}
```

- **CourseMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.ICourseMapper">
    <select id="getById" resultType="Course">
        SELECT * FROM course WHERE id = #{id}
    </select>
</mapper>
```

- **调用**

```java
/**
  * 多对多 延迟加载
  */
@Test
void lazyMany2Many() {
    List<Student> studentList = studentService.lazyFindAll();
    for (Student student : studentList) {
        log.info(student.getStudentName());
    }
}
```



## mybatis 分页



## mybatis PageHelper



## mybatis 一级缓存



## mybatis 二级缓存



## mybatis c3p0连接池



## mybatis 查询总数



## mybatis 逆向工程