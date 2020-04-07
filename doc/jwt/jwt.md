
# java集成jwt

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

