# 08 第三方 SDK 集成

## SDK 总览

| 分类 | SDK | 版本 | 用途 |
|------|-----|------|------|
| **播放器** | 火山引擎 TTSDK | 1.48.2.10 | 视频播放/下载(Premium版) |
| **社交** | 微信 SDK | 6.8.0 | 登录/分享/支付 |
| **社交** | 微博 SDK | 13.3.0 | 分享 |
| **分享** | MobSDK ShareSDK | 3.x | 统一分享(微信/微博/QQ) |
| **推送** | 友盟推送 | 6.5.9 | 消息推送 |
| **崩溃** | 腾讯 Bugly | 4.0.4 | 崩溃收集 |
| **统计** | 友盟统计 | 9.5.5 | 用户行为分析 |
| **登录** | YJYZ一键登录 | 4.0.5 | 号码认证登录 |
| **投屏** | 乐播投屏 | 5.x | 屏幕投射 |
| **弹幕** | DanmakuFlameMaster | 0.9.25 | B站开源弹幕引擎 |
| **动画** | SVGAPlayer | 2.6.1 | SVGA矢量动画 |
| **二维码** | ZXingLibrary | 1.9 | 扫码/生成二维码 |
| **存储** | 七牛云 | 8.4.2 | 文件上传 |
| **存储** | 腾讯云 COS | 5.x | 对象存储 |
| **网页** | Jsoup | 1.15.3 | HTML解析 |
| **数据库** | GreenDao | 3.3.0 | 本地数据库ORM |
| **网络** | OkHttp3 | 3.12.13 | HTTP客户端 |
| **网络** | Retrofit2 | 2.6.0 | REST API 框架 |
| **图片** | Glide | 4.12.0 | 图片加载/缓存 |
| **JSON** | Gson | 2.9.0 | JSON序列化 |
| **JSON** | FastJson | 1.x | JSON序列化(live模块) |
| **密钥** | Arkana | N/A | 编译期密钥混淆 |

---

## 1. 火山引擎播放器 (TTSDK)

### 初始化

```
MovieApplication.onCreate() 中延迟初始化:
1. 从 NDK (security.cpp) 获取加密的 AppId
2. SecurityHelper.getKeyStringNative() 获取 AES Key
3. SecurityHelper.getIVStringNative() 获取 AES IV
4. AES 解密得到明文 AppId
5. 配置 VolcPlayerManager:
   - AppId, AppName, AppChannel
   - CacheDir: 外部缓存/video/
   - 日志开关: 仅 Debug
6. 调用 TTSDK.init() 初始化
```

### 播放器功能

| 功能 | 说明 |
|------|------|
| 播放 | 支持 HTTP/HTTPS/本地文件 |
| 清晰度切换 | 多码率流切换 |
| URL 刷新 | AppUrlRefreshFetcher 动态刷新过期URL |
| 预加载 | 预buffer下一集 |
| 倍速 | 0.5x ~ 2.0x |
| 缓存下载 | VolcDownloadManager 加密缓存 |

### 关键类

| 类 | 职责 |
|---|------|
| `VolcPlayerManager` | 播放器初始化管理 |
| `AppUrlRefreshFetcher` | 实现播放URL自动刷新 |
| `VolcDownloadManager` | 视频下载管理 |

---

## 2. 友盟全家桶

### 初始化顺序

```
1. UMConfigure.preInit(appKey, channel)     ← 尽早调用
2. UMConfigure.init(appKey, channel, ...)   ← 同意隐私后
3. PushAgent.register(callback)             ← 注册推送
4. MobClickAgent.setPageCollectionMode(AUTO) ← 自动采集
```

### 配置参数

| 参数 | 值 |
|------|-----|
| AppKey | `5db1093d570df3b928000295` |
| MessageSecret | `a5c8c90f8ec7ed79c09f3bbf5de52f8e` |
| Channel | 从 Manifest `UMENG_CHANNEL` 读取 |

### 推送厂商通道

| 厂商 | SDK | 配置 |
|------|-----|------|
| 华为 | HuaweiPushAgent | AppId在Manifest |
| 小米 | XiaomiPushAgent | AppId/AppKey在代码中 |
| VIVO | VivoPushAgent | AppId/AppKey在代码中 |
| OPPO | OppoPushAgent | AppKey/AppSecret在代码中 |

### 推送处理

