---
title: 唤端
top: false
cover: false
toc: false
date: 2020-06-22 17:18:13
password:
tags: h5，mobile
---
# 唤端
app之间的跳转简称唤端。

# 方案
## URL Scheme
scheme 标示资源访问的方式。常见的scheme有http/https/ftp等。而在IOS/Andriod中，scheme相当于用哪个app打开URL。

```js
  <a href="tel://18511111111" /> // IOS中唤起拨号程序
```

#### Scheme 注册
需要在app里事先向系统注册自己的Scheme，才能让app通过Scheme方式唤起。

**系统只负责根据Scheme唤起对应的app，至于打开app之后的行为，需要app解析URL参数作出相应处理**

#### 适用性
不适应Android的Chrome及基于Chrome内核浏览器直接访问

#### 优点
兼容性好

#### 缺点
- 无法准确判断是否唤起成功。
- 如果用户没有安装对应的app，那么尝试跳转后在浏览器中会没有任何反应。甚至在一些webview里还会跳到一个类似 **无法打开xxxx** 这样的错误页面或者错误弹窗。
- 在很多浏览器和webview中会有一个弹窗提示是否打开对应app，会导致用户流失。
- 有URL Scheme劫持风险。比如某不知名app也向系统注册了 `xxx://` 这个scheme，唤起流量可能就会被劫持到这个app里。
- 很容易被屏蔽，app很容易拦截掉通过URL Scheme发起的跳转。




## Intent
谷歌定制版的URL Scheme。本质与URL Scheme区别不大，[构造链接](https://developer.chrome.com/multidevice/android/intents?spm=ata.13261165.0.0.d54f54a3toRfWr)方式不同。

#### 适用性
仅Android的Chrome及基于Chrome内核浏览器。




## Universal Link
IOS 9 中引入的功能，可以直接通过https协议的链接来打开app，如果没有安装则打开对应h5页面。
```js
    <a href="https://xxxx" />
```

#### [注册](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html?spm=ata.13261165.0.0.6a9072d111Nj92)
- app 中注册自己要支持的域名
- 在自己域名的根目录下配置一个 apple-app-site-associatio 文件。

#### 适用性
只在IOS 9 以上

#### 优点
- 唤端时没有弹窗提示，可减少一部分用户流失
- 对于没有安装应用的用户，点击链接就会直接打开对应的页面，可引导到中转页，一定程度解决URL Scheme无法判断唤端失败问题。

#### 缺点
- 只能在IOS 上用
- 只能由用户主动触发。比如浏览器扫码打开页面，没有办法由页面直接唤起app，需要用户手动点击页面按钮才能唤起。




## [App Links](https://developer.android.com/training/app-links#add-app-links)
安卓版的Universal Link。




## wx.launchApplication
微信JSSDK隐藏API。

#### 适用性
IOS，Android 微信浏览器环境。仅限腾讯系50个应用白名单，微信版本大于6.5.16

**在微信环境下，URL Scheme会被拦截，无法唤起应用。Universal Link/App Links均无效。**

## H5唤端兼容
#### iframe
```js
    var iframe = document.createElement('iframe');
    iframe.frameborder = '0';
    iframe.style.cssText = 'display: none;border: none;width: 0;height: 0';
    document.body.appendChild(iframe);
    iframe.src = 'zhihu://xxx'
```
即使没有安装app，当前页面也不会跳转到错误的页面


#### a标签
IOS 9 以上safari中，不支持iframe唤起，可以用a标签，并模拟点击事件
```js
    var a = document.createElement('a');
    a.setAttribute('href', url);
    a.style.display = 'none';
    document.body.appendChild(a);
    
    var e = document.createEvent('HEMLEvents');
    e.initEvent('click', false, false);
    a.dispatchEvent(e);
```

#### window.location
对于Intent和Universal Link，App Links可以直接设置window.location.href方式跳转，即使没有安装app也不会跳转到错误页面。


# 示例
```js
    if (isWeixin) {
        if (weixinVersion >= 7.0.12) {
            wx-open-launch-app // https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_Open_Tag.html
        } else if (weixinVersion >= 6.5.16 && HOST_WHITE) {
            wx.launchApplication
        } else {
            // 引导浏览器打开页面
        }
    } else if (isIOS) {
        if (isIOSVersion >= 9) {
            window.location.href = Universal Link
        } else {
            iframe.src = URL Scheme
        }
    } else if (isAndroid) {
        if (isChrome) {
            window.location.href = Intent
        } else {
            iframe.src = URL Scheme
        }
    }
```

#### 参考

[揭秘手淘召唤术](https://juejin.im/post/5eeb785251882565a762e50f)


