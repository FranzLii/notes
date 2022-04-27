# Spring Security + jwt + redis认证授权流程  

## 1.Spring Seurity配置  

配置登录跨域，以及开放登录接口匿名访问权限

```java

/**
 * @author maisann
 */

@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class MySecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    JwtTokenFilter jwtTokenFilter;

    @Autowired
    private AuthenticationEntryPoint authenticationEntryPoint;



    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder(){
        return new BCryptPasswordEncoder();//配置密码加密器
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //关闭csrf
                .csrf().disable()
                //不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                // 对于登录接口 允许匿名访问
                .antMatchers("/hello/login").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();

        http.exceptionHandling().authenticationEntryPoint(authenticationEntryPoint);
        http.addFilterBefore(jwtTokenFilter, UsernamePasswordAuthenticationFilter.class);
    }

    @Override
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();//将AuthenticationManager交给Spring管理，方便后面UserService调用进行认证
    }
}

```

## 2.编写UserDetail  

方便后面UserDetailService进行调用，封装用户名以及密码和权限

```java

@Data
@NoArgsConstructor
public class LoginUser implements UserDetails {

    private User user;

    private List<String> permissions;

    @JSONField(serialize = false)
    private List<GrantedAuthority> authorities;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        if(Objects.isNull(authorities)){
            authorities =  permissions.stream().map(SimpleGrantedAuthority::new).collect(Collectors.toList());
        }
        return authorities;
//        return null;
    }

    public LoginUser(User user,List<String> permissions){
        this.user = user;
        this.permissions = permissions;
    }


    @Override
    public String getPassword() {
        return this.user.getPassword();
    }

    @Override
    public String getUsername() {
        return this.user.getUserName();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}

```

## 3.编写UserDetailService

替换Spring Security中默认的获取用户的Service接口，实现数据库查询用户名密码以及封装权限，将权限交给上一层调用，封装到Authcation中

```java

@Service
@Slf4j
public class LoginDetailServiceImpl implements UserDetailsService {

    @Autowired
    UserService userService;

    @Autowired
    PermissionService permissionService;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        //根据用户名查询用户信息
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(User::getUserName,username);
        User user = userService.getOne(wrapper);
        //如果查询不到数据就通过抛出异常来给出提示
        if(Objects.isNull(user)){
            throw new RuntimeException("用户名或密码错误");
        }
        //根据用户查询权限信息 添加到LoginUser中
        LambdaQueryWrapper<Permission> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(Permission::getUid,user.getId());
        Permission permission = permissionService.getOne(queryWrapper);
        List<String> list = null;

        log.debug(user.toString());

        if(!Objects.isNull(permission)){
            String[] split = permission.getPermission().split(",");
            list = Arrays.asList(split);
        }
        return new LoginUser(user,list);
    }
}

```

## 4.编写UserService实现自定义登录

替换掉默认的UserNameAndPasswordAuthication

```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    private RedisCache redisCache;

    @Override
    public HashMap<String,String> login(User user) {
        
        UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(user.getUserName(),user.getPassword());
        //调用authenticationManager.authenticate实现认证授权,authenticate中会调用DaoAuthenticationProvider
        Authentication authenticate = authenticationManager.authenticate(authenticationToken);
        
        if(Objects.isNull(authenticate)){
            throw new RuntimeException("用户名或密码错误");
        }
        //使用userid生成token
        LoginUser loginUser = (LoginUser) authenticate.getPrincipal();

        String userId = loginUser.getUser().getId().toString();
        String jwt = JwtUtil.createJWT(userId);
        
        //authenticate存入redis
        redisCache.setCacheObject("login:"+userId,loginUser);
        //把token响应给前端
        HashMap<String,String> map = new HashMap<>();
        map.put("token",jwt);
        
        return map;
    }
}

```

## 5.编写JwtTokenFilter实现认证授权

```java
@Component
@Slf4j
public class JwtTokenFilter extends OncePerRequestFilter {


    @Autowired
    RedisCache redisCache;


    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        //从请求头中获取Authorization字段
        String token = request.getHeader("Authorization");


        //如果请求头中不存在Authorization字段直接放行,后续会被Spring Security进行拦截，或者进入../login进行认证获取token
        if (!Strings.hasText(token)) {
            filterChain.doFilter(request, response);
            return;
        }


        //解析jwt获取userid
        String userid;
        try {
            Claims claims = JwtUtil.parseJWT(token);
            userid = claims.getSubject();
        } catch (Exception e) {
            e.printStackTrace();
            //抛出异常给Security自带的异常处理filter处理
            throw new RuntimeException("jwt解析失败");
        }

        //从redis中获取用户数据
        LoginUser cacheObject = redisCache.getCacheObject("login:"+userid);

        System.out.println(cacheObject);

        if(Objects.isNull(cacheObject)){
            throw new RuntimeException("redis中不存在用户");
        }

      	//将登录的用户和用户的权限放入SecurityContextHolder中，方便后续框架进行认真
        SecurityContextHolder.getContext().setAuthentication(new UsernamePasswordAuthenticationToken(cacheObject,null,cacheObject.getAuthorities() ));

        filterChain.doFilter(request,response);
    }
}

```

