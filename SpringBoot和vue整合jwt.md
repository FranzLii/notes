# Spring Boot + jwt + vue实现前后端分离实现身份验证

---

###  1.后端配置  

---

#### 1.jwt配置  

---

##### 1.编写jwt配置类

```java
@Component
@ConfigurationProperties(prefix = "jwt")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class JWTconfig {
    private long expire;
    private String secret;
}

```

  

##### 2.配置Application.yml

_因为已经配置`@ConfigurationProperties`注解所以直接在Application.yml中配置jwt属性即可

```yaml
jwt:
  expire: 7200000 #token过期时间
  secret: dfasfdsadghgfaewqteq #token密钥
```

  

#### 2.编写Jwtutils工具类  



```java
@Component
@Data
public class JwtUtils {

    @Autowired
    private  JWTconfig jwtconfig;


    public  DecodedJWT verify(String token) throws Exception{
        try {
            JWTVerifier verifier = JWT.require(Algorithm.HMAC256(jwtconfig.getSecret())).build();
            DecodedJWT jwt = verifier.verify(token);
            return jwt;
        } catch (TokenExpiredException tokenExpiredException){
            throw new Exception(JwtVerifyConst.EXPIRED);
        } catch (SignatureVerificationException signatureVerificationException){
            throw new Exception(JwtVerifyConst.SIGNATURE_VERIFICATION);
        } catch (JWTDecodeException jwtDecodeException){
            throw new Exception(JwtVerifyConst.DECODE_ERROR);
        }
        catch (Exception exception) {
            exception.printStackTrace();
            throw new Exception(JwtVerifyConst.NOT_LOGIN);
        }
    }

    public  String sign(Map<String,String> info) {
        Date date = new Date(System.currentTimeMillis()+jwtconfig.getExpire());
        Algorithm algorithm = Algorithm.HMAC256(jwtconfig.getSecret());
        JWTCreator.Builder builder = JWT.create();
        info.forEach((k,v)->builder.withClaim(k,v));
        return builder.withExpiresAt(date).sign(algorithm);
    }

}
class JwtVerifyConst{
    public static String SUCCESS = "token验证成功";
    public static String EXPIRED = "token已过期";
    public static String SIGNATURE_VERIFICATION = "token签名失败";
    public static String DECODE_ERROR = "token解析失败，请重新登录获取token";
    public static String NOT_LOGIN = "未登录";
}

```

_Spring无法注入被static修饰的属性所以jwtconfig不要为static具体原因如下_

**态变量/类变量不是对象的属性,而是一个类的属性,spring则是基于对象层面上的依赖注入.
而使用静态变量/类变量扩大了静态方法的使用范围.静态方法在spring是不推荐使用的.依赖注入的主要目的,是让容器去产生一个对象的实例,然后在整个生命周期中使用他们,同时也让testing工作更加容易.**



#### 3.配置Interceptor拦截器  

---

##### 1.Interceptor编写  



```java
@Component
public class Tokeninterceptor implements HandlerInterceptor {

    @Autowired
    private JwtUtils jwtUtils;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        if (HttpMethod.OPTIONS.toString().equals(request.getMethod())) {
            response.setStatus(HttpServletResponse.SC_OK);
            return true;
        }
        String token = request.getHeader("Authorization");
        System.out.println(token);
        jwtUtils.verify(token);
        return true;
    }

}

```

**注意：因为ajax请求会发起两次请求，第一次为`OPTION`请求，用来预检，如果该请求不能成功，则不会进行后续请求，所以要放行该请求**

```java
if (HttpMethod.OPTIONS.toString().equals(request.getMethod())) {
            response.setStatus(HttpServletResponse.SC_OK);
            return true;
}
```

##### 2.webconfig配置  



```java
@Configuration
public class WebConfig  implements WebMvcConfigurer{

    @Autowired
    private Tokeninterceptor tokeninterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        WebMvcConfigurer.super.addInterceptors(registry);
        registry.addInterceptor(tokeninterceptor).addPathPatterns("/**");
    }
}
```



#### 4.Controller层编写  

--

```java
@RestController
@CrossOrigin
@RequestMapping("/jwttest")
public class JwtController {

    @Autowired
    JwtUtils jwtUtils;

    @PostMapping()
    public R login(@RequestBody LoginUser loginUser){
        R r = new R();
        r.setFlag(false);
        r.setData("用户名或密码错误");
        System.out.println(loginUser);
        if(loginUser.getUserName().equals("xiaoli") && loginUser.getPassWord().equals("123")){
            Map<String,String> map = new HashMap<>();
            map.put("username", loginUser.getUserName());
            String token =  jwtUtils.sign(map);
            r.setFlag(true);
            r.setData(token);
        }
        return r;
    }

    @GetMapping
    public R getall(){
        return new R(true,"全部数据");
    }
}
```



### 前端编写

![image-20220324170510248](https://gitee.com/ingachin/mdimage/raw/master/image-20220324170510248.png)



#### 1.界面组件  

---

```vue
  <el-row>
    <el-col :span="8" :offset="8">
      <div style="width: 500px">
        <el-form ref="form" :model="loginUser" label-width="80px">
          <el-form-item label="用户名">
            <el-input v-model="loginUser.userName"></el-input>
          </el-form-item>
          <el-form-item label="密码">
            <el-input v-model="loginUser.passWord"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="onSubmit">登录</el-button>
          </el-form-item>
        </el-form>
      </div>
    </el-col>
    <div v-model="test">{{ test }}</div>
    <el-button type="primary" @click="tokentest">token测试</el-button><br>
    <el-button type="primary" @click="removetoken">清除token</el-button>
  </el-row>
```



#### 2.表单提交函数  

---

```javascript
 onSubmit() {
      this.$http.post("http://localhost/jwttest", this.loginUser).then((resp) => {
        localStorage.setItem("token", resp.data.data)
      })
}
```

**将后台验证成功生成的TOKEN存储到本地的localStorage中，之后每一次访问在请求头的`Authorization`字段中添加TOKEN即可通过后台的Interceptor验证**

_添加Authorization字段前_

![image-20220324170948694](https://gitee.com/ingachin/mdimage/raw/master/image-20220324170948694.png)

---

_添加Authorization字段后_

![image-20220324171032507](https://gitee.com/ingachin/mdimage/raw/master/image-20220324171032507.png)



---

_后端返回数据_

![image-20220324171118795](https://gitee.com/ingachin/mdimage/raw/master/image-20220324171118795.png)



#### 3.测试函数如下  



```javascript
tokentest() {
      this.$http.get("http://localhost/jwttest", {
        headers: {
          "Authorization": localStorage.getItem("token")
        }
      }).then((resp) => {
        this.test = resp.data.data;
      })
    },
    removetoken() {
      localStorage.removeItem("token");
    }
  }
```





