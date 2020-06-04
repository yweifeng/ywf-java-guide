<!-- TOC -->

- [neo4j概念和语法](#neo4j概念和语法)
        - [Neo4j的特点](#neo4j的特点)
        - [Neo4j的优点](#neo4j的优点)
        - [Neo4j的缺点或限制](#neo4j的缺点或限制)
        - [Neo4j属性图数据模型](#neo4j属性图数据模型)
- [Neo4j - CQL简介](#neo4j---cql简介)
        - [Neo4j CQL命令/条款](#neo4j-cql命令条款)
        - [Neo4j CQL 函数](#neo4j-cql-函数)
        - [Neo4j CQL数据类型](#neo4j-cql数据类型)
- [CREATE命令](#create命令)
        - [CQL创建一个没有属性的节点](#cql创建一个没有属性的节点)
        - [CQL创建具有属性的节点](#cql创建具有属性的节点)
        - [CREATE命令语法](#create命令语法)
- [MATCH命令](#match命令)
        - [MATCH命令语法](#match命令语法)
- [RETURN子句](#return子句)
        - [RETURN命令语法](#return命令语法)
- [MATCH & RETURN](#match--return)
        - [MATCH RETURN命令语法](#match-return命令语法)
        - [MATCH命令语法](#match命令语法-1)
        - [RETURN命令语法](#return命令语法-1)
- [CREATE+MATCH+RETURN命令](#creatematchreturn命令)
        - [创建客户节点](#创建客户节点)
        - [创建CreditCard节点](#创建creditcard节点)
        - [观察节点](#观察节点)
- [关系基础](#关系基础)
- [CREATE创建标签](#create创建标签)
        - [Neo4j CQL创建节点标签](#neo4j-cql创建节点标签)
        - [单个标签到节点](#单个标签到节点)
        - [多个标签到节点](#多个标签到节点)
        - [单个标签到关系](#单个标签到关系)
        - [语法说明](#语法说明)
- [WHERE子句](#where子句)
        - [简单WHERE子句语法](#简单where子句语法)
        - [复杂WHERE子句语法](#复杂where子句语法)
        - [Neo4j CQL中的布尔运算符](#neo4j-cql中的布尔运算符)
        - [Neo4j CQL中的比较运算符](#neo4j-cql中的比较运算符)
        - [使用WHERE子句创建关系](#使用where子句创建关系)
- [DELETE删除](#delete删除)
        - [删除节点](#删除节点)
        - [DELETE节点子句语法](#delete节点子句语法)
        - [DELETE节点和关系子句语法](#delete节点和关系子句语法)
- [REMOVE删除](#remove删除)
        - [删除节点/关系的属性](#删除节点关系的属性)
        - [REMOVE属性子句语法](#remove属性子句语法)
        - [删除节点/关系的标签](#删除节点关系的标签)
- [SET子句](#set子句)
        - [SET子句语法](#set子句语法)

<!-- /TOC -->

# neo4j概念和语法

### Neo4j的特点

- SQL就像简单的查询语言Neo4j CQL
- 它遵循属性图数据模型
- 它通过使用Apache Lucence支持索引
- 它支持UNIQUE约束
- 它包含一个用于执行CQL命令的UI：Neo4j数据浏览器
- 它支持完整的ACID（原子性，一致性，隔离性和持久性）规则
- 它采用原生图形库与本地GPE（图形处理引擎）
- 它支持查询的数据导出到JSON和XLS格式
- 它提供了REST API，可以被任何编程语言（如Java，Spring，Scala等）访问
- 它提供了可以通过任何UI MVC框架（如Node JS）访问的Java脚本
- 它支持两种Java API：Cypher API和Native Java API来开发Java应用程序

### Neo4j的优点

- 它很容易表示连接的数据
- 检索/遍历/导航更多的连接数据是非常容易和快速的
- 它非常容易地表示半结构化数据
- Neo4j CQL查询语言命令是人性化的可读格式，非常容易学习
- 使用简单而强大的数据模型
- 它不需要复杂的连接来检索连接的/相关的数据，因为它很容易检索它的相邻节点或关系细节没有连接或索引

### Neo4j的缺点或限制

- AS的Neo4j 2.1.3最新版本，它具有支持节点数，关系和属性的限制。
- 它不支持Sharding。



### Neo4j属性图数据模型

Neo4j图数据库遵循属性图模型来存储和管理其数据。



**属性图模型规则**

- 表示节点，关系和属性中的数据
- 节点和关系都包含属性
- 关系连接节点
- 属性是键值对
- 节点用圆圈表示，关系用方向键表示。
- 关系具有方向：单向和双向。
- 每个关系包含“开始节点”或“从节点”和“到节点”或“结束节点”



在属性图数据模型中，关系应该是定向的。如果我们尝试创建没有方向的关系，那么它将抛出一个错误消息。

在Neo4j中，关系也应该是有方向性的。如果我们尝试创建没有方向的关系，那么Neo4j会抛出一个错误消息，“关系应该是方向性的”。

Neo4j图数据库将其所有数据存储在节点和关系中。我们不需要任何额外的RRBMS数据库或无SQL数据库来存储Neo4j数据库数据。它以图形的形式存储其数据的本机格式。

Neo4j使用本机GPE（图形处理引擎）引擎来使用它的本机图存储格式。

图形数据库数据模型的主要构建块是：

- **节点**
- **关系**
- **属性**

简单的属性图的例子



![img](img/neo4j1.jpg)


这里我们使用圆圈表示节点。 使用箭头的关系。 关系是有方向性的。 我们可以用Properties（键值对）来表示Node的数据。 在这个例子中，我们在Node的Circle中表示了每个Node的Id属性。



# Neo4j - CQL简介

CQL代表Cypher查询语言。 像Oracle数据库具有查询语言SQL，Neo4j具有CQL作为查询语言。



**Neo4j CQL -**

- 它是Neo4j图形数据库的查询语言。
- 它是一种声明性模式匹配语言
- 它遵循SQL语法。
- 它的语法是非常简单且人性化、可读的格式。

**如Oracle SQL -**

- Neo4j CQL 已命令来执行数据库操作。
- Neo4j CQL 支持多个子句像在哪里，顺序等，以非常简单的方式编写非常复杂的查询。
- NNeo4j CQL 支持一些功能，如字符串，Aggregation.In 加入他们，它还支持一些关系功能。

### Neo4j CQL命令/条款

常用的Neo4j CQL命令/条款如下：

| S.No. | CQL命令/条      | 用法                         |
| ----- | --------------- | ---------------------------- |
| 1。   | CREATE 创建     | 创建节点，关系和属性         |
| 2。   | MATCH 匹配      | 检索有关节点，关系和属性数据 |
| 3。   | RETURN 返回     | 返回查询结果                 |
| 4。   | WHERE 哪里      | 提供条件过滤检索数据         |
| 5。   | DELETE 删除     | 删除节点和关系               |
| 6。   | REMOVE 移除     | 删除节点和关系的属性         |
| 7。   | ORDER BY以…排序 | 排序检索数据                 |
| 8。   | SET 组          | 添加或更新标签               |



### Neo4j CQL 函数

以下是常用的Neo4j CQL函数：

| S.No. | 定制列表功能      | 用法                                             |
| ----- | ----------------- | ------------------------------------------------ |
| 1     | String 字符串     | 它们用于使用String字面量。                       |
| 2     | Aggregation 聚合  | 它们用于对CQL查询结果执行一些聚合操作。          |
| 3     | Relationship 关系 | 他们用于获取关系的细节，如startnode，endnode等。 |



### Neo4j CQL数据类型

这些数据类型与Java语言类似。 它们用于定义节点或关系的属性

Neo4j CQL支持以下数据类型：

| S.No. | CQL数据类型 | 用法                            |
| ----- | ----------- | ------------------------------- |
| 1.    | boolean     | 用于表示布尔文字：true，false。 |
| 2.    | byte        | 用于表示8位整数。               |
| 3.    | short       | 用于表示16位整数。              |
| 4.    | int         | 用于表示32位整数。              |
| 5.    | long        | 用于表示64位整数。              |
| 6.    | float       | I用于表示32位浮点数。           |
| 7.    | double      | 用于表示64位浮点数。            |
| 8.    | char        | 用于表示16位字符。              |
| 9.    | String      | 用于表示字符串。                |

# CREATE命令

Neo4j使用CQL“CREATE”命令

- 创建没有属性的节点
- 使用属性创建节点
- 在没有属性的节点之间创建关系
- 使用属性创建节点之间的关系
- 为节点或关系创建单个或多个标签

我们将在本章中讨论如何创建一个没有属性的节点。 对于其他情况，请参考后面的章节。



### CQL创建一个没有属性的节点

Neo4j CQL“CREATE”命令用于创建没有属性的节点。 它只是创建一个没有任何数据的节点。



**CREATE命令语法**

```
CREATE (<node-name>:<label-name>)
```



语法说明

| 语法元素     | 描述                       |
| ------------ | -------------------------- |
| CREATE       | 它是一个Neo4j CQL命令。    |
| <node-name>  | 它是我们要创建的节点名称。 |
| <label-name> | 它是一个节点标签名称       |

注意事项

 

1、Neo4j数据库服务器使用此<node-name>将此节点详细信息存储在Database.As中作为Neo4j DBA或Developer，我们不能使用它来访问节点详细信息。



2、Neo4j数据库服务器创建一个<label-name>作为内部节点名称的别名。作为Neo4j DBA或Developer，我们应该使用此标签名称来访问节点详细信息。



**例如**：

本示例演示如何创建一个简单的“Employee”节点。 按照以下步骤：

**步骤1** - 打开Neo4j数据浏览器

![打开Neo4j数据浏览器](https://atts.w3cschool.cn/attachments/day_161226/201612261037371182.png)

**步骤2** - 在数据浏览器中的美元提示符下键入以下命令。

```
CREATE (emp:Employee)
```

这里emp是一个节点名

Employee是emp节点的标签名称



![execute button](https://atts.w3cschool.cn/attachments/day_161226/201612261037375458.png)



**步骤3** - 单击执行按钮，并在数据浏览器中看到成功消息。



![单击执行按钮](https://atts.w3cschool.cn/attachments/day_161226/201612261037383683.png)



它显示在Neo4j数据库中创建一个标签和一个节点。 它在数据库中创建一个带有标签名“Employee”的节点“emp”。

**例如**：

本示例演示如何创建一个简单的“Dept”节点。 按照以下步骤：



**步骤1** - 打开Neo4j数据浏览器。

**步骤2** - 在数据浏览器中的美元提示符下键入以下命令。

```
CREATE (dept:Dept)
```



这里dept是一个节点名
Dept是dept节点的标签名称





![CREATE (dept:Dept)](https://atts.w3cschool.cn/attachments/day_161226/201612261037388113.png)



**步骤3** - 单击执行按钮，并在数据浏览器中看到成功消息。



![单击执行按钮](https://atts.w3cschool.cn/attachments/day_161226/201612261048102470.png)



它显示在Neo4j数据库中创建一个标签和一个节点。 它在数据库中创建一个标签名为“Dept”的节点“dept”。



### CQL创建具有属性的节点

Neo4j CQL“CREATE”命令用于创建带有属性的节点。 它创建一个具有一些属性（键值对）的节点来存储数据。



### CREATE命令语法 

```
CREATE (
   <node-name>:<label-name>
   { 	
      <Property1-name>:<Property1-Value>
      ........
      <Propertyn-name>:<Propertyn-Value>
   }
)
```



语法说明：

| 语法元素                              | 描述                                            |
| ------------------------------------- | ----------------------------------------------- |
| <node-name>                           | 它是我们将要创建的节点名称。                    |
| <label-name>                          | 它是一个节点标签名称                            |
| <Property1-name>...<Propertyn-name>   | 属性是键值对。 定义将分配给创建节点的属性的名称 |
| <Property1-value>...<Propertyn-value> | 属性是键值对。 定义将分配给创建节点的属性的值   |

**例如**：

此示例演示如何创建具有一些属性（deptno，dname，位置）的Dept节点。 按照下面给出的步骤 - 



**步骤1** - 打开Neo4j数据浏览器。

**步骤2** - 在数据浏览器中的dollar提示符下键入以下命令。

```
CREATE (dept:Dept { deptno:10,dname:"Accounting",location:"Hyderabad" })
```



这里dept是一个节点名
Dept是emp节点的标签名称





![创建dept节点](https://atts.w3cschool.cn/attachments/day_161226/201612261048106018.png)



这里的属性名称是deptno，dname，location

属性值为10，"Accounting","Hyderabad"

正如我们讨论的，属性一个名称 - 值对。

Property = deptno:10

因为deptno是一个整数属性，所以我们没有使用单引号或双引号定义其值10。

由于dname和location是String类型属性，因此我们使用单引号或双引号定义其值10。

**注意 -** 要定义字符串类型属性值，我们需要使用单引号或双引号。



**步骤3** -单击执行按钮，并在数据浏览器中查看成功消息。



![在数据浏览器中查看成功消息](https://atts.w3cschool.cn/attachments/day_161226/201612261048114777.png)



如果你观察到成功的消息，它告诉我们

- 创建一个标签，即“Dept”

- 创建一个节点，即“dept”

- 创建三个属性，即deptno，dname，location

  

例如：****

此示例演示如何创建具有一些属性（id，name，sal，deptno）的Employee节点。 按照下面给出的步骤 - 



**步骤1** -打开Neo4j数据浏览器。



**步骤2** -在数据浏览器中的dollar提示符下键入以下命令。

```
CREATE (emp:Employee{id:123,name:"Lokesh",sal:35000,deptno:10})
```



这里emp是一个节点名
Employee是dept节点的标签名称





![打开Neo4j数据浏览器](https://atts.w3cschool.cn/attachments/day_161226/201612261048119978.png)



**步骤3** - 单击执行按钮，并在数据浏览器中看到成功消息。



![观察成功消息](https://atts.w3cschool.cn/attachments/day_161226/201612261111149150.png)



观察成功消息

添加了1个标签，创建了1个节点，设置了4个属性，返回0行

此命令已创建一个具有4个属性（“id”，“name”，“sal”，“deptno”）的节点“emp”，并分配了一个标签“Employee”。



# MATCH命令

Neo4j CQL MATCH命令用于 - 

- 从数据库获取有关节点和属性的数据
- 从数据库获取有关节点，关系和属性的数据

### MATCH命令语法

```
MATCH 
(
   <node-name>:<label-name>
)
```

语法说明

| 语法元素     | 描述                         |
| ------------ | ---------------------------- |
| <node-name>  | 这是我们要创建一个节点名称。 |
| <label-name> | 这是一个节点的标签名称       |

注意事项

- Neo4j数据库服务器使用此<node-name>将此节点详细信息存储在Database.As中作为Neo4j DBA或Developer，我们不能使用它来访问节点详细信息。
- Neo4j数据库服务器创建一个<label-name>作为内部节点名称的别名。作为Neo4j DBA或Developer，我们应该使用此标签名称来访问节点详细信息。

**注意-**我们不能单独使用MATCH Command从数据库检索数据。 如果我们单独使用它，那么我们将InvalidSyntax错误。

**例如：**

这个例子演示了“如果我们单独使用MATCH命令从数据库检索数据会发生什么”。 按照下面给出的步骤 - 


**步骤1** -打开Neo4j的数据浏览器。

**步骤2** -在数据浏览器的dollar提示符处键入以下命令。

```
MATCH (dept:Dept)
```

这里 -

- dept是节点名称
- Dept是emp节点的标签名称



**![match](https://atts.w3cschool.cn/attachments/day_161226/201612261140299696.png)**



第3步 -单击执行按钮，并在数据浏览器中看到成功消息。

![执行](https://atts.w3cschool.cn/attachments/day_161226/201612261142146536.png)



如果你观察到错误消息，它告诉我们，我们可以使用MATCH命令与RETURN子句或更新子句。



# RETURN子句

Neo4j CQL RETURN子句用于 -

- 检索节点的某些属性
- 检索节点的所有属性
- 检索节点和关联关系的某些属性
- 检索节点和关联关系的所有属性

### RETURN命令语法

```
RETURN 
   <node-name>.<property1-name>,
   ........
   <node-name>.<propertyn-name>
```



语法说明:

| 语法元素                            | 描述                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| <node-name>                         | 它是我们将要创建的节点名称。                                 |
| <Property1-name>...<Propertyn-name> | 属性是键值对。 <Property-name>定义要分配给创建节点的属性的名称 |

**例如：**

本示例演示了“如果我们单独使用RETURN命令从数据库检索数据，会发生什么”。 按照下面给出的步骤 - 

**步骤1** -打开Neo4j的数据浏览器。



**步骤2** -在数据浏览器的dollar提示符处键入以下命令。

```
RETURN dept.deptno
```



这里 -

- dept是节点名称
- deptno是dept节点的属性名称

![dept.deptno](https://atts.w3cschool.cn/attachments/day_161226/201612261403469284.png)



**第3步** -单击执行按钮，并在数据浏览器中看到成功消息。





![执行成功](https://atts.w3cschool.cn/attachments/day_161226/201612261404407960.png)





如果您发现该错误消息，它告诉我们，我们不能单独使用RETURN子句。我们应该既MATCH使用或CREATE命令。

# MATCH & RETURN

在Neo4j CQL中，我们不能单独使用MATCH或RETURN命令，因此我们应该合并这两个命令以从数据库检索数据。

Neo4j使用CQL MATCH + RETURN命令 - 

- 检索节点的某些属性
- 检索节点的所有属性
- 检索节点和关联关系的某些属性
- 检索节点和关联关系的所有属性

### MATCH RETURN命令语法

```
MATCH Command
RETURN Command
```



语法说明：

| 语法元素   | 描述                       |
| ---------- | -------------------------- |
| MATCH命令  | 这是Neo4j CQL MATCH命令。  |
| RETURN命令 | 这是Neo4j CQL RETURN命令。 |



### MATCH命令语法

```
MATCH 
(
   <node-name>:<label-name>
)
```



语法说明：

| 语法元素     | 描述                         |
| ------------ | ---------------------------- |
| <node-name>  | 它是我们将要创建的节点名称。 |
| <label-name> | 它是一个节点标签名称         |



要点 -

- Neo4j数据库服务器使用此<node-name>将此节点详细信息存储在Database.As中作为Neo4j DBA或Developer，我们不能使用它来访问节点详细信息。
- Neo4j数据库服务器创建一个<label-name>作为内部节点名称的别名。作为Neo4j DBA或Developer，我们应该使用此标签名称来访问节点详细信息。

### RETURN命令语法

```
RETURN 
   <node-name>.<property1-name>,
   ...
   <node-name>.<propertyn-name>
```



语法说明：

| 语法元素                            | 描述                                            |
| ----------------------------------- | ----------------------------------------------- |
| <node-name>                         | 它是我们将要创建的节点名称。                    |
| <Property1-name>...<Propertyn-name> | 属性是键值对。 定义将分配给创建节点的属性的名称 |



**例如：**

本示例演示如何从数据库检索Dept节点的一些属性（deptno，dname）数据。



**注-**结点包含3个属性：deptno，dname，location。 然而在这个例子中，我们感兴趣的是只查看两个属性数据。 按照下面给出的步骤 - 



**步骤1** -打开Neo4j的数据浏览器。



**步骤2** -在数据浏览器中的dollar提示符下键入以下命令。

```
MATCH (dept: Dept)
RETURN dept.deptno,dept.dname
```

这里 -

- dept是节点名称
- 这里Dept是一个节点标签名
- deptno是dept节点的属性名称
- dname是dept节点的属性名

**
**



**![img](https://atts.w3cschool.cn/attachments/day_161226/201612261444296573.png)**





**
**

第3步 -单击执行按钮，并在数据浏览器中看到成功消息。





![dept.deptno](https://atts.w3cschool.cn/attachments/day_161226/201612261446297769.png)





如果观察到数据浏览器消息，它将显示有关两个属性的Dept节点的数据：deptno，dname。 它返回Neo4j数据库中可用的两个节点（行）。

**例如：**

本示例演示如何从数据库检索Dept Node的所有属性（deptno，dname，location）数据。

**
**

**注-**结点包含3个属性：deptno，dname，location。 按照下面给出的步骤 - 



**步骤1** -打开Neo4j数据浏览器。





![打开Neo4j数据浏览器](https://atts.w3cschool.cn/attachments/day_161226/201612261451195962.png)





它是Neo4j数据浏览器主页



**步骤2** -在数据浏览器中的dollar提示符下键入以下命令。

```
MATCH (dept: Dept)
RETURN dept.deptno,dept.dname,dept.location
```



这里 -

- dept是节点名称
- 这里Dept是一个节点标签名
- deptno是dept节点的属性名称
- dname是dept节点的属性名
- location是dept节点的属性名

**
**



**![MATCH (dept: Dept)](https://atts.w3cschool.cn/attachments/day_161226/201612261504057886.png)**





**
**

步骤3 -单击执行按钮，并在数据浏览器中看到成功消息。





![执行](https://atts.w3cschool.cn/attachments/day_161226/201612261506353497.png)





它返回Dept节点的所有属性数据。 由于数据库包含两个具有相同名称“dept：Dept”的节点，因此在执行此命令时，它将返回这两行。

**例如：**

此示例演示如何从数据库检索Dept节点的数据，而无需指定其属性。



**注-**结点包含3个属性：deptno，dname，location。 按照下面给出的步骤 - 



**步骤1** -打开Neo4j数据浏览器。

**步骤2** -在数据浏览器中的dollar提示符下键入以下命令。

```
MATCH (dept: Dept)
RETURN dept
```

这里dept是一个节点名

这里Dept是一个节点标签名

**
**



**![MATCH (dept: Dept) RETURN dept](https://atts.w3cschool.cn/attachments/day_161226/201612261511277816.png)**





**
**

步骤3 -单击执行按钮，并在数据浏览器中看到成功消息。





![两个圆圈](https://atts.w3cschool.cn/attachments/day_161226/201612261516161810.png)

在这里我们可以观察到两个圆圈与UI模式的一些ids



ID = 3215显示一个节点

ID = 25显示了另一个节点



当我们执行“RETURN”子句而不指定任何属性列表，如“RETURN dept”
默认情况下，它在UI模式下显示结果。



**步骤4** -单击网格视图按钮以网格格式查看两行。





![单击网格视图](https://atts.w3cschool.cn/attachments/day_161226/201612261531038545.png)



# CREATE+MATCH+RETURN命令

在Neo4j CQL中，我们不能单独使用MATCH或RETURN命令，因此我们应该结合这两个命令从数据库检索数据。

**例如：**

本示例演示如何使用属性和这两个节点之间的关系创建两个节点。

**注-**我们将创建两个节点：客id，name，dob属性。

- 客户节点包含：ID，姓名，出生日期属性
- CreditCard节点包含：id，number，cvv，expiredate属性
- 客户与信用卡关系：DO_SHOPPING_WITH
- CreditCard到客户关系：ASSOCIATED_WITH

我们将在以下步骤中处理此示例： -

- 创建客户节点
- 创建CreditCard节点
- 观察先前创建的两个节点：Customer和CreditCard
- 创建客户和CreditCard节点之间的关系
- 查看新创建的关系详细信息
- 详细查看每个节点和关系属性

**注-**我们将在本章讨论前三个步骤。我们将在以后的章节中讨论其余的步骤



### 创建客户节点

**步骤1** -打开Neo4j数据浏览器。



**![打开Neo4j数据浏览器](https://atts.w3cschool.cn/attachments/day_161226/201612261553356284.png)

**步骤2** -在数据浏览器中的dollar提示符下键入以下命令。

```
CREATE (e:Customer{id:"1001",name:"Abc",dob:"01/10/1982"})
```

这里 -

- e是节点名称
- 在这里Customer是节点标签名称
- id，name和dob是Customer节点的属性名称



**![CREATE](https://atts.w3cschool.cn/attachments/day_161226/201612261633151049.png)**



**步骤3** -单击执行按钮创建具有3个属性的客户节点。





![创建具有3个属性的客户节点](https://atts.w3cschool.cn/attachments/day_161226/201612261634069257.png)

如果您观察到数据浏览器消息，它显示在Neo4j数据库中创建一个带有3个属性的节点。



### 创建CreditCard节点

**步骤1** -打开Neo4j数据浏览器。



**步骤2** -在数据浏览器中的美元提示符下键入以下命令。

```
CREATE (cc:CreditCard{id:"5001",number:"1234567890",cvv:"888",expiredate:"20/17"})
```

这里c是一个节点名

这里CreditCard是节点标签名称

id，number，cvv和expiredate是CreditCard节点的属性名称





**![创建CreditCard节点](https://atts.w3cschool.cn/attachments/day_161226/201612261636298508.png)**

**步骤3** -单击执行按钮创建具有4个属性的CreditCard节点。







![创建具有4个属性的CreditCard节点](https://atts.w3cschool.cn/attachments/day_161226/201612261639171912.png)

如果您观察到数据浏览器消息，它显示在Neo4j数据库中创建一个带有4个属性的节点。





### 观察节点



现在我们创建了两个节点：Customer和CreditCard



我们需要使用带有RETURN子句的Neo4j CQL MATCH命令查看这两个节点的详细信息





**查看客户节点详细信息**

**步骤1** -打开Neo4j数据浏览器

**步骤2** -在数据浏览器中的美元提示符下键入以下命令。

```
MATCH (e:Customer)
RETURN e.id,e.name,e.dob
```

这里e是节点名

在这里Customer是节点标签名称

id，name和dob是Customer节点的属性名称





**![MATCH (e:Customer)](https://www.tutorialspoint.com/neo4j/images/create_match_return_example5.png)**



步骤3 -单击Execute按钮运行此命令。





![Execute](https://www.tutorialspoint.com/neo4j/images/create_match_return_example6.png)

如果您观察到数据浏览器消息，则显示在Neo4j数据库中创建具有3个属性的客户节点。





**查看CreditCard节点详细信息**

**步骤1** -打开Neo4j数据浏览器

**步骤2** -在数据浏览器中的dollar提示符下键入以下命令。

```
MATCH (cc:CreditCard)
RETURN cc.id,cc.number,cc.cvv,cc.expiredate
```

这里cc是一个节点名

这里CreditCard是节点标签名称

id，number，cvv，expiredate是CreditCard节点的属性名称





**![查看CreditCard节点详细信息](https://atts.w3cschool.cn/attachments/day_161226/201612261645552452.png)**



步骤3 -单击Execute按钮运行此命令。





![运行此命令](https://www.tutorialspoint.com/neo4j/images/create_match_return_example8.png)



如果您观察到数据浏览器消息，则会显示在Neo4j数据库中创建了4个属性的CreditCard节点。



# 关系基础

Neo4j图数据库遵循属性图模型来存储和管理其数据。

根据属性图模型，关系应该是定向的。 否则，Neo4j将抛出一个错误消息。

基于方向性，Neo4j关系被分为两种主要类型。

- 单向关系
- 双向关系

在以下场景中，我们可以使用Neo4j CQL CREATE命令来创建两个节点之间的关系。 这些情况适用于Uni和双向关系。

- 在两个现有节点之间创建无属性的关系
- 在两个现有节点之间创建与属性的关系
- 在两个新节点之间创建无属性的关系
- 在两个新节点之间创建与属性的关系
- 在具有WHERE子句的两个退出节点之间创建/不使用属性的关系

**注意 -**

我们将创建客户和CreditCard之间的关系，如下所示：







![创建客户和CreditCard之间的关系](https://www.tutorialspoint.com/neo4j/images/create_relationship_example1.png)





在上一章中，我们已经创建了Customer和CreditCard节点。 现在我们将看到如何创建它们之间的关系

此图描述了客户与CreditCard之间的关系



客户→信用卡

这里的关系是箭头标记（→）

由于Neo4j CQL语法是以人类可读的格式。 Neo4j CQL也使用类似的箭头标记来创建两个节点之间的关系。

每个关系（→）包含两个节点

- 从节点
- 到节点

从上图中，Customer节点是“From Node”，CreditCard Node是“To Node”这种关系。

对于节点，它们是两种关系

- 外向关系
- 传入关系



从上图中，关系是到客户节点的“外向关系”，并且相同的关系是到信用卡节点的“到达关系”。



考虑下面的图。 这里我们创建了从“CreditCard”节点到“客户”节点的关系。







![从“CreditCard”节点到“客户”节点的关系](https://atts.w3cschool.cn/attachments/day_161226/201612261732166330.png)





从上面的图中，关系是“出局关系”到“信用卡”节点，并且相同的关系是“到达关系”到“客户”节点。



考虑下面的图。 我们在“CreditCard”和“Customer”节点之间创建了两个关系：一个从“CreditCard”到“Customer”。 另一个从“客户”到“信用卡”。 这意味着它是双向关系。



![双向关系](https://atts.w3cschool.cn/attachments/day_161226/201612261733553967.png)





# CREATE创建标签

### Neo4j CQL创建节点标签

Label是Neo4j数据库中的节点或关系的名称或标识符。

我们可以将此标签名称称为关系为“关系类型”。

我们可以使用CQL CREATE命令为节点或关系创建单个标签，并为节点创建多个标签。 这意味着Neo4j仅支持两个节点之间的单个关系类型。

我们可以在UI模式和网格模式下在CQL数据浏览器中观察此节点或关系的标签名称。 并且我们引用它执行CQL命令。

到目前为止，我们只创建了一个节点或关系的标签，但我们没有讨论它的语法。



使用Neo4j CQL CREATE命令

- 为节点创建单个标签
- 为节点创建多个标签
- 为关系创建单个标签

我们将在本章中讨论如何创建一个节点的单个标签或多个标签。 我们将在下一章讨论如何为关系创建一个单独的标签。



### 单个标签到节点

语法：

```
CREATE (<node-name>:<label-name>)
```

| S.No. | 语法元素               | 描述                      |
| ----- | ---------------------- | ------------------------- |
| 1     | CREATE 创建            | 它是一个Neo4j CQL关键字。 |
| 2     | <node-name> <节点名称> | 它是一个节点的名称。      |
| 3     | <label-name><标签名称> | 这是一个节点的标签名称。  |

**
**

**注意 -**

- 我们应该使用colon（:)运算符来分隔节点名和标签名。
- Neo4j数据库服务器使用此名称将此节点详细信息存储在Database.As Neo4j DBA或Developer中，我们不能使用它来访问节点详细信息
- Neo4j数据库服务器创建一个标签名称作为内部节点名称的别名。作为Neo4j DBA或开发人员，我们应该使用此标签名称来访问节点详细信息。

**例如：**

本示例演示如何为“GooglePlusProfile”节点创建单个标签。



**步骤1** -打开Neo4j数据浏览器

**
**



**![打开Neo4j数据浏览器](https://atts.w3cschool.cn/attachments/day_161226/201612261742523630.png)**





**
**

步骤2 -在数据浏览器上键入以下命令

```
CREATE (google1:GooglePlusProfile)
```

这里google1是一个节点名

GooglePlusProfile是google1node的标签名称





**![CREATE (google1:GooglePlusProfile)](https://atts.w3cschool.cn/attachments/day_161226/201612261745096081.png)**





**
**

步骤3 -点击“执行”按钮并观察结果。





![一个标签和一个节点在Neo4j的数据库中创建](https://atts.w3cschool.cn/attachments/day_161226/201612261745489559.png)

我们可以观察到在Neo4j数据库中创建了一个标签和一个节点。





### 多个标签到节点

语法：

```
CREATE (<node-name>:<label-name1>:<label-name2>.....:<label-namen>)
```

| S.No. | 语法元素                                         | 描述                           |
| ----- | ------------------------------------------------ | ------------------------------ |
| 1。   | CREATE 创建                                      | 这是一个Neo4j CQL关键字。      |
| 2。   | <node-name> <节点名称>                           | 它是一个节点的名称。           |
| 3。   | <label-name1>,<label-name2> <标签名1>，<标签名2> | 它是一个节点的标签名称的列表。 |

**
**

**注意 -**

- 我们应该使用colon（:)运算符来分隔节点名和标签名。
- 我们应该使用colon（:)运算符将一个标签名称分隔到另一个标签名称。

**例如：**

本示例演示如何为“Cinema”节点创建多个标签名称。

我们的客户提供的多个标签名称：Cinema,Film,Movie,Picture。



**步骤1** -打开Neo4j数据浏览器



**步骤2** -在数据浏览器上键入以下命令

```
CREATE (m:Movie:Cinema:Film:Picture)
```

这里m是一个节点名

Movie, Cinema, Film, Picture是m节点的多个标签名称





**![为“Cinema”节点创建多个标签名称](https://atts.w3cschool.cn/attachments/day_161226/201612261749594325.png)**





**
**

步骤3 -点击“执行”按钮并观察结果。





![结果](https://atts.w3cschool.cn/attachments/day_161226/201612261750382910.png)

这里我们可以观察到在Neo4j数据库中创建了四个标签和一个节点。





### 单个标签到关系 

语法：

```
CREATE (<node1-name>:<label1-name>)-
	[(<relationship-name>:<relationship-label-name>)]
	->(<node2-name>:<label2-name>)
```



### 语法说明

| S.No. | 语法元素                                 | 描述                      |
| ----- | ---------------------------------------- | ------------------------- |
| 1     | CREATE 创建                              | 它是一个Neo4J CQL关键字。 |
| 2     | <node1-name> <节点1名>                   | 它是From节点的名称。      |
| 3     | <node2-name> <节点2名>                   | 它是To节点的名称。        |
| 4     | <label1-name> <LABEL1名称>               | 它是From节点的标签名称    |
| 5     | <label1-name> <LABEL1名称>               | 它是To节点的标签名称。    |
| 6     | <relationship-name> <关系名称>           | 它是一个关系的名称。      |
| 7     | <relationship-label-name> <相关标签名称> | 它是一个关系的标签名称。  |

**
**

**注意 -**

- 我我们应该使用colon（:)运算符来分隔节点名和标签名。
- 我们应该使用colon（:)运算符来分隔关系名称和关系标签名称。
- 我们应该使用colon（:)运算符将一个标签名称分隔到另一个标签名称。
- Neo4J数据库服务器使用此名称将此节点详细信息存储在Database.As中作为Neo4J DBA或开发人员，我们不能使用它来访问节点详细信息。
- Neo4J Database Server创建一个标签名称作为内部节点名称的别名。作为Neo4J DBA或Developer，我们应该使用此标签名称来访问节点详细信息。

**例如：**

**本示例演示如何为关系创建标签**

**
**

**步骤1** -打开Neo4J数据浏览器



**步骤2** -在数据浏览器上键入以下命令

```
CREATE (p1:Profile1)-[r1:LIKES]->(p2:Profile2)
```

这里p1和profile1是节点名称和节点标签名称“From Node”

p2和Profile2是“To Node”的节点名称和节点标签名称

r1是关系名称

LIKES是一个关系标签名称





**![为关系创建标签](https://atts.w3cschool.cn/attachments/day_161226/201612261757289046.png)**





**
**

**步骤3** -点击“执行”按钮并观察结果。





![两个节点，两个标签和一个关系被添加到Neo4J数据库](https://atts.w3cschool.cn/attachments/day_161226/201612261758046637.png)

这里我们可以观察到两个节点，两个标签和一个关系被添加到Neo4J数据库。



# WHERE子句

像SQL一样，Neo4j CQL在CQL MATCH命令中提供了WHERE子句来过滤MATCH查询的结果。



### 简单WHERE子句语法

```
WHERE <condition>
```



### 复杂WHERE子句语法

```
WHERE <condition> <boolean-operator> <condition>
```

我们可以使用布尔运算符在同一命令上放置多个条件。 请参考下一节，了解Neo4j CQL中可用的布尔运算符。


**语法：**

```
<property-name> <comparison-operator> <value>
```



语法说明：

| S.No. | 语法元素                           | 描述                                                         |
| ----- | ---------------------------------- | ------------------------------------------------------------ |
| 1     | WHERE                              | 它是一个Neo4j CQL关键字。                                    |
| 2     | <property-name> <属性名称>         | 它是节点或关系的属性名称。                                   |
| 3     | <comparison-operator> <比较运算符> | 它是Neo4j CQL比较运算符之一。请参考下一节查看Neo4j CQL中可用的比较运算符。 |
| 4     | <value> <值>                       | 它是一个字面值，如数字文字，字符串文字等。                   |



### Neo4j CQL中的布尔运算符

Neo4j支持以下布尔运算符在Neo4j CQL WHERE子句中使用以支持多个条件。

| S.No. | 布尔运算符 | 描述                                   |
| ----- | ---------- | -------------------------------------- |
| 1     | AND        | 它是一个支持AND操作的Neo4j CQL关键字。 |
| 2     | OR         | 它是一个Neo4j CQL关键字来支持OR操作。  |
| 3     | NOT        | 它是一个Neo4j CQL关键字支持NOT操作。   |
| 4     | XOR        | 它是一个支持XOR操作的Neo4j CQL关键字。 |



### Neo4j CQL中的比较运算符

Neo4j 支持以下的比较运算符，在 Neo4j CQL WHERE 子句中使用来支持条件。

S.No.布尔运算符描述

1.=它是Neo4j CQL“等于”运算符。

2.<>它是一个Neo4j CQL“不等于”运算符。

3.<它是一个Neo4j CQL“小于”运算符。

4.>它是一个Neo4j CQL“大于”运算符。

5.<=它是一个Neo4j CQL“小于或等于”运算符。

6.>=它是一个Neo4j CQL“大于或等于”运算符。 

**例如：**

此示例演示如何在MATCH Command中使用CQL WHERE子句根据员工名称检索员工详细信息。

**
**

**步骤1** -打开Neo4j数据浏览器





![在MATCH Command中使用CQL WHERE子句](https://atts.w3cschool.cn/attachments/day_161226/201612261818306549.png)

这是Neo4j数据浏览器主页



**
**

**步骤2** -在数据浏览器上键入以下命令

```
MATCH (emp:Employee)
RETURN emp.empid,emp.name,emp.salary,emp.deptno
```





![MATCH (emp:Employee) RETURN emp.empid,emp.name,emp.salary,emp.deptno](https://atts.w3cschool.cn/attachments/day_161226/201612261819349206.png)





**步骤3** -点击“执行”按钮并观察结果。





![返回4个员工节点详细信息](https://atts.w3cschool.cn/attachments/day_161226/201612261820099498.png)

如果我们观察结果，它返回4个员工节点详细信息。



**
**

**步骤4** -在数据浏览器上键入以下命令

```
MATCH (emp:Employee) 
WHERE emp.name = 'Abc'
RETURN emp
```

**
**



**![MATCH (emp:Employee)  WHERE emp.name = 'Abc' RETURN emp](https://atts.w3cschool.cn/attachments/day_161226/201612261821005897.png)**





**
**

**步骤5** -点击“执行”按钮并观察结果。





![返回一个名为“Abc”的员工详细信息](https://atts.w3cschool.cn/attachments/day_161226/201612261821315567.png)

使用“网格视图”查看节点详细信息。如果我们观察结果，它只返回一个名为“Abc”的员工详细信息。





**例如：**

此示例演示如何在MATCH Command中的CQL WHERE子句中使用多个条件与布尔运算符，以根据员工名称检索员工详细信息。



**步骤1** -打开Neo4j数据浏览器

**
**



**![根据员工名称检索员工详细信息](https://atts.w3cschool.cn/attachments/day_161226/201612261822324844.png)**





**
**

**步骤2** -在数据浏览器上键入以下命令

```
MATCH (emp:Employee)
RETURN emp.empid,emp.name,emp.salary,emp.deptno
```





![MATCH (emp:Employee) RETURN emp.empid,emp.name,emp.salary,emp.deptno](https://atts.w3cschool.cn/attachments/day_161226/201612261823191030.png)





**步骤3** -点击“执行”按钮并观察结果。





![它返回4个员工节点详细信息](https://atts.w3cschool.cn/attachments/day_161226/201612261823404035.png)

如果我们观察结果，它返回4个员工节点详细信息。



**
**

**步骤4** -在数据浏览器上键入以下命令

```
MATCH (emp:Employee) 
WHERE emp.name = 'Abc' OR emp.name = 'Xyz'
RETURN emp
```

**
**



**![MATCH (emp:Employee)  WHERE emp.name = 'Abc' OR emp.name = 'Xyz' RETURN emp](https://atts.w3cschool.cn/attachments/day_161226/201612261824313749.png)**

**
**



**步骤5**-点击“执行”按钮并观察结果。





![返回两个名为“Abc”或“Xyz”的员工详细信息。](https://atts.w3cschool.cn/attachments/day_161226/201612261825172294.png)

使用“网格视图”查看节点详细信息。如果我们观察到结果，它只返回两个名为“Abc”或“Xyz”的员工详细信息。





### 使用WHERE子句创建关系

在Neo4J CQL中，我们可以以不同的方式创建拖曳节点之间的关系。

- 创建两个现有节点之间的关系
- 一次创建两个节点和它们之间的关系
- 使用WHERE子句创建两个现有节点之间的关系

我们已经讨论了前两章中的前两种方法。 现在我们将在本章中讨论“使用WHERE子句创建两个现有节点之间的关系”。

语法

```
MATCH (<node1-label-name>:<node1-name>),(<node2-label-name>:<node2-name>) 
WHERE <condition>
CREATE (<node1-label-name>)-[<relationship-label-name>:<relationship-name>
       {<relationship-properties>}]->(<node2-label-name>) 
```



语法说明：

| S.No. | 语法元素                  | 描述                                                         |
| ----- | ------------------------- | ------------------------------------------------------------ |
| 1     | MATCH,WHERE,CREATE        | 他们是Neo4J CQL关键字。                                      |
| 2     | <node1-label-name>        | 它是一个用于创建关系的节点一标签名称。                       |
| 3     | <node1-name>              | 它是一个用于创建关系的节点名称。                             |
| 4     | <node2-label-name>        | 它是一个用于创建关系的节点一标签名称。                       |
| 5     | <node2-name>              | 它是一个用于创建关系的节点名称。                             |
| 6     | <condition>               | 它是一个Neo4J CQL WHERE子句条件。 它可以是简单的或复杂的。   |
| 7     | <relationship-label-name> | 这是新创建的节点一和节点二之间的关系的标签名称。             |
| 8     | <relationship-name>       | 这是新创建的节点1和节点2之间的关系的名称。                   |
| 9     | <relationship-properties> | 这是一个新创建节点一和节点二之间关系的属性列表（键 - 值对）。 |

**例如：**

此示例演示如何使用WHERE子句创建两个现有节点之间的关系。



**步骤1** -打开Neo4J数据浏览器



**步骤2** -在数据浏览器上键入以下命令，以验证我们的Neo4J数据库中是否存在所需的客户节点。

```
MATCH (cust:Customer)
RETURN cust.id,cust.name,cust.dob
```





![使用WHERE子句创建两个现有节点之间的关系](https://atts.w3cschool.cn/attachments/day_161226/201612261829584973.png)





**步骤3** -点击“执行”按钮并观察结果。





![客户节点在我们的Neo4J数据库中可用](https://atts.w3cschool.cn/attachments/day_161226/201612261830297705.png)

如果我们观察结果，它表明我们所需的客户节点在我们的Neo4J数据库中可用。



**
**

**步骤4** -在数据浏览器上键入以下命令，验证我们的Neo4J数据库中是否存在所需的CreditCard节点。

```
MATCH (cc:CreditCard)
RETURN cc.id,cc.number,cc.expiredate,cc.cvv
```

**
**



**![验证我们的Neo4J数据库中是否存在所需的CreditCard节点。](https://atts.w3cschool.cn/attachments/day_161226/201612261831316404.png)**

**
**



**步骤5** -点击“执行”按钮并观察结果。





![CreditCard节点在我们的Neo4J数据库中可用](https://atts.w3cschool.cn/attachments/day_161226/201612261832054866.png)

如果我们观察结果，它表明我们所需的CreditCard节点在我们的Neo4J数据库中可用。



**
**

**步骤6** -在数据浏览器上键入以下命令以创建客户和CreditCard节点之间的关系。

```
MATCH (cust:Customer),(cc:CreditCard) 
WHERE cust.id = "1001" AND cc.id= "5001" 
CREATE (cust)-[r:DO_SHOPPING_WITH{shopdate:"12/12/2014",price:55000}]->(cc) 
RETURN r
```



**![创建客户和CreditCard节点之间的关系](https://www.tutorialspoint.com/neo4j/images/relationship_with_where_clause5.png)**





**
**

步骤7 -点击“执行”按钮并观察结果。





![img](https://atts.w3cschool.cn/attachments/day_161226/201612261833495022.png)

单击关系并在单独的窗口中观察其属性







![在两个现有节点之间创建了一个NEW关系](https://atts.w3cschool.cn/attachments/day_161226/201612261834131221.png)

现在我们通过使用Neo4J CQL WHERE子句在两个现有节点之间创建了一个NEW关系



# DELETE删除

Neo4j使用CQL DELETE子句

- 删除节点。
- 删除节点及相关节点和关系。

我们将在本章中讨论如何删除一个节点。 我们将在下一章讨论如何删除节点和相关的节点和关系。



### 删除节点

通过使用此命令，我们可以从数据库永久删除节点及其关联的属性。



### DELETE节点子句语法

```
DELETE <node-name-list>
```

| S.No. | 语法元素         | 描述                                     |
| ----- | ---------------- | ---------------------------------------- |
| 1.    | DELETE           | 它是一个Neo4j CQL关键字。                |
| 2.    | <node-name-list> | 它是一个要从数据库中删除的节点名称列表。 |



**注意 -**

我们应该使用逗号（，）运算符来分隔节点名。



例如：

此示例演示如何从数据库中永久删除节点。



**步骤1** - 打开Neo4j数据浏览器。



**步骤2** - 在数据浏览器上键入以下命令

```
MATCH (e: 'Employee') RETURN e 
```

**
**

**注意 -**

MATCH (e: 'Employee') RETURN e

MATCH (e: "Employee") RETURN e

MATCH (e: Employee) RETURN e



所有三个命令都相同，我们可以选择这些命令中的任何一个。



![MATCH](https://atts.w3cschool.cn/attachments/day_161227/201612270916084585.png)

**
**

**步骤3** - 点击“执行”按钮并观察结果。



![Execute](https://atts.w3cschool.cn/attachments/day_161227/201612270916088550.png)



这里我们可以观察到在数据库中有一个节点可用“Employee”名称。



**步骤4** - 在数据浏览器上键入以下命令。

```
MATCH (e: Employee) DELETE e
```

现在，而不是“返回e”，使用“DELETE e”命令删除Employee节点



![删除Employee节点](https://atts.w3cschool.cn/attachments/day_161227/201612270916092602.png)



**步骤5** - 单击“执行”按钮并观察结果。

![删除Employee节点](https://atts.w3cschool.cn/attachments/day_161227/201612270926105525.png)



这里我们可以看到一个节点从数据库中删除。

现在检查是否从数据库中删除Employee节点。





**步骤6** - 键入以下命令，然后单击执行命令。

```
MATCH (e: Employee) RETURN e
```

![Employee节点被永久删除](https://atts.w3cschool.cn/attachments/day_161227/201612270933014842.png)

在这里我们可以观察到Employee节点被永久删除为零查询返回的行。





### DELETE节点和关系子句语法

```
DELETE <node1-name>,<node2-name>,<relationship-name>
```

| S.No. | 语法元素            | 描述                                                       |
| ----- | ------------------- | ---------------------------------------------------------- |
| 1.    | DELETE              | 它是一个Neo4j CQL关键字。                                  |
| 2.    | <node1-name>        | 它是用于创建关系<relationship-name>的一个结束节点名称。    |
| 3.    | <node2-name>        | 它是用于创建关系<relationship-name>的另一个节点名称。      |
| 4.    | <relationship-name> | 它是一个关系名称，它在<node1-name>和<node2-name>之间创建。 |

**
**

**注意 -**

我们应该使用逗号（，）运算符来分隔节点名称和关系名称。



例如：

此示例演示如何从数据库永久删除节点及其关联节点和关系。



**步骤1** - 打开Neo4j数据浏览器



**步骤2** - 在数据浏览器上键入以下命令

```
MATCH (cc:CreditCard)-[r]-(c:Customer)RETURN r 
```

![MATCH (cc:CreditCard)-[r]-(c:Customer)RETURN r ](https://atts.w3cschool.cn/attachments/day_161227/201612270916101810.png)



**Step 3** **-** 点击“执行”按钮并观察结果。



![节点关系可用](https://atts.w3cschool.cn/attachments/day_161227/201612270916112732.png)



在这里我们观察到一个节点为客户，一个节点为信用卡和它们之间的关系是可用的。

**
**

**步骤4** - 在数据浏览器上键入以下命令

```
MATCH (cc: CreditCard)-[rel]-(c:Customer) 
DELETE cc,c,rel
```

![在数据浏览器上键入以下命令](https://atts.w3cschool.cn/attachments/day_161227/201612270916119810.png)



**步骤5** - 点击“执行”按钮并观察结果。



![节点及关系被删除](https://atts.w3cschool.cn/attachments/day_161227/201612270957448124.png)

这里我们可以观察到两个节点及其关联的10个关系被成功删除。


现在检查DELETE操作是否成功完成。





**步骤6** - 在数据浏览器上键入以下命令。

```
MATCH (cc:CreditCard)-[r]-(c:Customer) RETURN r
```

![检查DELETE操作](https://atts.w3cschool.cn/attachments/day_161227/201612271001119873.png)





**步骤7** - 点击“执行”按钮并观察结果。



![从数据库返回的零行](https://atts.w3cschool.cn/attachments/day_161227/201612270916125124.png)



这里我们可以看到从数据库返回的零行。



# REMOVE删除

有时基于我们的客户端要求，我们需要向现有节点或关系添加或删除属性。

我们使用Neo4j CQL SET子句向现有节点或关系添加新属性。

我们使用Neo4j CQL REMOVE子句来删除节点或关系的现有属性。



Neo4j CQL REMOVE命令用于

- 删除节点或关系的标签
- 删除节点或关系的属性

Neo4j CQL DELETE和REMOVE命令之间的主要区别 - 

- DELETE操作用于删除节点和关联关系。
- REMOVE操作用于删除标签和属性。

Neo4j CQL DELETE和REMOVE命令之间的相似性 - 

- 这两个命令不应单独使用。
- 两个命令都应该与MATCH命令一起使用。

### 删除节点/关系的属性

我们可以使用相同的语法从数据库中永久删除节点或关系的属性或属性列表。



### REMOVE属性子句语法

```
REMOVE <property-name-list>
```

| S.No. | 语法元素             | 描述                                                 |
| ----- | -------------------- | ---------------------------------------------------- |
| 1。   | REMOVE               | 它是一个Neo4j CQL关键字。                            |
| 2。   | <property-name-list> | 它是一个属性列表，用于永久性地从节点或关系中删除它。 |

语法

```
<node-name>.<property1-name>,
<node-name>.<property2-name>, 
.... 
<node-name>.<propertyn-name> 
```

语法说明：

| S.No. | 语法元素        | 描述                 |
| ----- | --------------- | -------------------- |
| 1。   | <node-name>     | 它是节点的名称。     |
| 2。   | <property-name> | 它是节点的属性名称。 |

**
**

**注意 -**

- 我们应该使用逗号（，）运算符来分隔标签名称列表。
- 我们应该使用dot（。）运算符来分隔节点名称和标签名称。

例如：

此示例演示如何创建节点并从数据库中永久删除此节点的属性。



**步骤1** - 打开Neo4j数据浏览器





![打开Neo4j数据浏览器](https://atts.w3cschool.cn/attachments/day_161227/201612271025122643.png)





**步骤2** -在数据浏览器上键入以下命令

```
CREATE (book:Book {id:122,title:"Neo4j Tutorial",pages:340,price:250}) 
```

**
**



**![CREATE (book:Book {id:122,title:"Neo4j Tutorial",pages:340,price:250}) ](https://atts.w3cschool.cn/attachments/day_161227/201612271028411434.png)**





**
**

**步骤3** -点击“执行”按钮并观察结果。





![两个SQL命令](https://atts.w3cschool.cn/attachments/day_161227/201612271029123618.png)

它类似于以下两个SQL命令在一个镜头。



```
CREATE TABLE BOOK(
	id number,
	title varchar2(20),
	pages number,
	pages number
);
INSERT INTO BOOK VALUES (122,'Neo4j Tutorial',340,250);
```

这里我们可以观察到一个标签和一个节点有4个属性被成功创建。



**步骤4** -在数据浏览器上键入以下命令

```
MATCH (book : Book)
RETURN book
```

它类似于下面的SQL命令。

```
SELECT * FROM BOOK;
```

**
**

**步骤5** -点击“执行”按钮并观察结果。





![4个属性](https://atts.w3cschool.cn/attachments/day_161227/201612271032275948.png)

这里我们可以观察到这个书节点有4个属性。



**
**

**步骤6** -在数据浏览器上键入以下命令，然后单击执行按钮从书节点中删除“price”属性。

```
MATCH (book { id:122 })
REMOVE book.price
RETURN book
```

它类似于下面的SQL命令。

```
ALTER TABLE BOOK REMOVE COLUMN PRICE;
SELECT * FROM BOOK WHERE ID = 122;
```





![img](https://atts.w3cschool.cn/attachments/day_161227/201612271033181733.png)



在这里，我们只能看到节点书的3个属性，因为“价格”属性被删除。

有时基于客户端要求，我们需要删除一些现有的属性到节点或关系。



我们需要使用REMOVE子句来删除一个属性或一组属性。





例如

此示例演示如何从数据库中永久删除现有节点的属性。



**步骤1** - 打开Neo4j数据浏览器



**步骤2** - 在数据浏览器上键入以下命令

```
MATCH (dc:DebitCard) 
RETURN dc
```

**
**



**![MATCH (dc:DebitCard)  RETURN dc](https://atts.w3cschool.cn/attachments/day_161227/201612271034465052.png)**





**
**

**步骤3** -点击“执行”按钮并观察结果。





![DebitCard节点包含6个属性](https://atts.w3cschool.cn/attachments/day_161227/201612271036013507.png)

这里我们可以观察到DebitCard节点包含6个属性。





**步骤4** -在数据浏览器上键入以下命令

```
MATCH (dc:DebitCard) 
REMOVE dc.cvv
RETURN dc
```

**
**



**![MATCH (dc:DebitCard)  REMOVE dc.cvv RETURN dc](https://atts.w3cschool.cn/attachments/day_161227/201612271036453719.png)**





**
**

步骤5 -点击“执行”按钮并观察结果。



![img](https://atts.w3cschool.cn/attachments/day_161227/201612271037146724.png)
如果我们观察输出，“cvv”属性从“DebitCard”节点中删除。



### 删除节点/关系的标签

我们可以使用相同的语法从数据库中永久删除节点或关系的标签或标签列表。



REMOVE一个Label子句语法：

```
REMOVE <label-name-list> 
```

S.No.语法元素描述

1.REMOVE它是一个Neo4j CQL关键字。

2.它是一个标签列表，用于永久性地从节点或关系中删除它。


**语法**

```
<node-name>:<label2-name>, 
.... 
<node-name>:<labeln-name> 
```



语法说明：

| S.No. | 语法元素                | 描述                     |
| ----- | ----------------------- | ------------------------ |
| 1。   | <node-name> <节点名称>  | 它是一个节点的名称。     |
| 2。   | <label-name> <标签名称> | 这是一个节点的标签名称。 |

**注意 -**

- 我们应该使用逗号（，）运算符来分隔标签名称列表。
- 我们应该使用colon（:)运算符来分隔节点名和标签名。

例如：

此示例演示如何从数据库永久删除不需要的标签到节点。



**步骤1** - 打开Neo4j数据浏览器



**步骤2** - 在数据浏览器上键入以下命令

```
MATCH (m:Movie) RETURN m
```

**
**



**![MATCH (m:Movie) RETURN m](https://atts.w3cschool.cn/attachments/day_161227/201612271041011939.png)**





**
**

**步骤3** -点击“执行”按钮并观察结果。





**![执行结果](https://atts.w3cschool.cn/attachments/day_161227/201612271043101562.png)**





**
**

步骤4 -点击节点以查看其属性窗口。





![查看其属性窗口](https://atts.w3cschool.cn/attachments/day_161227/201612271043369880.png)

在这里我们可以观察到四个标签可用于单个节点。





根据我们的客户要求，我们需要删除“图片”标签到此节点。



**步骤5** -在浏览器上键入以下命令，然后单击执行按钮。

```
MATCH (m:Movie) 
REMOVE m:Picture
```

**
**



**![MATCH (m:Movie)  REMOVE m:Picture](https://atts.w3cschool.cn/attachments/day_161227/201612271044452174.png)**

**步骤6** -点击“执行”按钮并观察结果。







![一个标签从数据库永久删除的节点](https://atts.w3cschool.cn/attachments/day_161227/201612271046063445.png)

在这里我们可以观察到一个标签从数据库永久删除的节点。





**步骤7** -在数据浏览器上键入以下命令

```
MATCH (m:Movie) RETURN m
```



**![MATCH (m:Movie) RETURN m](https://atts.w3cschool.cn/attachments/day_161227/201612271046374722.png)**



**
**

**步骤8** -点击“执行”按钮并观察结果。



![结果](https://atts.w3cschool.cn/attachments/day_161227/201612271047291210.png)

**
**

**步骤9** -单击节点以查看其属性窗口。





![命令已成功删除](https://atts.w3cschool.cn/attachments/day_161227/201612271048198051.png)

这里我们可以观察到这个节点只有三个标签：Movie，Cinema，Film from Properties Window。 这意味着我们的上一个命令已成功删除图片标签



# SET子句

有时，根据我们的客户端要求，我们需要向现有节点或关系添加新属性。

要做到这一点，Neo4j CQL提供了一个SET子句。

Neo4j CQL已提供SET子句来执行以下操作。

- 向现有节点或关系添加新属性
- 添加或更新属性值

### SET子句语法

```
SET  <property-name-list>
```

| S.No. | 语法元素             | 描述                                                       |
| ----- | -------------------- | ---------------------------------------------------------- |
| 1     | SET                  | 它是一个Neo4j的CQL关键字。                                 |
| 2     | <property-name-list> | 它是一个属性列表，用于执行添加或更新操作以满足我们的要求。 |

**<属性名称列表>语法：**

```
<node-label-name>.<property1-name>,
<node-label-name>.<property2-name>, 
.... 
<node-label-name>.<propertyn-name> 
```



语法说明：

| S.No. | 语法元素                         | 描述                     |
| ----- | -------------------------------- | ------------------------ |
| 1     | <node-label-name> <节点标签名称> | 这是一个节点的标签名称。 |
| 2     | <property-name> <属性名称>       | 它是一个节点的属性名。   |

**
**

**注意 -**

我们应该使用逗号（，）运算符来分隔属性名列表。



例如：

此示例演示如何向现有DebitCard节点添加新属性。



**步骤1** -打开Neo4j数据浏览器





**![DebitCard节点添加新属性](https://atts.w3cschool.cn/attachments/day_161227/201612271518233594.png)**





**
**

步骤2 -在数据浏览器上键入以下命令

```
MATCH (dc:DebitCard)
RETURN dc
```



**![MATCH (dc:DebitCard) RETURN dc](https://atts.w3cschool.cn/attachments/day_161227/201612271521097425.png)**

**步骤3** -点击“执行”按钮并观察结果。







![img](https://atts.w3cschool.cn/attachments/day_161227/201612271534218645.png)





这里我们可以观察到“DebitCard”节点有5个属性。 现在我们将向此节点添加新属性“atm_pin”。

**
**

**步骤4** -在数据浏览器上键入以下命令

```
MATCH (dc:DebitCard)
SET dc.atm_pin = 3456
RETURN dc
```

**
**



**![命令](https://atts.w3cschool.cn/attachments/day_161227/201612271539175094.png)**





**
**

步骤5 -点击“执行”按钮并观察结果。





![这里我们可以观察到新的属性被添加到“DebitCard”节点。](https://atts.w3cschool.cn/attachments/day_161227/201612271539563571.png)

这里我们可以观察到新的属性被添加到“DebitCard”节点。