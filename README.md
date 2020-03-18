### samesite本地测试问题总结：

#### 准备
1. 浏览器SameSite by default cookies开启enabled
2. 使用`http-server`启动https服务，本地修改host，从而确保可以生成两个跨域https服务
a服务本地地址：`https://mytest.info:8080`（自行在host中改为自己想要的域名）
b服务本地地址：`https://172.30.32.222:8081/`（因本地IP不同会有差异）

#### 制造页面
a页面内容写入cookie
```
setCookie('cooki1', 'withoutSameSite', '')
setCookie('cooki2', 'samaSiteNone', 'SameSite=none')
setCookie('cooki3', 'samaSiteStrict', 'SameSite=Strict')
setCookie('cooki4', 'samaSiteLax', 'SameSite=Lax')
```
b页面内容：
```
# 0.自身存储cooki
      // 情况一：不设置
      document.cookie="b-cookie=b; " + expires;
      document.cookie="b-cookie1=b1; SameSite=none;" + expires;
      // console.log(document.cookie); // b-cookie=b;b-cookie1=b1;

      // 情况二：设置 SameSite
      document.cookie="b-cookie2=b2; SameSite=lax; " + expires;
      document.cookie="b-cookie3=b3; SameSite=Strict; " + expires;

# 1.内嵌a页面链接
<a href="https://mytest.info.com:8080">link</a>

# 2.内嵌a页面图片
 <img src="https://mytest.info.com:8080" alt="">

# 3.自身发起请求
fetch('/api/request/...')
        .then(res => res.json())
        .then(json => console.log(json))
```

#### 启动双服务，使用不同域名后，分场景测试
1. 直接访问b页面，点击内嵌的a链接发送cookie如下，在log存留的情况下，可以看到请求头:
```
Cookie: cooki2=samaSiteNone
```
相当于跨域请求a域，但是cookie只发送了设置为`none`的cookie,其他不会获取

2.直接访问b页面，查看内嵌的图片获得的cookie信息
```
Cookie: cooki2=samaSiteNone
```

3.直接访问a页面，iframe内嵌有b页面，在b页面中无论是从`document.cookie`,还是直接发送请求，能拿到的cookie只有设置为 None的b-cookie：
```
Cookie: b-cookie1=b1
```

由此看来，所有默认开启 samesite的站点想要跨域支持携带cookie,都需要设置samesite=none,同时根据浏览器报错，后续都需要开启Secure,也就是需要https支持。
> A cookie associated with a resource at http://mytest.info.com/ was set with `SameSite=None` but without `Secure`. A future release of Chrome will only deliver cookies marked `SameSite=None` if they are also marked `Secure`. You can review cookies in developer tools under Application>Storage>Cookies and see more details at https://www.chromestatus.com/feature/5633521622188032.