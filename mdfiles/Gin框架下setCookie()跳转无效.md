### gin框架下通过post请求设置登录cookie后跳转其他页面无法收到cookie

通过POST请求提交登录表单,验证后设置cookie

之后c.GET("/home")中会通过c.Cookie("login_cookie")来判断页面是重定向到登录界面还是显示home界面

#### 出现问题

- 页面无法跳转

  chrome内核过80后第三发cookie限制 ,可通过浏览器设置更改

  chrome://flags/   把这句复制到浏览器回车
  SameSite by default cookies
  Cookies without SameSite must be secure
  找到上面这两两项设置成 Disable即可

  重启浏览器

- 页面依然无法跳转,但通过postman工具检测接口正常

  发现跳转状态码为301,永久定向,浏览器第一次缓存后会一直重定向到login界面

  所以把跳转代码更改为302临时跳转,运行正常

  

### 需要注意的是,在post请求中,会因为浏览器安全限制cookie导致cookie传递出现问题.同时,301会导致代码运行更改后运行出现相同的问题,但清除浏览器缓存后可以偶尔正常运行,页面跳转需要注意使用

