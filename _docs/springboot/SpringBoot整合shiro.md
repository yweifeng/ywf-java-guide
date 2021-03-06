---
title: SpringBoot整合shiro
category: springboot
order: 21
---



### Shiro架构与功能介绍

#### **认证与授权相关基本概念**

**安全实体**：系统需要保护的具体对象数据

**权限**：系统相关的功能操作，例如基本的CRUD

**Authentication**：身份认证/登录，验证用户是不是拥有相应的身份；

**Authorization**：授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用户是否能做事情，常见的如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限；

**Session Manager**：会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；会话可以是普通JavaSE环境的，也可以是如Web环境的；

**Cryptography**：加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储；

**Web Support**：Web支持，可以非常容易的集成到Web环境；

**Caching**：缓存，比如用户登录后，其用户信息、拥有的角色/权限不必每次去查，这样可以提高效率；

**Concurrency**：shiro支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能把权限自动传播过去；

**Testing**：提供测试支持；

**Run As**：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问；

**Remember Me**：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了。

#### Shiro四大核心功能

**Authentication,Authorization,Cryptography,Session Management**

![img](../../images/springboot/sb28.png)



#### Shiro三个核心组件

**Subject**：主体，代表了当前“用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是Subject，如网络爬虫，机器人等；即一个抽象概念；所有Subject都绑定到SecurityManager，与Subject的所有交互都会委托给SecurityManager；可以把Subject认为是一个门面；SecurityManager才是实际的执行者；

**SecurityManager**：安全管理器；即所有与安全有关的操作都会与SecurityManager交互；且它管理着所有Subject；可以看出它是Shiro的核心，它负责与后边介绍的其他组件进行交互，如果学习过SpringMVC，你可以把它看成DispatcherServlet前端控制器；

**Realm**：域，Shiro从从Realm获取安全数据（如用户、角色、权限），就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法；也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作；可以把Realm看成DataSource，即安全数据源。



### 代码实现

#### pom.xml

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.5.3</version>
</dependency>
```



#### ShiroConfig.java

```java
package com.ywf.springboot.config;

import com.ywf.springboot.shiro.realm.CustomRealm;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class ShiroConfig {

    //不加这个注解不生效，具体不详
    @Bean
    @ConditionalOnMissingBean
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
        DefaultAdvisorAutoProxyCreator defaultAAP = new DefaultAdvisorAutoProxyCreator();
        defaultAAP.setProxyTargetClass(true);
        return defaultAAP;
    }

    //将自己的验证方式加入容器
    @Bean
    public CustomRealm myShiroRealm() {
        CustomRealm customRealm = new CustomRealm();
        return customRealm;
    }

    //权限管理，配置主要是Realm的管理认证
    @Bean
    public SecurityManager securityManager() {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(myShiroRealm());
        return securityManager;
    }

    //Filter工厂，设置对应的过滤条件和跳转条件
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        Map<String, String> map = new HashMap<>();
        //登出
        map.put("/logout", "logout");
        //对所有用户认证
        map.put("/**", "authc");
        //登录
        shiroFilterFactoryBean.setLoginUrl("/login");
        //首页
        shiroFilterFactoryBean.setSuccessUrl("/index");
        //错误页面，认证不通过跳转
        shiroFilterFactoryBean.setUnauthorizedUrl("/error");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(map);
        return shiroFilterFactoryBean;
    }

    //加入注解的使用，不加入这个注解不生效
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
        authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
        return authorizationAttributeSourceAdvisor;
    }
}

```



#### User.java

```java
package com.ywf.springboot.entity;

import java.util.Set;

public class User {
    private String id;
    private String userName;
    private String password;

    public User(String id, String userName, String password) {
        this.id = id;
        this.userName = userName;
        this.password = password;
    }

    /**
     * 用户对应的角色集合
     */
    private Set<Role> roles;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Set<Role> getRoles() {
        return roles;
    }

    public void setRoles(Set<Role> roles) {
        this.roles = roles;
    }
}

```



#### Role.java

```java
package com.ywf.springboot.entity;

import java.util.Set;

public class Role {
    private String id;
    private String roleName;

    public Role(String id, String roleName) {
        this.id = id;
        this.roleName = roleName;
    }

    /**
     * 角色对应权限集合
     */
    private Set<Permission> permissions;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getRoleName() {
        return roleName;
    }

    public void setRoleName(String roleName) {
        this.roleName = roleName;
    }

    public Set<Permission> getPermissions() {
        return permissions;
    }

    public void setPermissions(Set<Permission> permissions) {
        this.permissions = permissions;
    }
}

```



#### Permission.java

```java
package com.ywf.springboot.entity;

public class Permission {
    private String id;
    private String permissionsName;

