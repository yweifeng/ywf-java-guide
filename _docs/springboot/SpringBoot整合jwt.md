---
title: SpringBoot整合jwt
category: springboot
category-order: 1
order: 18
---

## 什么是jwt

> Json web token (JWT), 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（(RFC 7519).该token被设计为紧凑且安全的，特别适用于分布式站点的单点登录（SSO）场景。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。



## jwt有什么好处

1、支持跨域访问: Cookie是不允许垮域访问的，这一点对Token机制是不存在的，前提是传输的用户认证信息通过HTTP头传输.

2、无状态(也称：服务端可扩展行):Token机制在服务端不需要存储session信息，因为Token 自身包含了所有登录用户的信息，只需要在客户端的cookie或本地介质存储状态信息.

4、更适用CDN: 可以通过内容分发网络请求你服务端的所有资料（如：javascript，HTML,图片等），而你的服务端只要提供API即可.（居于前面两点得出这个更适用于CDN内容分发网络）

5、去耦: 不需要绑定到一个特定的身份验证方案。Token可以在任何地方生成，只要在你的API被调用的时候，你可以进行Token生成调用即可.（这个似乎也在继续说前面第一点和第二点的好处。。。）

6、更适用于移动应用: 当你的客户端是一个原生平台（iOS, Android，Windows 8等）时，Cookie是不被支持的（你需要通过Cookie容器进行处理），这时采用Token认证机制就会简单得多。

7、CSRF:因为不再依赖于Cookie，所以你就不需要考虑对CSRF（跨站请求伪造）的防范。（如果token是用cookie保存，CSRF还是需要考虑,一般建议使用1、在HTTP请求中以参数的形式加入一个服务器端产生的token。或者2.放入http请求头中也就是一次性给所有该类请求加上csrftoken这个HTTP头属性，并把token值放入其中） 
ps:后面会推出一些常见的网络安全的处理

8、性能: 一次网络往返时间（通过数据库查询session信息）总比做一次HMACSHA256计算 的Token验证和解析要费时得多.

9、不需要为登录页面做特殊处理: 如果你使用Protractor 做功能测试的时候，不再需要为登录页面做特殊处理.（知识面太窄，不是太明白这个的意思）

10、基于标准化:你的API可以采用标准化的 JSON Web Token (JWT). 这个标准已经存在多个后端库（.NET, Ruby, Java,Python, PHP）和多家公司的支持（如：Firebase,Google, Microsoft）



## jwt的组成

**header.payload.signature**（中间用.隔开）

### header

jwt的头部承载两部分信息：

```
声明类型，这里是jwt
声明加密的算法 通常直接使用 HMAC SHA256
```

header模拟数据：

```json
{
  'typ': 'JWT',
  'alg': 'HS256'
}
```

base64加密得到：

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```



### payload

包含三个部分：

```
标准中注册的声明
公共的声明
私有的声明
```

**标准中注册的声明：**

```
iss: jwt签发者
sub: jwt所面向的用户
aud: 接收jwt的一方
exp: jwt的过期时间，这个过期时间必须要大于签发时间
nbf: 定义在什么时间之前，该jwt都是不可用的.
iat: jwt的签发时间
jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。
```

**公共的声明:**

```
公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息，因为该部分在客户端可解密.
```

**私有的声明：**

```
私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。

这个指的就是自定义的claim。比如前面那个结构举例中的admin和name都属于自定的claim。这些claim跟JWT标准规定的claim区别在于：JWT规定的claim，JWT的接收方在拿到JWT之后，都知道怎么对这些标准的claim进行验证(还不知道是否能够验证)；而private claims不会验证，除非明确告诉接收方要对这些claim进行验证以及规则才行。
```

payload模拟数据：

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

base64加密得到：

```
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
```



### signature

签证信息由三部分组成:

```
header (base64加密后的)
payload (base64加密后的)
secret
```

获取方式如下：

```javascript
// javascript
var encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);
 
var signature = HMACSHA256(encodedString, 'secret'); // TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```



完整的jwt数据如下：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```



# SpringBoot整合jwt

**maven.pom**

```xml
<dependencies>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
        <version>0.9.1</version>
    </dependency>
    <dependency>
        <groupId>com.auth0</groupId>
        <artifactId>java-jwt</artifactId>
        <version>3.10.0</version>
    </dependency>
</dependencies>
```

**JwtUtil.java**

```java
package com.ywf.jwt.util;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtBuilder;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;

import java.security.NoSuchAlgorithmException;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * jwt工具类
 */
public class JwtUtil {

    private final static String JWT_SECRET = "ywf jwt";

    /**
     * 生成jwt
     * @param jwtId 可传入业务id, 唯一
     * @param sub  代表这个JWT的主体，即它的所有人，这个是一个json格式的字符串，
     *             可以存放什么userid，roldid之类的，作为什么用户的唯一标志
     * @param expTime 过期时间长度  如30000
     * @return
     */
    public static String createJwt(String jwtId, String sub, long expTime) throws NoSuchAlgorithmException {
        String jwt = "";

        // 设置claims 存储用户信息
        Map<String, Object> claims = new HashMap<String, Object>();
        claims.put("uid", "id0001");
        claims.put("name", "ywf");

        // 签发时间
        long currentTimeMillis = System.currentTimeMillis();
        Date now = new Date(currentTimeMillis);

        // 过期时间
        Date expiration = new Date(currentTimeMillis + expTime);

        JwtBuilder jwtBuilder = Jwts.builder()
                .setClaims(claims)
                .setId(jwt)
                .setIssuedAt(now)
                .setSubject(sub)
                .setExpiration(expiration)
                .signWith(SignatureAlgorithm.HS256, JWT_SECRET);
        jwt = jwtBuilder.compact();
        System.out.println("jwt = " + jwt);
        return jwt;
    }

    /**
     * 解析jwt
     * @param jwt
     * @return
     */
    public static Claims parseJwt(String jwt) {
        Claims claims = Jwts.parser().setSigningKey(JWT_SECRET).parseClaimsJws(jwt).getBody();
        return  claims;
    }

}
```

**测试**

```java
public static void main(String[] args) throws NoSuchAlgorithmException {
    String jwtId = "yang";
    String sub = "subject info";
    long time = 300000;
    String jwt = JwtUtil.createJwt(jwtId, sub, time);

    // 解析jwt
    Claims claims = JwtUtil.parseJwt(jwt);
    System.out.println(claims.get("uid", String.class));
    System.out.println(claims.get("name", String.class));
    System.out.println(claims.getSubject());
}
```

**执行结果**

```bash
jwt = eyJhbGciOiJIUzI1NiJ9.eyJ1aWQiOiJpZDAwMDEiLCJzdWIiOiJzdWJqZWN0IGluZm8iLCJuYW1lIjoieXdmIiwiZXhwIjoxNTg2MjUzMjYwLCJpYXQiOjE1ODYyNTI5NjAsImp0aSI6IiJ9.Z8V4BP27Yn9gCCxgc4HFhkY--nkIzQP245sFYl8PI6M
id0001
ywf
subject info
```

