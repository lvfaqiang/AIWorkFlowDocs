# 02 网络层实现

## 总体架构

```
业务层 (17个*NetUtils.kt)
        ↓ 调用
EHttp DSL 框架 (FrameModule提供)
        ↓ 封装
OkHttp3 (3.12.13) + Retrofit2 (2.6.0)
        ↓ 拦截器链
┌──────────────────────────────────────┐
│ 1. Auth Header 拦截器 (NetConfig)     │
│ 2. mTLS 客户端证书 (ClientCertHelper) │
│ 3. 全局请求头注入                      │
└──────────────────────────────────────┘
        ↓
服务端 API
```

---

## EHttp DSL 请求框架

项目使用自封装的 `EHttp` DSL 构建请求，核心用法：

```
// GET 请求
EHttp {
    src = "接口路径"
    type = TYPE.METHOD_GET
    data = { "参数名"("参数值") }
    header = { "header名"("header值") }
}.go<ResponseType>(
    success = { data -> },
    failed = { call, exception -> },
    progress = { value, total -> }  // 可选
)

// POST 请求
EHttp {
    src = "接口路径"
    type = TYPE.METHOD_POST
    data = { "参数名"("参数值") }
}.go<ResponseType>(success, failed)

// 指定不同 baseUrl
EHttp {
    baseUrl = ApiUrlProvider.API_USER_URL  // 覆盖默认baseUrl
    src = "v1/user/info"
    ...
}
```

---

## 网络初始化配置 (NetConfig.kt)

### 全局 OkHttp 配置

| 配置项 | 值 |
|-------|-----|
| 默认请求方式 | GET |
| 连接超时 | 60秒 |
| 读取超时 | 60秒 |
| 写入超时 | 60秒 |
| 失败重试 | 开启 |
| 缓存 | null（不缓存） |
| mTLS | 开启客户端证书 |

### 全局请求头

每个请求自动附带以下 Header：

| Header | 值 | 说明 |
|--------|-----|------|
| `C-PLATFORM` | `"1"` | 平台标识（1=Android） |
| `C-APP-VERSION` | 动态 | App版本号，如 "1.0.0.135" |
| `versionCode` | 动态 | 整数版本码，如 "135" |
| `width` | 动态 | 屏幕宽度（px） |
| `height` | 动态 | 屏幕高度（px） |
| `Content-Type` | `application/x-www-form-urlencoded` | 请求编码方式 |
| `promotion-channel` | 动态 | 渠道名（从Manifest的UMENG_CHANNEL读取） |
| `deviceId` | 动态 | Android ID |
| `cache-Control` | `no-cache` | 禁用客户端缓存 |
| `model` | 动态 | 如 "Xiaomi  Redmi Note 11" |
| `C-DEVICE-IDENTIFICATION` | 动态 | 同 Android ID |

### 认证 Header（已登录用户追加）

| Header | 值 | 条件 |
|--------|-----|------|
| `movienow-userid` | 用户ID | userID非空/非"0"/非"null" |
| `C-ACCESS-TOKEN` | Token值 | token非空/非"null" |
| `Authorization` | `Bearer {token}` | 同上 |
| `umToken` | 友盟Token | umToken非空 |

---

## 401 未授权处理流程

```
1. 拦截器检测到 response.code == 401
2. 调用 UserInfoConst.clearUserData() 清除本地用户数据
3. 关闭当前 response
4. 构建无认证Header的新请求并重试
5. 在主线程调用 ApiErrorUiRouter.onUnauthorized()
6. 显示 Toast "登录已过期，请重新登录"（节流1200ms防重复弹出）
```

### 静默后台请求（不弹错误提示的URL列表）

以下URL的网络错误和业务错误不会弹出UI提示：
- `api/v1/rest/personal/record/play` — 播放记录上报
- `api/v1/rest/personal/refresh` — 个人信息刷新
- `xzmovie2/rest/personal/refresh` — 旧版刷新
- `xzmovie2/rest/personal/movie/playrecord` — 旧版播放记录
- `api/v1/rest/personal/add/zan` — 点赞
- `api/v1/rest/personal/delete/zan` — 取消点赞
- `api/v1/rest/personal/add/collect` — 收藏
- `api/v1/rest/personal/delete/collect` — 取消收藏

---

## API URL 管理 (ApiUrlProvider.kt)

后端采用**微服务架构**，不同业务域使用不同子域名：

| 模块 | prod URL | dev URL | 业务范围 |
|------|----------|---------|---------|
| COMMON | `https://api-common.rrys.wasu.tv/` | `https://api-common-dev.yyets.com.cn/` | 验证码等通用 |
| USER | `https://api-user.rrys.wasu.tv/` | `https://api-user-dev.yyets.com.cn/` | 登录/个人中心 |
| MEDIA | `https://api-media.rrys.wasu.tv/` | `https://api-media-dev.yyets.com.cn/` | 电影/片库/短视频 |
| ZIXUN | `https://api-zixun.rrys.wasu.tv/` | `https://api-zixun-dev.yyets.com.cn/` | 资讯/新闻 |
| HOME | `https://api-home.rrys.wasu.tv/` | `https://api-home-dev.yyets.com.cn/` | 首页推荐 |
| KANKAN | `https://api-kankan.rrys.wasu.tv/` | `https://api-kankan-dev.yyets.com.cn/` | 看看tab短视频 |
| VIP | `https://api-vip.rrys.wasu.tv/` | `https://api-vip-dev.yyets.com.cn/` | VIP商品/支付 |
| COMMENT | `http://api-comment.rrys.wasu.tv/` | `http://api-comment-dev.yyets.com.cn/` | 评论 |
| STATIC | `https://api-static.rrys.wasu.tv/` | `https://api-static.yyets.com.cn/` | 全局配置 |

### 旧版 baseUrl（兼容）

| 环境 | URL | 用途 |
|------|-----|------|
| 正式 | `http://movie.ldd-tech.club/front/` | 旧版接口默认baseUrl |
| 测试 | `https://api-dev.yyets.com.cn/` | 旧版测试环境 |

### H5 页面 URL

| 环境 | URL |
|------|-----|
| 正式 | `https://h5.rrys.wasu.tv/` |
| 测试 | `https://h5-dev.yyets.com.cn/` |

---

## 全局配置加载 (GlobalConfigManager)

应用启动时从 `{API_STATIC_URL}/v1/global_config` GET 请求加载全局配置：

**响应结构**: `GlobalConfigResult` -> `GlobalConfigBean`

**控制开关**:
| 字段 | 类型 | 说明 |
|------|------|------|
| `isGreyDisplay` | Boolean | 全局灰色显示（哀悼模式，全App黑白化） |
| `forbidComment` | Boolean | 全局禁止评论 |
| `forbidUserModify` | Boolean | 全局禁止修改个人资料 |

配置加载后保存到 SharedPreferences，并通过 EventBus 发送 `GlobalConfigEvent` 通知所有 Activity。

---

## 错误处理策略

| 错误类型 | 处理方式 |
|---------|---------|
| HTTP 401 | 清除用户数据 → 无Token重试 → Toast提示（节流1200ms） |
| 网络错误 | 静默URL不提示，其他交由 `RequestManager.onNetworkError` |
| 业务错误 | code≠401时，静默URL不提示，其他记录日志 |
| OkHttp拦截器null崩溃 | 自定义UncaughtExceptionHandler捕获并忽略 |
| RxJava未处理错误 | FrameApplication全局RxJavaPlugins.setErrorHandler记录日志 |