```
收到推送 → MyPushIntentService:
  - 收到DeviceToken → 保存到UserInfoConst.umToken
  - 收到消息 → 解析payload.extra中的action:
    ├── action含"detail" + movieId → 跳转影片详情
    ├── action含"web" + url → 打开WebView
    └── 默认 → 打开首页
```

---

## 3. 腾讯 Bugly

### 初始化

```
CrashReport.initCrashReport(context, appId, isDebug)
```

| 参数 | 值 |
|------|-----|
| AppId | `0d026bf359` |
| Debug | BuildConfig.DEBUG |

### 使用场景
- 全局崩溃收集
- ANR监控
- 自定义异常上报

---

## 4. 乐播投屏

### 初始化

```
LeBoService（Android Service）
  → 启动后监听局域网投屏设备
  → 接收 CONNECT_LEBO 事件连接设备
  → 推送视频URL到投屏设备
```

### 关键事件

| 事件 | 说明 |
|------|------|
| `CONNECT_LEBO` | 发起投屏连接 |
| `CALL_SELECT_LELINK` | 用户选择了投屏设备 |
| `LEBO_SHARPNESS_SELECT` | 投屏清晰度选择 |
| `LEBO_LINK_PLAY_PAUSE` | 投屏播放/暂停 |
| `LEBO_VIEW_SHOW` | 投屏占位View |
| `NOTIFY_SAVE_CUR_MOVIEID` | 保存当前投屏影片 |

---

## 5. 微信 SDK

### 配置

| 参数 | 来源 |
|------|------|
| AppId | `config.gradle` → `wxAppId` |
| AppSecret | `config.gradle` → `wxAppSecret` |

### 功能

| 功能 | 实现 |
|------|------|
| 微信登录 | 通过MobSDK授权获取code/openId |
| 微信分享 | 分享链接/图片/小程序到好友/朋友圈 |
| 微信支付 | 调用WxPayApi发起支付 |

---

## 6. MobSDK/ShareSDK（分享）

### 初始化

```
MobSDK.init(context)
// 配置文件: assets/ShareSDK.xml
```

### 支持平台

| 平台 | 功能 |
|------|------|
| 微信好友 | 分享链接/图片 |
| 微信朋友圈 | 分享链接/图片 |
| 微博 | 分享链接/图片 |
| QQ | 已注释,暂不可用 |

---

## 7. YJYZ 一键登录

### 初始化

```
MovieApplication中初始化 YJYZ SDK:
  - 检测运营商类型 (CMCC/CUCC/CTCC)
  - 预取号 prefetchPhoneNumber()
```

### 集成的运营商SDK

| 运营商 | SDK |
|--------|-----|
| 中国移动 | CMCC SDK |
| 中国联通 | CUCC SDK |
| 中国电信 | CTCC SDK |

---

## 8. 弹幕 (DanmakuFlameMaster)

### 使用

```
B站开源弹幕引擎，集成在播放器页面:
  - 从服务端/GreenDao缓存加载弹幕数据
  - 用户可发送弹幕(POST API)
  - 支持弹幕开/关
  - 弹幕样式: 滚动弹幕/顶部/底部
```

### 缓存

弹幕数据缓存到 GreenDao `DAN_MU_BEAN` 表，以 `movieId` 为键。

---

## 9. SVGAPlayer

### 用途
- 礼物动画播放（`PLAY_GIFT_SVGA` 事件触发）
- 从网络下载 SVGA 文件播放

---

## 10. 七牛云 / 腾讯云 COS

### 用途
- 用户头像上传
- 动态图片/视频上传
- 分别用于不同的存储后端

---

## 11. Arkana（密钥管理）

### 用途
- 编译期将 `.env` 中的敏感字符串混淆后编译到代码中
- 运行时通过 `Secrets.Global.xxx` 访问
- 保护: 客户端证书 PEM、私钥 PEM

### 配置
- 配置文件: `.arkana.yml`
- 密钥源: `.env`

---

## 12. 其他库

| 库 | 用途 |
|---|------|
| SmartRefreshLayout | 下拉刷新/上拉加载 |
| BGABanner | Banner轮播 |
| CircleImageView | 圆形头像 |
| PhotoView | 图片查看/缩放 |
| RxJava2 | 异步编程 |
| Timber | 日志框架 |
| PermissionX | 运行时权限请求 |
| OkGo + OkServer | 旧版下载框架 |
