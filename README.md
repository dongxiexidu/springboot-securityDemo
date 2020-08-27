# springboot-securityDemo
 springboot-security Demo
 
Spring Security是一个功能强大且高度可定制的身份验证和访问控制框架。它实际上是保护基于spring的应用程序的标准。

Spring 是一个非常流行和成功的 Java 应用开发框架。Spring Security 基于 Spring 框架，提供了一套 Web 应用安全性的完整解决方案。一般来说，Web 应用的安全性包括**用户认证**（Authentication）和**用户授权**（Authorization）两个部分

### “认证”（Authentication）

身份验证是关于验证您的凭据，如用户名/用户ID和密码，以验证您的身份。

身份验证通常通过用户名和密码完成，有时与身份验证因素结合使用。
```java
// 认证
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {

    // 必须要加密,不给使用明文
    // 这些数据正常应该从数据库中读取
    auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
            .withUser("ldx").password(new BCryptPasswordEncoder().encode("123456")).roles("vip2","vip3")
            .and()
            .withUser("root").password(new BCryptPasswordEncoder().encode("123456")).roles("vip2","vip3","vip1")
            .and()
            .withUser("guest").password(new BCryptPasswordEncoder().encode("123456")).roles("vip1");
}
```

###  “授权” （Authorization）

授权发生在系统成功验证您的身份后，最终会授予您访问资源（如信息，文件，数据库，资金，位置，几乎任何内容）的完全权限。

```java
// 授权
@Override
protected void configure(HttpSecurity http) throws Exception {

    http.authorizeRequests()
            .antMatchers("/").permitAll()
            .antMatchers("/level1/**").hasRole("vip1")
            .antMatchers("/level2/**").hasRole("vip2")
            .antMatchers("/level3/**").hasRole("vip3");

    // formLogin()没有权限默认会到登录页面
    // loginPage("/login") 默认会使用系统的登录页,使用自己的登录页面
    http.formLogin().loginPage("/login").usernameParameter("username").passwordParameter("password").loginProcessingUrl("/login");
    
    // 开启注销,注销完毕默认会跳转到登录页面！
    // logoutSuccessUrl("/"),可以指定跳到首页
    http.logout().logoutSuccessUrl("/");
    http.csrf().disable();
    // 开启记住我功能
    http.rememberMe().rememberMeParameter("remember-me");

}
```

### 依赖
```xml
<dependencies>

   <dependency>
       <groupId>org.thymeleaf.extras</groupId>
       <artifactId>thymeleaf-extras-springsecurity5</artifactId>
       <version>3.0.4.RELEASE</version>
   </dependency>

   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-security</artifactId>
       <version>2.3.1.RELEASE</version>
   </dependency>

   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-thymeleaf</artifactId>
       <version>2.3.1.RELEASE</version>
   </dependency>

   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
</dependencies>
```

### Controller
```java
@Controller
public class RouterController {

    // 首页
    @RequestMapping({"/", "/index"})
    public String index() {
        return "index";
    }

    // 登录
    @RequestMapping("/login")
    public String toLogin() {
        return "views/login";
    }

    @RequestMapping("/level1/{id}")
    public String level1(@PathVariable("id") int id) {
        return "/views/level1/"+id;
    }

    @RequestMapping("/level2/{id}")
    public String level2(@PathVariable("id") int id) {
        return "/views/level2/"+id;
    }
    @RequestMapping("/level3/{id}")
    public String level3(@PathVariable("id") int id) {
        return "/views/level3/"+id;
    }
}
```
