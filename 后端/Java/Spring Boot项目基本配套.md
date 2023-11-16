# Spring Boot项目基本配套

好久没有写`spring boot`了，最近想整理一下`spring boot`项目的最基本的框架的搭建及一些别的配套





### 三层架构

`spring boot`项目最经典的三层架构：`controller`、`service`、`mapper`，我们简单概述一下

1. `controller`层就是接口本身，它对外提供访问，调用`service`层的服务

2. `service`层实现具体的业务逻辑，它一般分有实现类和接口

3. `mapper`层专注于数据库操作，并对`service`层外提供接口（有的还会分出来一个`dao`层）





### 统一返回结果类

这应该是每个项目所必需的，封装一个`Result<T>`类，统一返回不同的`data`

```java
package com.lordmoon.pojo;


/**
 * 统一响应结果类
 */

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Result<T> {
    private Integer code; // 业务状态码  0-成功  1-失败
    private String message; // 提示信息
    private T data; // 响应数据

    // 操作成功响应结果(带响应数据)
    public static <T> Result success(T data) {
        return new Result<>(0, "操作成功", data);
    }

    // 操作成功响应结果(无数据)
    public static Result success() {
        return new Result(0, "操作成功", null);
    }

    // 操作失败响应结果
    public static Result error(String message) {
        return new Result(1, message, null);
    }
}
```





### 整合MySql

数据库是必须的，首先引入依赖

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

然后在`yml`配置文件中配置属性

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/big_event
    username: root
    password: *****
```





### 整合Mybatis

在`mapper`层中，操作数据库一般都是用`mybatis`或者`mybatis plus`，可以大大简化开发

依赖如下，具体如何使用我到时候会单独写一篇博客

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```





### Token签发及校验

`token`的签发校验一般都是封装好一个工具类，我建议直接贴到代码片段中，用的时候直接粘贴就好了

封装工具类之前，首先我们要引入`JWT`的依赖

```xml
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>4.4.0</version>
</dependency>
```

然后下面是我的`JwtUtil`的封装，可以参考一下，其实都大差不差，网上找一个看懂了复制粘贴就行

```java
package com.lordmoon.utils;

import com.auth0.jwt.JWT;
import com.auth0.jwt.algorithms.Algorithm;

import java.util.Date;
import java.util.Map;

public class JwtUtil {

    private static final String KEY = "ldm-key";

    // 接收业务数据,生成token并返回
    public static String genToken(Map<String, Object> claims) {
        return JWT.create()
                .withClaim("claims", claims)
                .withExpiresAt(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 12))
                .sign(Algorithm.HMAC256(KEY));
    }

    // 接收token,验证token,并返回业务数据
    public static Map<String, Object> parseToken(String token) {
        return JWT.require(Algorithm.HMAC256(KEY))
                .build()
                .verify(token)
                .getClaim("claims")
                .asMap();
    }

}
```





### MD5数据加密

在业务中，数据加密是必须的，例如用户的密码不能明文存储，我们直接来看封装

```java
package com.lordmoon.utils;


import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

/**
 * MD5加密工具类
 */
public class Md5Util {

    // 默认的密码字符串组合，用来将字节转换成 16 进制表示的字符,apache校验下载的文件的正确性用的就是默认的这个组合
    protected static char hexDigits[] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};

    protected static MessageDigest messagedigest = null;

    static {
        try {
            messagedigest = MessageDigest.getInstance("MD5");
        } catch (NoSuchAlgorithmException nsaex) {
            System.err.println(Md5Util.class.getName() + "初始化失败，MessageDigest不支持MD5Util。");
            nsaex.printStackTrace();
        }
    }

    /**
     * 生成字符串的md5校验值
     *
     * @param s
     * @return
     */
    public static String getMD5String(String s) {
        return getMD5String(s.getBytes());
    }

    /**
     * 判断字符串的md5校验码是否与一个已知的md5码相匹配
     *
     * @param password  要校验的字符串
     * @param md5PwdStr 已知的md5校验码
     * @return
     */
    public static boolean checkPassword(String password, String md5PwdStr) {
        String s = getMD5String(password);
        return s.equals(md5PwdStr);
    }


    public static String getMD5String(byte[] bytes) {
        messagedigest.update(bytes);
        return bufferToHex(messagedigest.digest());
    }

    private static String bufferToHex(byte bytes[]) {
        return bufferToHex(bytes, 0, bytes.length);
    }

    private static String bufferToHex(byte bytes[], int m, int n) {
        StringBuffer stringbuffer = new StringBuffer(2 * n);
        int k = m + n;
        for (int l = m; l < k; l++) {
            appendHexPair(bytes[l], stringbuffer);
        }
        return stringbuffer.toString();
    }

    private static void appendHexPair(byte bt, StringBuffer stringbuffer) {
        char c0 = hexDigits[(bt & 0xf0) >> 4]; // 取字节中高 4 位的数字转换, >>>
        // 为逻辑右移，将符号位一起右移,此处未发现两种符号有何不同
        char c1 = hexDigits[bt & 0xf]; // 取字节中低 4 位的数字转换
        stringbuffer.append(c0);
        stringbuffer.append(c1);
    }

}
```





