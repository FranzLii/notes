# Vue 路由守卫+Token实现前后端分离身份验证

#### 1.JWT前端前端以及后端配置

```vue
...
```

#### 2.VUE配置路由守卫

配置于`main.js`中

```javascript
router.beforeEach((to,from,next)=>{
  let token = localStorage.getItem("token");
  if(to.path == '/login' || token ){
    next();
    console.log(token);
  }else{
    alert('您还没有登录，请先登录');
    next('/login');
  }
})
```

**防止用户通过手动输地址进入未授权界面**

#### 3.配置请求拦截器

```javascript
axios.interceptors.request.use(
  config =>{
    let token = localStorage.getItem("token");
    if(token){
      config.headers.Authorization = token;
    }
    return config;
  },
  error => {
    return Promise.reject(error)
  }
)
```

**使得前端每次请求都在Header的`Authorization`字段中添加token，使后端判断请求是否合法**

#### 4.配置相应拦截器

```javasc
//在token失效之后判断并跳转的登录页面，清除本地的token
```



