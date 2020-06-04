<!-- TOC -->

- [mybatis动态SQL](#mybatis动态sql)
    - [mybatis if](#mybatis-if)
    - [mybatis where](#mybatis-where)
    - [mybatis choose](#mybatis-choose)
    - [mybatis bind](#mybatis-bind)
    - [mybatis foreach](#mybatis-foreach)
        - [单传参是array类型](#单传参是array类型)
        - [单传参是List类型](#单传参是list类型)
        - [单传参是Map类型](#单传参是map类型)
        - [多参数（此情况一定用Map）](#多参数此情况一定用map)

<!-- /TOC -->
# mybatis动态SQL

## mybatis if

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IDeptMapper">    
    <select id="findList" resultType="Dept">
        SELECT * FROM DEPT
        <if test="deptName != null">
            where dept_name like concat('%', #{deptName}, '%')
        </if>
    </select>
</mapper>
```



## mybatis where

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IDeptMapper">
    <select id="findList" resultType="Dept">
        SELECT * FROM DEPT
        <where>
            <if test="deptName != null">
                and dept_name like concat('%', #{deptName}, '%')
            </if>
        </where>
    </select>
</mapper>
```



## mybatis choose

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IDeptMapper">
    <select id="findList" resultType="Dept">
        SELECT * FROM DEPT
        <where>
            <choose>
                <when test="deptName != null">
                    and dept_name like concat('%', #{deptName}, '%')
                </when>
            </choose>
        </where>
    </select>
</mapper>
```



## mybatis bind

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IDeptMapper">
    <select id="findList" resultType="Dept">
        SELECT * FROM DEPT
        <where>
            <choose>
                <when test="deptName != null">
                    <bind name="deptNameLike" value="'%' + deptName + '%'"></bind>
                    and dept_name like #{deptNameLike}
                </when>
            </choose>
        </where>
    </select>
</mapper>
```



## mybatis foreach

foreach 用于迭代传入过来的参数。 
它的属性介绍分别是

- collection：表示传入过来的参数的数据类型。该参数为必选。要做 foreach 的对象，作为入参时，List 对象默认用 list 代替作为键，数组对象有 array 代替作为键，Map 对象没有默认的键。当然在作为入参时可以使用 @Param(“keyName”) 来设置键，设置 keyName 后，list,array 将会失效。 除了入参这种情况外，还有一种作为参数对象的某个字段的时候。举个例子： 

  如果 User 有属性 List ids。入参是 User 对象，那么这个 collection = “ids” 如果 User 有属性 Ids ids;其中 Ids 是个对象，Ids 有个属性 List id;入参是 User 对象，那么 collection = “ids.id” 

  - 如果传入的是单参数且参数类型是一个 List 的时候，collection 属性值为 list
  - 如果传入的是单参数且参数类型是一个 array 数组的时候，collection 的属性值为 array
  - 如果传入的参数是多个的时候，我们就需要把它们封装成一个 Map 了，当然单参数也可以封装成 map。

- item： 循环体中的具体对象。支持属性的点路径访问，如 item.age,item.info.details。具体说明：在 list 和数组中是其中的对象，在 map 中是 value，该参数为必选。（它是每一个元素进行迭代时的别名）

- index：在 list 和数组中,index 是元素的序号；在 map 中，index 是元素的 key。

- open：表示该语句以什么开始

- close：表示该语句以什么结束

- separator：表示在每次进行迭代之间以什么符号作为分隔符



### 单传参是array类型

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
     * 根据id数组查询部门列表
     * @param idArray
     * @return
     */
    List<Dept> findByForeachArray(@Param("idArray") int[] idArray);
}

```

- **DeptMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IDeptMapper">
    <select id="findByForeachArray" resultType="Dept">
        SELECT * FROM DEPT
        <where>
            id IN
            <foreach collection="idArray" item="ids" open="(" close=")" separator=",">
                #{ids}
            </foreach>
        </where>
    </select>
</mapper>
```

- **调用方式**

```java
int[] idArray = new int[]{1, 2};
List<Dept> deptList = deptService.findByForeach(idArray);
```



### 单传参是List类型

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
     * 根据id List查询部门列表
     * @param idList
     * @return
     */
    List<Dept> findByForeachList(@Param("idList") List idList);
}

```

- **DeptMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IDeptMapper">
    <select id="findByForeachList" resultType="Dept">
        SELECT * FROM DEPT
        <where>
            id IN
            <foreach collection="idList" item="ids" open="(" close=")" separator=",">
                #{ids}
            </foreach>
        </where>
    </select>
</mapper>
```

- **调用方式**

```java
List<Integer> idList = new ArrayList();
idList.add(1);
idList.add(2);
List<Dept> deptList = deptService.findByForeach(idList);
```



### 单传参是Map类型

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
     * 根据id Map查询部门列表
     * @param idMap
     * @return
     */
    List<Dept> findByForeachMap(@Param("idMap") Map idMap);
}
```

- **DeptMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IDeptMapper">
    <select id="findByForeachMap" resultType="Dept">
        SELECT * FROM DEPT
        <where>
            id IN
            <foreach collection="idMap.idArray" item="ids" open="(" close=")" separator=",">
                #{ids}
            </foreach>
        </where>
    </select>
</mapper>
```

- **调用方式**

```java
int[] idArray = new int[]{1, 2};
Map idMap = new HashMap();
idMap.put("idArray", idArray);
List<Dept> deptList = deptService.findByForeach(idMap);
```



### 多参数（此情况一定用Map）

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
     * 根据Ids 和 部门名称查询
     * @param queryMap
     * @return
     */
    List<Dept> findByIdsAndDeptName(@Param("queryMap") Map queryMap);
}

```

- **DeptMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.ywf.mybatis.mapper.IDeptMapper">
    <select id="findByIdsAndDeptName" resultType="Dept">
        SELECT * FROM DEPT
        <where>
            <if test="queryMap.idArray != null">
                and id IN
                <foreach collection="queryMap.idArray" item="ids" open="(" close=")" separator=",">
                    #{ids}
                </foreach>
            </if>
            <if test="queryMap.deptName != null">
                and dept_name like concat('%', #{queryMap.deptName}, '%');
            </if>
        </where>
    </select>
</mapper>
```



- **调用方式**

```java
int[] idArray = new int[]{1, 2};
Map queryMap = new HashMap();
queryMap.put("idArray", idArray);
queryMap.put("deptName", "研发");
List<Dept> deptList = deptService.findByIdsAndDeptName(queryMap);
```