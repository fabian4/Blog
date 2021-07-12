---
title: 基于SpringSecurity 实现的完善的登录鉴权系统
date: 2020/9/20
description: 基于SpringSecurity 实现的完善的登录鉴权系统
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/v3UzwM6HJIrk8ib.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/v3UzwM6HJIrk8ib.jpg'
categories:
  - Spring
tags:
  - SpringSecurity
abbrlink: 44020
---

# 基于SpringSecurity 实现的完善的登录鉴权系统

最近接触到一套完善的前后端分离的后台管理系统，这里就其登录鉴权部分做一个简单的梳理。[项目地址](https://github.com/elunez/eladmin)。

模块涉及：**SpringSecurity**、**JWT**、**Redis**



## 一、整体登录鉴权流程

### 用户登录

![](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/6TdXaSWcvoh5fRU.png)

### 接口鉴权

![](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/gY4jBAHiP3sLJzh.png)





## 二、Controller的处理和登录参数的封装

### AuthorizationController

```java
@AnonymousAccess
//自定义的匿名放行注解
@PostMapping(value = "/login")
public ResponseEntity<Object> login(@Validated @RequestBody AuthUserDto authUser, HttpServletRequest request) throws Exception {
    // 这里将登录参数封装成一个实体类 AuthUserDto
    
    String password = RsaUtils.decryptByPrivateKey(RsaProperties.privateKey, authUser.getPassword());
    // 对前端传过来的密文密码进行私钥解密
    
    String code = (String) redisUtils.get(authUser.getUuid());
    redisUtils.del(authUser.getUuid());
    if (StringUtils.isBlank(code)) {
        throw new BadRequestException("验证码不存在或已过期");
    }
    if (StringUtils.isBlank(authUser.getCode()) || !authUser.getCode().equalsIgnoreCase(code)) {
        throw new BadRequestException("验证码错误");
    }
    // 验证码验证操作，结合redis对验证码进行查询和校验
    
    UsernamePasswordAuthenticationToken authenticationToken =
            new UsernamePasswordAuthenticationToken(authUser.getUsername(), password);
    // 将用户名和明文密码传入Springsecurity框架提供的构造器来生成 authenticationToken
    // 至于用户名和密码的校验则全部由框架来执行

    Authentication authentication = authenticationManagerBuilder.getObject().authenticate(authenticationToken);
    // 根据生成的 authenticationToken 对用户进行鉴权并生成 authentication
    SecurityContextHolder.getContext().setAuthentication(authentication);
    // 将生成的 authentication 存入SpringSecurity的存储域来存储用户信息
    
    String token = tokenProvider.createToken(authentication);
    // 根据 authentication 的信息来生成用户令牌
    
    final JwtUserDto jwtUserDto = (JwtUserDto) authentication.getPrincipal();
    // 将用户信息封装成一个类 jwrUserDto 
    onlineUserService.save(jwtUserDto, token, request);
    // 把用户信息类和用户的token令牌存入redis缓存中
    
    Map<String,Object> authInfo = new HashMap<String,Object>(2){{
        put("token", properties.getTokenStartWith() + token);
        put("user", jwtUserDto);
    }};
    // 封装一个map将信息返回给用户
    
    if(singleLogin){
        // 如果开启了singleLogin模式，就将同一用户之前的登录信息和token令牌从缓存中清除
        // 以确保每一个用户同一时刻只有一个有效令牌
        onlineUserService.checkLoginOnUser(authUser.getUsername(),token);
    }
    return ResponseEntity.ok(authInfo);
}
```



### AuthUserDto

~~~java
@Getter
@Setter
public class AuthUserDto {

    @NotBlank
    private String username;

    @NotBlank
    private String password;

    private String code;

    private String uuid = "";

    @Override
    public String toString() {
        return "{username=" + username  + ", password= ******}";
    }
}
~~~



### JwtUserDto

~~~java
@Getter
@AllArgsConstructor
public class JwtUserDto implements UserDetails {

    private final UserDto user;

    private final List<Long> dataScopes;

    @JsonIgnore
    private final List<GrantedAuthority> authorities;

    public Set<String> getRoles() {
        return authorities.stream().map(GrantedAuthority::getAuthority).collect(Collectors.toSet());
    }

    @Override
    @JsonIgnore
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    @JsonIgnore
    public String getUsername() {
        return user.getUsername();
    }

    @JsonIgnore
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @JsonIgnore
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @JsonIgnore
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    @JsonIgnore
    public boolean isEnabled() {
        return user.getEnabled();
    }
}
~~~



## 三、继承接口重写校验方法

### UserDetailsServiceImpl

~~~java
@RequiredArgsConstructor
@Service("userDetailsService")
@Transactional(propagation = Propagation.SUPPORTS, readOnly = true, rollbackFor = Exception.class)
public class UserDetailsServiceImpl implements UserDetailsService {

    private final UserService userService;
    private final RoleService roleService;
    private final DataService dataService;

    @Override
    public JwtUserDto loadUserByUsername(String username) {
        // 重写此方法，根据用户名返回实体的用户类交给框架进行登录信息比对
        UserDto user;
        try {
            user = userService.findByName(username);
            // 自定义的方法，从数据库根据用户名查询信息
        } catch (EntityNotFoundException e) {
            throw new UsernameNotFoundException("", e);
        }
        if (user == null) {
            throw new UsernameNotFoundException("");
        } else {
            if (!user.getEnabled()) {
                throw new BadRequestException("账号未激活");
            }
            return new JwtUserDto(
                    user,
                    dataService.getDeptIds(user),
                    roleService.mapToGrantedAuthorities(user)
                // 这里根据用户信息来新建一个类返回给框架进行比对
            );
        }
    }
}
~~~



## 四、在线用户的信息管理

### OnlineUserService

~~~java
	/**
     * 保存在线用户信息
     * @param jwtUserDto /
     * @param token /
     * @param request /
     */
    public void save(JwtUserDto jwtUserDto, String token, HttpServletRequest request){
        String dept = jwtUserDto.getUser().getDept().getName();
        String ip = StringUtils.getIp(request);
        String browser = StringUtils.getBrowser(request);
        String address = StringUtils.getCityInfo(ip);
        // 获取用户登录的详细信息
        OnlineUserDto onlineUserDto = null;
        try {
            onlineUserDto = new OnlineUserDto(jwtUserDto.getUsername(), jwtUserDto.getUser().getNickName(), dept, browser , ip, address, EncryptUtils.desEncrypt(token), new Date());
        } catch (Exception e) {
            e.printStackTrace();
        }
        // 存入系统redis缓存中，以token为键值
        redisUtils.set(properties.getOnlineKey() + token, onlineUserDto, properties.getTokenValidityInSeconds()/1000);
    }

    
    /**
     * 踢出用户
     * @param key /
     */
    public void kickOut(String key){
        // 从缓存中删除
        key = properties.getOnlineKey() + key;
        redisUtils.del(key);
    }

	/**
     * 查询全部数据
     * @param filter /
     * @return /
     */
    public List<OnlineUserDto> getAll(String filter){
        List<String> keys = redisUtils.scan(properties.getOnlineKey() + "*");
        // 扫描redis中的所有键值
        Collections.reverse(keys);
        List<OnlineUserDto> onlineUserDtos = new ArrayList<>();
        for (String key : keys) {
            OnlineUserDto onlineUserDto = (OnlineUserDto) redisUtils.get(key);
            if(StringUtils.isNotBlank(filter)){
                if(onlineUserDto.toString().contains(filter)){
                    onlineUserDtos.add(onlineUserDto);
                }
            } else {
                onlineUserDtos.add(onlineUserDto);
            }
        }
        // 将对应的对象取除存入链表，并以他们的登录时间排序
        onlineUserDtos.sort((o1, o2) -> o2.getLoginTime().compareTo(o1.getLoginTime()));
        return onlineUserDtos;
    }


    /**
     * 检测用户是否在之前已经登录，已经登录踢下线
     * @param userName 用户名
     */
    public void checkLoginOnUser(String userName, String igoreToken){
        List<OnlineUserDto> onlineUserDtos = getAll(userName);
        if(onlineUserDtos ==null || onlineUserDtos.isEmpty()){
            return;
        }
        for(OnlineUserDto onlineUserDto : onlineUserDtos){
            if(onlineUserDto.getUserName().equals(userName)){
                // 将同一用户名的其他用户的信息从缓存中清除
                try {
                    String token =EncryptUtils.desDecrypt(onlineUserDto.getKey());
                    if(StringUtils.isNotBlank(igoreToken)&&!igoreToken.equals(token)){
                        this.kickOut(token);
                    }else if(StringUtils.isBlank(igoreToken)){
                        this.kickOut(token);
                    }
                } catch (Exception e) {
                    log.error("checkUser is error",e);
                }
            }
        }
    }

~~~



### OnlineUserDto

~~~java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class OnlineUserDto {

    /**
     * 用户名
     */
    private String userName;

    /**
     * 昵称
     */
    private String nickName;

    /**
     * 岗位
     */
    private String dept;

    /**
     * 浏览器
     */
    private String browser;

    /**
     * IP
     */
    private String ip;

    /**
     * 地址
     */
    private String address;

    /**
     * token
     */
    private String key;

    /**
     * 登录时间
     */
    private Date loginTime;
    
}
~~~



## 五、Token的发放

### TokenProvider

```java
@Slf4j
@Component
@RequiredArgsConstructor
public class TokenProvider implements InitializingBean {

   private final SecurityProperties properties;
   private final RedisUtils redisUtils;
   private static final String AUTHORITIES_KEY = "auth";
   private Key key;

   @Override
   public void afterPropertiesSet() {
      byte[] keyBytes = Decoders.BASE64.decode(properties.getBase64Secret());
      this.key = Keys.hmacShaKeyFor(keyBytes);
   }

     /**
     * 根据用户的鉴权信息来生成token
     */
   public String createToken(Authentication authentication) {
      String authorities = authentication.getAuthorities().stream()
         .map(GrantedAuthority::getAuthority)
         .collect(Collectors.joining(","));

      return Jwts.builder()
              .setSubject(authentication.getName())
              .claim(AUTHORITIES_KEY, authorities)
              .signWith(key, SignatureAlgorithm.HS512)
              // 加入ID确保生成的 Token 都不一致
              .setId(IdUtil.simpleUUID())
              .compact();
   }
    
     /**
     * 根据token解析出用户的鉴权信息
     */
   Authentication getAuthentication(String token) {
      Claims claims = Jwts.parserBuilder()
         .setSigningKey(key)
         .build()
         .parseClaimsJws(token)
         .getBody();
      Object authoritiesStr = claims.get(AUTHORITIES_KEY);
      Collection<? extends GrantedAuthority> authorities =
              ObjectUtil.isNotEmpty(authoritiesStr) ?
       Arrays.stream(authoritiesStr.toString().split(","))
            .map(SimpleGrantedAuthority::new)
            .collect(Collectors.toList()) : Collections.emptyList();

      User principal = new User(claims.getSubject(), "", authorities);

      return new UsernamePasswordAuthenticationToken(principal, token, authorities);
   }

   /**
    * @param token 需要检查的token
    */
   public void checkRenewal(String token){
      // 判断是否续期token,计算token的过期时间
      long time = redisUtils.getExpire(properties.getOnlineKey() + token) * 1000;
      Date expireDate = DateUtil.offset(new Date(), DateField.MILLISECOND, (int) time);
      // 判断当前时间与过期时间的时间差
      long differ = expireDate.getTime() - System.currentTimeMillis();
      // 如果在续期检查的范围内，则续期
      if(differ <= properties.getDetect()){
         long renew = time + properties.getRenew();
         redisUtils.expire(properties.getOnlineKey() + token, renew, TimeUnit.MILLISECONDS);
      }
   }
    
     /**
     * 从请求头中获取token
     */
   public String getToken(HttpServletRequest request){
      final String requestHeader = request.getHeader(properties.getHeader());
      if (requestHeader != null && requestHeader.startsWith(properties.getTokenStartWith())) {
         return requestHeader.substring(7);
      }
      return null;
   }
}
```



## 六、Token过滤器鉴权

### TokenFilter

~~~java
@Slf4j
@RequiredArgsConstructor
public class TokenFilter extends GenericFilterBean {

   private final TokenProvider tokenProvider;

   @Override
   public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
      throws IOException, ServletException {
      HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
      String token = resolveToken(httpServletRequest);
      // 从请求头中来获取token
      if(StrUtil.isNotBlank(token)){
         OnlineUserDto onlineUserDto = null;
         SecurityProperties properties = SpringContextHolder.getBean(SecurityProperties.class);
         try {
            OnlineUserService onlineUserService = SpringContextHolder.getBean(OnlineUserService.class);
            onlineUserDto = onlineUserService.getOne(properties.getOnlineKey() + token);
             // 从redis中查询用户的权限信息
         } catch (ExpiredJwtException e) {
            log.error(e.getMessage());
         }
         if (onlineUserDto != null && StringUtils.hasText(token)) {
            Authentication authentication = tokenProvider.getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(authentication);
            // 将用户的个人信息放到SpringSecurity的存储域中，来允许系统对其权限的验证
            // Token 续期
            tokenProvider.checkRenewal(token);
         }
      }
      filterChain.doFilter(servletRequest, servletResponse);
   }

   private String resolveToken(HttpServletRequest request) {
      SecurityProperties properties = SpringContextHolder.getBean(SecurityProperties.class);
      String bearerToken = request.getHeader(properties.getHeader());
      if (StringUtils.hasText(bearerToken) && bearerToken.startsWith(properties.getTokenStartWith())) {
         // 去掉令牌前缀
         return bearerToken.replace(properties.getTokenStartWith(),"");
      }
      return null;
   }
}

~~~



## 七、配置匿名访问注解和Token过滤器链

### 匿名访问注解 @AnonymousAccess

~~~java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AnonymousAccess {

}
~~~



### 配置token过滤器链 TokenConfigurer

~~~java
@RequiredArgsConstructor
public class TokenConfigurer extends SecurityConfigurerAdapter<DefaultSecurityFilterChain, HttpSecurity> {

    private final TokenProvider tokenProvider;

    @Override
    public void configure(HttpSecurity http) {
        TokenFilter customFilter = new TokenFilter(tokenProvider);
        http.addFilterBefore(customFilter, UsernamePasswordAuthenticationFilter.class);
        // 这里要注意过滤器链添加的位置
    }
}
~~~



## 八、SpringSecurity配置

~~~java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final TokenProvider tokenProvider;
    private final CorsFilter corsFilter;
    private final JwtAuthenticationEntryPoint authenticationErrorHandler;
    private final JwtAccessDeniedHandler jwtAccessDeniedHandler;
    private final ApplicationContext applicationContext;

    @Bean
    GrantedAuthorityDefaults grantedAuthorityDefaults() {
        // 去除 ROLE_ 前缀
        return new GrantedAuthorityDefaults("");
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        // 密码加密方式
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        // 搜寻匿名标记 url： @AnonymousAccess
        Map<RequestMappingInfo, HandlerMethod> handlerMethodMap = applicationContext.getBean(RequestMappingHandlerMapping.class).getHandlerMethods();
        Set<String> anonymousUrls = new HashSet<>();
        for (Map.Entry<RequestMappingInfo, HandlerMethod> infoEntry : handlerMethodMap.entrySet()) {
            HandlerMethod handlerMethod = infoEntry.getValue();
            AnonymousAccess anonymousAccess = handlerMethod.getMethodAnnotation(AnonymousAccess.class);
            if (null != anonymousAccess) {
                anonymousUrls.addAll(infoEntry.getKey().getPatternsCondition().getPatterns());
            }
        }
        httpSecurity
                // 禁用 CSRF
                .csrf().disable()
                .addFilterBefore(corsFilter, UsernamePasswordAuthenticationFilter.class)
                // 授权异常
                .exceptionHandling()
                .authenticationEntryPoint(authenticationErrorHandler)
                .accessDeniedHandler(jwtAccessDeniedHandler)

                // 防止iframe 造成跨域
                .and()
                .headers()
                .frameOptions()
                .disable()

                // 不创建会话
                .and()
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)

                .and()
                .authorizeRequests()
                // 静态资源等等
                .antMatchers(
                        HttpMethod.GET,
                        "/*.html",
                        "/**/*.html",
                        "/**/*.css",
                        "/**/*.js",
                        "/webSocket/**"
                ).permitAll()
                // swagger 文档
                .antMatchers("/swagger-ui.html").permitAll()
                .antMatchers("/swagger-resources/**").permitAll()
                .antMatchers("/webjars/**").permitAll()
                .antMatchers("/*/api-docs").permitAll()
                // 文件
                .antMatchers("/avatar/**").permitAll()
                .antMatchers("/file/**").permitAll()
                // 阿里巴巴 druid
                .antMatchers("/druid/**").permitAll()
                // 放行OPTIONS请求
                .antMatchers(HttpMethod.OPTIONS, "/**").permitAll()
                // 自定义匿名访问所有url放行 ： 允许匿名和带权限以及登录用户访问
                .antMatchers(anonymousUrls.toArray(new String[0])).permitAll()
                // 所有请求都需要认证
                .anyRequest().authenticated()
                .and().apply(securityConfigurerAdapter());
    }

    private TokenConfigurer securityConfigurerAdapter() {
        return new TokenConfigurer(tokenProvider);
    }
}
~~~



> 到此就完成了整个系统的登录模块的配置，实现了在SpringSecurity框架基础上的身份的鉴别，权限的授予，Token的发放和利用过滤器链实现Token的截取和表述，融合了Redis可以达到对所有在线用户的管理功能。达到了不再使用Session的用户无状态登录功能。