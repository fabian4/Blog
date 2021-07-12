---
title: JWT 的实现和与 SpringSecurity的整合
date: 2020/6/21
description: JWT 的实现和与 SpringSecurity的整合
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/dPNkq5pH6Rm9A4K.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/dPNkq5pH6Rm9A4K.jpg'
categories:
  - Spring
tags:
  - JWT
  - SpringSecurity
abbrlink: 38317
---

# JWT 的实现和与 SpringSecurity的整合

## 一、Token的生成与校验

### 1. 引入maven依赖

~~~xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.6.0</version>
</dependency>
~~~

### 2. Token的生成

这里我们在测试类中进行测试

~~~java
 @Test
    void jwt_generate_test() {
        JwtBuilder builder= Jwts.builder()
            .setId("888")//设置id
            .setSubject("丁生")//设置user对象
            .setIssuedAt(new Date())//设置签发时间
            .signWith(SignatureAlgorithm.HS256,"fabian");//设置签名秘钥
        System.out.println( builder.compact() );
    }
~~~

我们就可以在控制台找到我们生成的 Token 了

~~~text
eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiIwMDEiLCJzdWIiOiLkuIHnlJ8iLCJpYXQiOjE1OTI1NTgwODV9.VgAUtgjVW5adx85GoEcGIP6-w5VtBtYttA_zsazEO4M 
~~~

### 3. Token的校验

~~~java
@Test
    void jwt_check_test() {
        String token="eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiLkuIHnlJ8iLCJpYXQiOjE1OTI1NTc3MjB9.Dn1w7uIpTZnCLF3X21bR5J3nBqKbuqSJqnA3yn6kpAQ";
        Claims claims = Jwts.parser().setSigningKey("fabian").parseClaimsJws(token).getBody();
        System.out.println("id:"+claims.getId());
        // id:888
        System.out.println("subject:"+claims.getSubject());
        // subject:丁生
        System.out.println("IssuedAt:"+claims.getIssuedAt());
        // IssuedAt:Fri Jun 19 17:08:40 CST 2020
    }
~~~

### 4. 异常的 Token

**在解码 Token 的时候，我们会先检查签名的有效性，如果签名无效则直接抛出异常。**

当我们将 Token 修改后再进行校验时，会直接告诉我们这个 Token 不能被信任

![image-20200619172740916](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/GlXNwnbU4T5Oxpd.png)

## 二、整合 JWT 和 SpringSecurity

### 1.  JWT 生成和校验工具类

~~~java
@Component
@RequiredArgsConstructor
public class TokenProvider{
    public String createToken(Authentication authentication) {
        System.out.println("开始通过授权登录信息生成token");
        String authorities = authentication.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority)
                .collect(Collectors.joining(","));
        return Jwts.builder()
                .setSubject(authentication.getName())
                .claim("auth", authorities)
                .signWith(SignatureAlgorithm.HS512, "fabian")
                .compact();
    }
    Authentication getAuthentication(String token) {
        // 解析token 获取用户权限
        System.out.println("开始从token中解析信息");
        Claims claims = Jwts.parser()
                .setSigningKey("fabian")
                .parseClaimsJws(token)
                .getBody();
        System.out.println("解析结果："+claims.toString());
        Object authoritiesStr = claims.get("auth");
        Collection<? extends GrantedAuthority> authorities =
                !isEmpty(authoritiesStr) ?
                        Arrays.stream(authoritiesStr.toString().split(","))
                                .map(SimpleGrantedAuthority::new)
                                .collect(Collectors.toList()) : Collections.emptyList();
        User principal = new User(claims.getSubject(), "", authorities);
        return new UsernamePasswordAuthenticationToken(principal, token, authorities);
    }

}
~~~

### 2. JWT 拦截操作器

~~~java
@RequiredArgsConstructor
public class TokenFilter extends GenericFilterBean {
    private final TokenProvider tokenProvider;
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException {
        System.out.println("进入自定义TokenFilter拦截器==========================");
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        String token = resolveToken(httpServletRequest);
        System.out.println("获取token："+token);
        if(token!=null){
            Authentication authentication = tokenProvider.getAuthentication(token);
            System.out.println("获取到的权限信息： "+authentication.getAuthorities());
        }
        filterChain.doFilter(servletRequest, servletResponse);
    }
    private String resolveToken(HttpServletRequest request) {
        // 从请求头中提取 token
        String bearerToken = request.getHeader("Authorization");
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer")) {
            // 去掉令牌前缀
            return bearerToken.replace("Bearer","");
        }
        return null;
    }
}