    public Permission(String id, String permissionsName) {
        this.id = id;
        this.permissionsName = permissionsName;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getPermissionsName() {
        return permissionsName;
    }

    public void setPermissionsName(String permissionsName) {
        this.permissionsName = permissionsName;
    }
}

```



#### CustomRealm.java

```java
package com.ywf.springboot.shiro.realm;

import com.ywf.springboot.entity.Permission;
import com.ywf.springboot.entity.Role;
import com.ywf.springboot.entity.User;
import com.ywf.springboot.shiro.service.LoginService;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;

import java.util.Set;

public class CustomRealm extends AuthorizingRealm {

    @Autowired
    private LoginService loginService;

    /**
     * 用户授权
     * @param principalCollection
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        String userName = (String) principalCollection.getPrimaryPrincipal();
        Set<Role> roleSet = loginService.getRoleByUserName(userName);
        // 添加权限
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        if (userName != null) {
            for (Role role : roleSet) {
                authorizationInfo.addRole(role.getRoleName());
                for (Permission permission : role.getPermissions()) {
                    authorizationInfo.addStringPermission(permission.getPermissionsName());
                }
            }
        }
        return authorizationInfo;
    }

    /**
     * 登录验证
     * @param authenticationToken
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        // 获取用户基本信息
        String userName = (String) authenticationToken.getPrincipal();
        User user = loginService.getByUserName(userName);
        if (null == user) {
            return null;
        }
        return new SimpleAuthenticationInfo(userName, user.getPassword(), user.getUserName());
    }
}

```



#### LoginService.java

```java
package com.ywf.springboot.shiro.service;

import com.ywf.springboot.entity.Role;
import com.ywf.springboot.entity.User;

import java.util.Set;

public interface LoginService {
    User getByUserName(String userName);

    Set<Role> getRoleByUserName(String userName);
}

```



#### LoginServiceImpl.java

```java
package com.ywf.springboot.shiro.service.impl;

import com.ywf.springboot.entity.Permission;
import com.ywf.springboot.entity.Role;
import com.ywf.springboot.entity.User;
import com.ywf.springboot.shiro.service.LoginService;
import org.springframework.stereotype.Component;

import java.util.LinkedHashSet;
import java.util.Set;

@Component
public class LoginServiceImpl implements LoginService {
    @Override
    public User getByUserName(String userName) {
        if ("ywf".equalsIgnoreCase(userName)) {
            return new User("001", userName, "123456");
        } else if ("wyp".equalsIgnoreCase(userName)) {
            return new User("00", userName, "123456");
        } else {
            return null;
        }
    }

    @Override
    public Set<Role> getRoleByUserName(String userName) {
        // 模拟数据 设置用户信息、角色和权限
        if ("ywf".equalsIgnoreCase(userName)) {
            // 设置角色、权限
            Role role = new Role("001", "管理员");
            role.setPermissions(new LinkedHashSet<Permission>(){{
                add(new Permission("001", "添加"));
                add(new Permission("002", "删除"));
                add(new Permission("002", "查看"));
            }});
            return new LinkedHashSet<Role>() {{add(role);}};
        } else if ("ywf".equalsIgnoreCase(userName)) {
            // 设置角色、权限
            Role role = new Role("002", "普通用户");
            role.setPermissions(new LinkedHashSet<Permission>(){{
                add(new Permission("003", "查看"));
            }});
            return new LinkedHashSet<Role>() {{add(role);}};
        } else {
            return new LinkedHashSet();
        }
    }
}

```



#### MyExceptionHandler.java

```java
package com.ywf.springboot.exception;

import org.apache.shiro.authz.AuthorizationException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

@ControllerAdvice
public class MyExceptionHandler {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @ExceptionHandler
    @ResponseBody
    public String ErrorHandler(AuthorizationException e) {
        logger.error("没有通过权限验证！", e);
        return "没有通过权限验证！";
    }
}

```



#### LoginController.java

```java
package com.ywf.springboot.controller;

import com.ywf.springboot.entity.User;
import com.ywf.springboot.shiro.realm.CustomRealm;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.authz.AuthorizationException;
import org.apache.shiro.authz.annotation.RequiresPermissions;
import org.apache.shiro.authz.annotation.RequiresRoles;
import org.apache.shiro.subject.Subject;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.*;

@RestController
public class LoginController {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @PostMapping("/login")
    public String login(@RequestBody User user) {
        // 添加用户认证信息
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken(user.getUserName(), user.getPassword());
        try {
            subject.login(token);
        } catch (AuthenticationException e) {
            logger.info(e.getMessage(), e.getCause());
            return "用户名密码错误";
        } catch (AuthorizationException e) {
            logger.info(e.getMessage(), e.getCause());
            return "用户名无权限";
        }
        return "login success";
    }

    @RequiresRoles("管理员")
    @RequiresPermissions("添加")
    @GetMapping("/add")
    public String add() {
        return "add";
    }
}

```



### 测试

- 管理员角色

  - 用户登录

    ![img](../../images/springboot/sb24.png)

  - 执行添加操作

    ![img](../../images/springboot/sb25.png)

- 普通用户

  - 登录

    ![img](../../images/springboot/sb26.png)

  - 执行添加操作

    ![img](../../images/springboot/sb27.png)