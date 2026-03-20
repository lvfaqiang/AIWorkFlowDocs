# 01 架构总览

## 项目信息

| 项目 | 值 |
|------|-----|
| 应用名 | 人人影视 (RRYS) |
| 包名 | `com.wasu.rrys`（prod）/ `com.wasu.rrys.test`（dev） |
| 核心代码包 | `com.chuanying.xianzaikan` |
| 语言 | Kotlin + Java 混合 |
| 最低版本 | Android 6.0 (API 23) |
| 目标版本 | Android 11 (API 30) |
| 版本号 | v1.0.0.135 (versionCode 135) |
| NDK ABI | armeabi-v7a, arm64-v8a |

---

## 模块架构（9个Gradle模块）

```
app (Application模块)
├── FrameModule        → 基础框架（Base类/网络/日志/扩展/SwipeBack）
├── imagelibrary       → 图片选择与裁剪
├── push               → 推送封装
├── mobs               → ShareSDK/MobTech 分享
├── jsBridgeLib        → WebView JS 桥接
├── wheelview          → 滚轮控件
├── pickerview         → 选择器控件
└── vod-playerkit      → 视频播放器组件
    ├── vod-player           → 播放器核心接口
    ├── vod-player-utils     → 播放器工具类
    └── vod-player-volcengine → 火山引擎播放器实现
```

---

## App 模块包结构（22个子包）

```
com.chuanying.xianzaikan/
├── app/            → Application入口、全局配置、常量、URL管理
├── api/            → 17个网络请求工具类（按业务域划分）
├── base/           → BaseActivity, BaseFragment, BaseDialog
├── bean/           → 数据模型（含 movie/, event/, enums/ 子目录）
├── adapter/        → 公共 RecyclerView Adapter
├── custom/         → 自定义 View
├── db/             → GreenDao 数据库工具（CacheOpenHelper/MigrationHelper）
├── dialog/         → 通用对话框组件
├── download/       → 下载管理（OkGo旧版 + 火山引擎新版）
├── event/          → EventBus 事件定义
├── greendao/       → GreenDao 自动生成的DAO代码
├── interfaces/     → 业务接口定义
├── live/           → 直播功能（beauty美颜/video视频/common通用）
├── manager/        → 全局管理器类
├── player/         → 播放器 URL 刷新（AppUrlRefreshFetcher）
├── service/        → 后台 Service（乐播投屏/动态发布）
├── third/          → 第三方集成（wechat/weibo/qq/payment）
├── ui/             → UI层（14个功能模块，详见07_ui_pages）
├── utils/          → 通用工具类
├── validator/      → 登录验证器、Activity生命周期管理
├── viewmodel/      → ViewModel（目前仅2个空壳）
└── widget/         → 自定义Widget（dialog/toast/ratingstar/tags/loading等）
```

---

## FrameModule 框架层内容

```
com.moving.kotlin.frame/
├── base/
│   ├── FrameApplication    → Application基类（Activity栈管理/RxJava错误处理）
│   ├── SharedPreferencesUtils → SP持久化工具（委托属性模式，50+字段）
│   └── CrashHandler        → 全局崩溃捕获
├── http/
│   ├── Api.kt               → Retrofit 初始化（含SSL配置）
│   ├── ClientCertificateHelper → mTLS 客户端证书配置
│   ├── EHttp / EClient       → DSL 风格的HTTP请求框架
│   ├── IpConfig              → IP配置
│   └── ViewModelExt / ExcuteObserverExt → RxJava扩展
├── ext/                     → Kotlin 扩展函数集合
│   ├── BindView / ViewExt    → View绑定与扩展
│   ├── ImageLoader           → Glide 图片加载扩展
│   ├── GsonExt               → Gson序列化扩展
│   ├── StringExt / TextFormatExt → 字符串处理
│   ├── StartActivityExt      → 跳转辅助
│   ├── ContextExt / BundleExt → Context和Bundle扩展
│   └── ToastExt / ToastBridge → Toast显示
├── log/                     → 日志系统（链式处理器模式）
├── abs/                     → Adapter/ViewHolder 基类
├── swipeback/               → 侧滑返回
├── utils/                   → StatusBarUtil/BlurTransformation等工具
└── custom/                  → NoTouchViewPager 等自定义控件
```

---

## Base 类继承体系

```
Application
└── MultiDexApplication
    └── FrameApplication     → Activity栈/RxJava错误处理/渠道获取
        └── MovieApplication → 全部SDK初始化入口

AppCompatActivity
└── BaseActivity             → EventBus注册/状态栏/灰色模式/字体锁定
    ├── MovieDetailsAty      → 影片详情
    ├── HomeActivity         → 首页
    ├── LoginActivity        → 登录
    └── ... (所有业务Activity)

Fragment
└── BaseFragment             → 基础Fragment
```

### BaseActivity 模板方法

每个 Activity 必须实现：
1. `layout(): Int` — 返回布局资源ID
2. `createView()` — 初始化View
3. `initListener()` — 绑定事件（可选override）

BaseActivity 自动处理：
- EventBus 注册/注销（`onCreate`/`onDestroy`）
- 透明状态栏 + 深色文字图标
- 灰色显示模式（通过 `GlobalConfigManager` 控制全局黑白化）
- 系统字体缩放锁定（忽略系统字体大小设置）
- 导航栏颜色设置
- 友盟页面采集 `PushAgent.onAppStart()`

---

## 事件通信系统

项目使用**两套 EventBus**：

| 库 | 包名 | 使用场景 |
|---|-------|---------|
| AndroidEventBus | `org.simple.eventbus` | 主要使用，BaseActivity自动注册，通过tag区分事件 |
| GreenRobot EventBus | `org.greenrobot.eventbus` | 友盟推送回调等少数场景 |

事件常量全部定义在 `EventConfig.kt`（90+常量），按功能分组。
事件Bean定义在 `bean/event/` 目录（21个事件类）。

---

## CI/CD

| 项目 | 配置 |
|------|------|
| CI平台 | GitHub Actions |
| 发布脚本 | `scripts/release.sh`（自动递增版本号） |
| 分发平台 | 蒲公英 (PGYER) |
| APK命名 | `rrys_{yyyyMMdd}_v{版本号}.apk` |
| 签名文件 | `rrzjKeystore.jks` |
| 构建变体 | `prodRelease` / `devDebug` |
