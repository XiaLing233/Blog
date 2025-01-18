---
title: 同济统一身份认证登录 1 系统
date: 2025-01-18
updated: 2025-01-18
permalink: 2025/01/tongji-login/
categories: Cryptography
---

## 需求

为 1 系统的通知公告内容存档。参见 [Github 项目](https://github.com/XiaLing233/fetch-1-dot-tongji)。然而，请求内容需要 `Cookies`，列表如下

```json
{
    "JSESSIONID": "xxx",
    "sessionid": "yyy",
}
```

否则会提示 `session` 不存在。那么，如何获取这两个 `Cookies` 就是我们这篇文章的主线。

## 过程

### 访问: `https://1.tongji.edu.cn`

首先，访问 `https://1.tongji.edu.cn`，因为之前没有登录过，因此会被重定向到登录界面 `https://iam.tongji.edu.cn`。

然而，这其中更细节的过程是什么？

使用 `curl` 进行测试。

```bash
# -i 的意思是展示请求头
curl -i https://1.tongji.edu.cn/ssologin
# 得到
HTTP/1.1 200 OK
Date: Sat, 18 Jan 2025 05:50:45 GMT
Content-Type: text/html
Content-Length: 1104
# 下略
```

居然没有被重定向，而返回的是 `200 OK`。推测：从 `1.tongji.edu.cn` 重定向到其他网站应该是通过 `js` 实现的。

在浏览器中进一步排查，使用的方法比较原始。在 `Console` 中输入 `debugger`，会逐点暂停，可以观察到每一步的变化。

在 `app.23886001cadcb91ff1bc.js` 中，存在

```js
if (o)
// xxx
else
    "/ssologin" === e.path || "/locallogin" === e.path || "/accountFreeze" === e.path || "/tip" === e.path ? a() : a(3 === c ? {
        path: "/locallogin"
    } : {
        path: "/ssologin"
    })
```

执行到此，地址栏会被替换为 `https://1.tongji.edu.cn/ssologin`。

而后可能会进行是否登录的验证。如果没有登录过，在这一函数

```js
created: function() {
    var e = this.$loading({
        fullscreen: !0
    })
        , t = {
        uid: this.$route.query.uid,
        token: this.$route.query.token,
        ts: this.$route.query.ts
    };
    t.uid && t.token ? this.init(t, e) : (this.$store.commit("clear_login_info"),
    window.location.href = this.$store.state.ssourl) // 这一步进行跳转
},
```

跳转到：`https://1.tongji.edu.cn/api/ssoservice/system/loginIn`

### 跳转到 `https://1.tongji.edu.cn/api/ssoservice/system/loginIn`

方法是 `GET`，返回的非平凡的内容有：

* Cookie: `JESSIONID`
* 返回头中的 `Location`

注意，`1.tongji` 有一个 `JSESSION`，`iam.tongji` 也有一个，两个是不一样的。

因为返回的状态码是 `302`，重定向，因此返回头中的 `Location` 字段存放了下一个要请求的地址。

### 跳转到 `https://iam.tongji.edu.cn/idp/oauth2/authorize?xxx`

方法是 `GET`，返回的非平凡内容有：

* Cookie: `_idp_authn_lc_key`
* Cookie: `SESSION`
* 返回头中的 `Location`

### 跳转到 `https://iam.tongji.edu.cn:443/idp/AuthnEngine?xxx`

方法是 `GET`，返回的非平凡内容有：

* 返回头中的 `Location`

### 跳转到 `https://iam.tongji.edu.cn:443/idp/authcenter/ActionAuthChain?xxx`

方法是 `GET`，返回的非平凡内容有：

* 返回头中的 `Location`

### 跳转到 `https://iam.tongji.edu.cn/idp/authcenter/ActionAuthChain?xxx`

方法是 `GET`，返回 `200 OK`，不需要额外处理。等待输入。

### 稍适休息

前面的步骤都可以用一个 `session` 搞定，因为 `session` 会自动搞定重定向和 `Cookie` 的事情。换句话说，`Cookie` 不用显式写在请求头中。

目前为止，我们获得了什么呢？

* `JSESSIONID`
* `SESSION`
* `_idp_authn_lc_key`

可以回顾一下这些 `Cookie` 都是在哪步获得的。

接下来，需要进行登录操作。如果在浏览器中点击登录，页面就会跳转，能获取到的第一个页面是向 `AuthnEngine?` 发送的请求。然而要是真的向它第一次发送请求，就会有问题：获取不到后续必要的 `Cookie`，换言之，就是验证失败了。为什么呢？

通过一步步排查 `js`，发现在点击按钮后，在 `secondAuth.js?date=202211232022` 中会有这样一个函数发送网络请求：

```js
function doSubmit(usernameId) {
    var authenTypes = $("#authenTypes").val();
    var spAuthentication = $("#spAuthentication").val();
    var authMethodIDs = $("#authMethodIDs").val();
    //var username = $("#"+usernameId).val();

    /*if (typeof(passwordId) != "undefined") {
        $("#" + passwordId).val(encryptByDES($("#" + passwordId).val()));
    }*/
    //debugger;
    var passmm=""; 
    var dataes=$('#' + tabFromId).serialize(); // 储存了表格信息
    if (typeof(passwordId) != "undefined" && $("#" + passwordId).val().length<100) {
        var pass=$("#" + passwordId).val();
        passmm=encryptByRSA(pass); // RSA 加密并 base64 编码后的密码
        pass=encodeURIComponent(pass);
        dataes=dataes.replace('j_password='+pass,'j_password='+passmm); // 把明文密码进行替换

    }
    // basePath 是 /idp
    var urlPath = basePath + "/authcenter/ActionAuthChain?authnLcKey="+$("#authnLcKey").val();
    var view = "";

    $('.loginBt').each(function() {
        $(this).attr('disabled', 'disabled');
    })

    $.ajax({ // 发送请求，方法是 POST，数据是 dataes
        url: urlPath,
        type: "POST",
        cache: false,
        async: true,
        data: dataes,
        success: function (data) {
            console.log("ActionAuthChain", JSON.stringify(data));
            $("#loginButton").html(login1);
            view = data.view;
            if (!data.loginFailed || data.loginFailed == "false") {
                //覆盖掉 明文密码串
                if (typeof(passwordId) != "undefined") {
                    $("#" + passwordId).val(passmm);
                }

                setTimeout(function(){
                    $("#loginButton").removeAttr("disabled");
                },1500);
                form_submit();
                return;
            }
            // $("#loginButton").removeAttr("disabled");
            $('.loginBt').each(function() {
                $(this).removeAttr('disabled');
            })

            /* if (typeof(passwordId) != "undefined") {
                 $("#" + passwordId).val(decryptByDESModeEBC($("#" + passwordId).val()));
            }*/
            $("#zhezhao").hide();
            //show view base view

            // 下略...
        },
    });
}
```

### 向 `https://iam.tongji.edu.cn/idp/authcenter/ActionAuthChain?xxx` 发送请求

具体发送什么，上面的代码块注释已经说的很清楚了。要补充的一点是，需要设置好 `Content-type` 和 `Content-Length`，不然不会得到 `Set-Cookie` 字段。

如果登录成功，会返回：

```json
{
    "loginFailed":"false"
}
```

如果登录失败，会返回一个 `HTML` 格式内容，提示要加强认证，也就是手机号接收验证码；或者是登录失败，还有 `%d` 次尝试机会。要是触发了加强认证，就不太好办，暂时没有想好用什么办法。

### 向 `https://iam.tongji.edu.cn/idp/AuthnEngine?xxx` 发送请求

该有的还得有，给这个链接发送内容也是必要的。

发送的内容是一样的，再发送一次，可以得到

* Cookie: `_idp_session`
* Location

### 跳转到 `https://iam.tongji.edu.cn:443/idp/profile/OAUTH2/AuthorizationCode/SSO?xxx`

> 注意需要重新设置一下请求头，不要再带着 `Content-xxx` 了！

方法是 `GET`，可以得到

* Cookie: `_idp_authn_lc_key` 是不是有些眼熟？
* Location

### 跳转到 `https://1.tongji.edu.cn/api/ssoservice/system/loginIn?xxx`

> 终于到了 1 系统的地界了！

我们已经有 `JSESSIONID` 了。跳转到的这个链接是不是很眼熟（看看第二个 section）？由于之前获取到的 `JSESSIONID` 并没有过期，所以我们可以使用原 `Cookie` 来访问。

当前链接会返回一个 `302`，给我们重定向到下面的链接。

### 跳转到 `http://1.tongji.edu.cn/ssologin?token=xxx&uid=yyy&ts=timestamp`

和前面跳转到 `iam` 不同，这次我们跳转到了 1 系统，这个链接同时包含了

```json
{
    "token": "xxx",
    "uid":  "xxx-yyy-zzzz",
    "ts": "1737777777"
}
```

这三个东西很重要，后续请求的时候需要它，来获得 `sessionid`。

结束了。当前链接会返回一个 `200 OK`，然而，我们还缺少一个 `sessionid` 没有获得呢！

注意到 `Network` 中有一个向 `https://1.tongji.edu.cn/api/sessionservice/session/login` 发送的 `POST` 请求，设置了这一 `Cookie`。

### 向 `https://1.tongji.edu.cn/api/sessionservice/session/login` 发送请求

发送的是 `POST` 请求，发送的内容就是刚刚提到的三元组。

返回的非平凡内容有：

* Cookie: `sessionid`

## 结束

到此为止，我们获得了全部需要的 `Cookies`，完结撒花。

至于说如何进行内容的加密、编码，以及如何模拟请求，会有的。