### ThreadLocal工具类

在项目中，往往需要拿到拦截器中解析`token`得到的用户`claims`，而我们又不应该重复地解析`token`

所以我们就维护一个全局的`ThreadLocal`对象，将每次请求的那个线程内的数据存储到`ThreadLocal`中

这样我们就可以在本次请求时的其他任何地方都拿到这些数据，直接来看如何封装

```java
package com.lordmoon.utils;

import java.util.HashMap;
import java.util.Map;

/**
 * ThreadLocal 工具类
 */
@SuppressWarnings("all")
public class ThreadLocalUtil {
    // 提供ThreadLocal对象常量，维护全局单例的ThreadLocal对象
    private static final ThreadLocal THREAD_LOCAL = new ThreadLocal();

    // 根据键获取值
    public static <T> T get() {
        return (T) THREAD_LOCAL.get();
    }

    // 存储键值对
    public static void set(Object value) {
        THREAD_LOCAL.set(value);
    }


    // 清除ThreadLocal 防止内存泄漏
    public static void remove() {
        THREAD_LOCAL.remove();
    }
}
```





### 添加统一拦截器

为了实现后端身份校验，而又不想在每个接口内都解析`token`，就需要在所有请求打到接口前拦截并校验

我们首先定义一个拦截器类，该拦截器会在请求到达前和完成后作出某些操作

```java
package com.lordmoon.interceptors;

import com.lordmoon.pojo.Result;
import com.lordmoon.utils.JwtUtil;
import com.lordmoon.utils.ThreadLocalUtil;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;

import java.util.Map;

@Component // 自动注入到spring容器中
public class LoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handle) throws Exception {
        String token = request.getHeader("Authorization");
        try {
            Map<String, Object> claims = JwtUtil.parseToken(token);
            ThreadLocalUtil.set(claims); // 用户信息放入ThreadLocal

            return true; // true表示放行
        } catch (Exception e) {
            e.printStackTrace();
            response.setStatus(401);
            return false;
        }
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        ThreadLocalUtil.remove(); // 请求完成后清空TreadLocal中的数据
    }
}
```

然后我们定义一个配置类，在配置类中添加上该拦截器

```java
package com.lordmoon.config;

import com.lordmoon.interceptors.LoginInterceptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration // 自动注入到spring容器中
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private LoginInterceptor loginInterceptor; // 注入自定义的登录拦截器

    /**
     * 添加拦截器
     *
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loginInterceptor)
                .excludePathPatterns("/user/login", "/user/register");
    }
}
```





### 全局异常处理

全局异常处理主要说的是处理`controller`层的异常，其他层的异常就可以向上抛或者直接`catch`

这里用到了`@RestControllerAdvide`注解，我们直接看如何写

```java
package com.lordmoon.exception;

import com.lordmoon.pojo.Result;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

/**
 * 全局异常处理
 */
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public Result handleException(Exception e) {
        e.printStackTrace();
        // 先判断该异常有无输出信息
        return Result.error(StringUtils.hasLength(e.getMessage()) ? e.getMessage() : "操作失败");
    }
}
```





### 单元测试

配置好了上面的工具类和全局异常处理，我们就需要进行测试，这就需要用到`spring boot`的单元测试，首先引入依赖

```xml
<!--  单元测试依赖  -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>

<!--  junit依赖  -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>test</scope>
</dependency>
```

然后我们在`test`文件夹中创建类来测试即可，如下是我测试`JWT`工具类的示例

```java
package com.lordmoon;

import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.interfaces.Claim;
import com.auth0.jwt.interfaces.DecodedJWT;
import org.junit.jupiter.api.Test;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * JWT包含3部分：
 * header、payload、signature
 */
public class JwtTest {

    @Test
    public void testGenerate() {
        Map<String, Object> claims = new HashMap<>();
        claims.put("username", "lordmoon");
        claims.put("id", 1);

        String token = JWT.create()
                .withClaim("user", claims) // 添加payload
                .withExpiresAt(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 12)) // 添加过期时间
                .sign(Algorithm.HMAC256("ldm-key")); // 设置签名算法
        System.out.println(token);
    }

    @Test
    public void testParse() {
        String token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9" +
                ".eyJ1c2VyIjp7ImlkIjoxLCJ1c2VybmFtZSI6ImxvcmRtb29uIn0sImV4cCI6MTY5OTM5OTc1NX0" +
                ".JEqwCzYPt7jbkxBW_bd_V9LOrbXr8DugE43F55jIqJg";

        JWTVerifier jwtVerifier = JWT.require(Algorithm.HMAC256("ldm-key")).build();

        DecodedJWT decodedJWT = jwtVerifier.verify(token); // 生成解析后的JWT对象
        Map<String, Claim> claims = decodedJWT.getClaims(); // 得到所有载荷

        System.out.println(claims.get("user"));
    }
}
```