~~~

### 3. 加入过滤器链

~~~java
@RequiredArgsConstructor
public class TokenConfigurer extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> {
    private final TokenProvider tokenProvider;
    @Override
    public void configure(HttpSecurity http) {
        TokenFilter customFilter = new TokenFilter(tokenProvider);
        http.addFilterBefore(customFilter, UsernamePasswordAuthenticationFilter.class);
    }
}
~~~

### 4. 配置 Spring Secutity

~~~java
@Configuration
@EnableWebSecurity //启用Web安全功能
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private final TokenProvider tokenProvider;
    @Bean
    public PasswordEncoder passwordEncoder(){
        // 使用BCrypt加密密码
        return new BCryptPasswordEncoder();
    }
    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.authorizeRequests().antMatchers("/login").permitAll()
            							// 对登录接口放行
            							.anyRequest().authenticated()
                					.and()
            						.apply(securityConfigurerAdapter());
    }
    private TokenConfigurer securityConfigurerAdapter() {
        return new TokenConfigurer(tokenProvider);
    }
}
~~~

### 5. 准备基本业务操作用户类

~~~java
@Repository
public class UserInfo {
    // 基本用户信息类
    private String username;
    private String password;
    private String role;
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    public String getRole() {
        return role;
    }
    public void setRole(String role) {
        this.role = role;
    }
}
~~~

~~~java
@Service
public class UserInfoService {
    public UserInfo getUserInfo(String username){
        // 这里就不操作数据库，简单模拟一下
        if (username.equals("admin")){
            UserInfo userInfo = new UserInfo();
            userInfo.setUsername("admin");
            userInfo.setPassword("admin");
            userInfo.setRole("admin");
            return userInfo;
        }
        return null;
    }
}

~~~

### 6. 接管 UserDetailsService

~~~java
@Component
public class CustomUserDetailsService implements UserDetailsService {
    @Autowired
    private UserInfoService userInfoService;
    @Autowired
    private PasswordEncoder passwordEncoder;
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 通过用户名从数据库获取用户信息
        UserInfo userInfo = userInfoService.getUserInfo(username);
        if (userInfo == null) {
            throw new UsernameNotFoundException("用户不存在");
        }
        // 得到用户角色
        String role = userInfo.getRole();
        // 角色集合
        List<GrantedAuthority> authorities = new ArrayList<>();
        // 角色必须以`ROLE_`开头，数据库中没有，则在这里加
        authorities.add(new SimpleGrantedAuthority(role));
        return new User(
                userInfo.getUsername(),
                // 因为数据库是明文，所以这里需加密密码
                passwordEncoder.encode(userInfo.getPassword()),
                authorities
        );
    }
}
~~~



### 7. 对外接口

~~~java
@RestController
@RequiredArgsConstructor
public class HelloController {
    private final TokenProvider tokenProvider;
    private final CustomUserDetailsService customUserDetailsService;
    private final AuthenticationManagerBuilder authenticationManagerBuilder ;
    @GetMapping("/hello")
    public String hello() {
        // 模拟正常业务接口
        return "hello";
    }
    @GetMapping("/login")
    public String login() throws Exception {
        // 模拟登录请求
        UserDetails user = customUserDetailsService.loadUserByUsername("admin");
        UsernamePasswordAuthenticationToken authenticationToken =
                new UsernamePasswordAuthenticationToken(user.getUsername(), "admin");
        Authentication authentication = authenticationManagerBuilder.getObject().authenticate(authenticationToken);
        // 获取登录用户权限
        SecurityContextHolder.getContext().setAuthentication(authentication);
        // 将信息保存到 SecurityContext 中
        String token = tokenProvider.createToken(authentication);
        // 生成令牌
        System.out.println("生成的令牌："+token);
        return token;
    }
}
~~~

## 三、 接口测试

这里我们是要 **postman** 来作为我们的接口测试工具

### 测试 `http://localhost:8080/login`

![image-20200621202750125](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/G2Eobwj7r8iWIVh.png)

**控制台打印**

![image-20200621202819830](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/oHBXVkcgL2JhReG.png)

### 测试 `http://localhost:8080/hello`

**将token加入请求头**

![image-20200621203004469](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/16fK2rtTwP9XmHx.png)

**控制台打印**

![image-20200621203028591](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/WHEberqmzZtJ2Oi.png)