# 06 核心业务流程

## 1. 登录流程

### 1.1 短信验证码登录

```
用户输入手机号 → 请求验证码
  ├── 新版: POST api-common/v1/captcha/noa/send/sms
  └── 旧版: POST xzmovie2/rest/personal/noa/smsLogin

用户输入验证码 → 登录
  ├── 新版: POST api-user/v1/users/noa/login/sms
  └── 旧版: POST xzmovie2/rest/personal/noa/login

登录成功 → 处理 LoginBean/NewLoginResult:
  1. 保存 token 到 SharedPreferences
  2. 保存 userId/nick/headImg 等用户信息
  3. 更新 UserInfoConst 内存状态
  4. EventBus 发送 LOGIN_STATE = true
  5. 提交友盟 DeviceToken
  6. 判断用户信息是否完善(nick/gender等)
     → 不完善: 跳转完善信息页
     → 已完善: 返回前页
```

### 1.2 一键登录（三大运营商号码认证）

```
初始化 YJYZ SDK → 获取运营商类型
  ↓
YJYZ.getToken() → 获取 opToken
  ↓
POST xzmovie2/rest/personal/noa/oneKeyLogin
  请求: { md5(签名MD5), needLogin=true, opToken, operator(CMCC/CUCC/CTCC), token }
  ↓
成功 → PhoneNumBean { loginResponse, phone }
  → 同短信登录后续流程
失败 → 回退到短信验证码登录
```

### 1.3 三方登录（微信/QQ）

```
调用 MobTech ShareSDK 授权
  → 获取 accessToken + openId
  ↓
POST xzmovie2/rest/personal/noa/thirdPartyQuickLogin
  请求: { thirdLoginType(0=微信/3=QQ), accessToken, openId }
  ↓
├── 已绑定手机号 → 直接登录成功
└── 未绑定 → 弹出绑定手机号弹窗
    → 输入手机号+验证码
    → POST thirdBindPhoneLogin
    → 登录成功
```

### 1.4 登出流程

```
UserInfoConst.loginOut()
  1. clearUserData() — 清空内存+SP中所有用户字段
  2. LoginUtils.deleteHistory() — 清除搜索历史
  3. LoginUtils.deleteVideoHistory() — 清除观看历史
  4. EventBus.post(false, LOGIN_STATE) — 广播登出
  → 所有监听 LOGIN_STATE 的页面更新UI
```

---

## 2. 播放流程

### 2.1 进入影片详情

```
点击影片 → MovieDetailsAty
  ↓
1. 请求影片详情: GET api-media/.../movieDetail?movieId=xxx
2. 解析 MovieDetailNewData → 映射为旧版 DetailsInfo
3. 显示:
   - 封面/标题/评分/简介/演员导演
   - 正片列表 mainMovieItemList
   - 预告片列表 previewMovieItemList
   - 推荐影片列表
   - 影评列表
4. 恢复播放记录: playRecord → 定位到上次观看的集+进度
```

### 2.2 开始播放

```
用户选择剧集 → 检查播放权限:
  ├── VIP用户 → 直接播放
  ├── 免费剧集(priceType=="0") → 直接播放
  ├── 已购买(isPurchased) → 直接播放
  └── 需付费 → 弹出付费弹窗
       ├── 选择VIP充值 → 跳转VIP购买页
       └── 选择人人币解锁 → POST consumeMovieItem

获取播放地址:
  1. 从 EpisodeBean.movieUrlInfoList 获取 itemUrlId
  2. GET api-media/.../movieItemUrl?itemUrlId=xxx
  3. 返回真实播放URL
  4. 传给火山引擎播放器播放

播放过程中:
  - 每隔N秒自动保存播放记录 POST api-media/.../record/play
  - 支持切换清晰度(720P/1080P/2K/4K)
  - 支持发送/显示弹幕(DanmakuFlameMaster)
  - 支持投屏(乐播SDK)
  - 支持手势：左右滑动快进/快退，上下滑动亮度/音量
  - 支持倍速播放
  - 支持画中画
```

### 2.3 投屏流程

```
用户点击投屏按钮
  → LeBoService 扫描局域网投屏设备
  → 显示设备列表
  → 用户选择设备
  → EventBus.post(CALL_SELECT_LELINK)
  → 连接设备，将播放地址推送到投屏设备
  → 本地变为控制器模式（暂停/快进/音量/清晰度）
```

### 2.4 电视剧集切换

```
选集列表:
  - 默认显示第一集
  - 支持分段（如 1-50, 51-100）
  - 全屏模式下可点击选集

自动下一集:
  播放完成 → 检查是否有下一集
    → 有: 自动加载下一集
    → 无: 显示推荐列表
```

---

## 3. 下载/缓存流程

### 3.1 双引擎架构

```
下载管理
├── 旧版: DownloadManager (OkGo/OkServer)
│   └── 直接 HTTP 下载 mp4 文件
└── 新版: VolcDownloadManager (火山引擎 SDK)
    └── 加密视频下载+本地解密播放
```

### 3.2 发起下载

```
1. 用户在详情页点击"缓存"
2. 选择清晰度（跟随 selectBitrate 或手动选择）
3. 检查权限:
   ├── VIP用户 → 可下载全部
   ├── 免费剧集 → 可下载
   └── 需付费 → 提示购买
4. 批量获取下载URL:
   POST api-media/.../play-url/batch { itemUrlIdList }
5. 创建 CacheInfoBean 写入 GreenDao
6. 加入下载队列（最大并行: 3个任务）
7. 非Wi-Fi时检查 isDownloadControl 设置
```

### 3.3 下载状态机

```
STATE_NONE(0) → STATE_ADD(1) → STATE_WAITING(2) → STATE_DOWNLOADING(4)
  → STATE_SUCCESS(5) → STATE_PLAY(7)

可能的中间状态:
  - STATE_PAUSED(3): 用户暂停
  - STATE_ERROR(6): 下载失败
  - STATE_NO_NETWORK(13): 网络断开
  - STATE_CANCEL(8): 取消

批量操作:
  - STATE_PAUSE_ALL(9): 暂停全部
  - STATE_RECOVER_ALL(10): 恢复全部
  - STATE_DELETE_ALL(14): 删除全部
```

---

## 4. 支付/VIP 流程

### 4.1 VIP 购买

```
用户点击开通VIP → VIP商品页
  ↓
1. GET api-vip/v1/vip/noa/product/list
   → 获取商品列表 + isSubscribeUser + isBoughtVip
2. 用户选择商品（月卡/季卡/年卡/永久）
3. 选择支付方式:
   ├── 支付宝: payChannelType=0
   └── 微信: payChannelType=1
4. POST xzmovie2/rest/order/createOrder
   → 获取 PayBean { url/payParams, tradeNo }
5. 调起支付:
   ├── 支付宝: 打开 url
   └── 微信: 解析 payParams 调用 WxPayApi
6. 支付完成回调 → POST getOrderPayResult { tradeNo }
7. 确认成功:
   - 更新 UserInfoConst.isVipUser = true
   - 更新 vipExpiredTime
   - EventBus.post(PAY_SUCCESS)
   - 刷新页面状态
```

### 4.2 单集解锁（人人币）

```
未购买的付费剧集 → 弹出付费弹窗
  → 显示价格（人人币）
  → 确认解锁
  → POST consumeMovieItem { movieItemIdList }
  → 成功: 更新isPurchased，开始播放
  → 失败(余额不足): 跳转充值页
```

---

## 5. 搜索流程

```
搜索页:
  1. 显示搜索历史（从 SP.searchHistoryData 读取，JSON数组）
  2. 显示热门搜索（从 API 获取热词列表）
  3. 用户输入关键词
  4. 搜索接口:
     ├── 影片搜索: POST api-media/.../list { content, pageNo, pageSize }
     └── 资讯搜索: GET api-zixun/.../search/news { keyword }
  5. 保存搜索历史到 SP（去重，最多N条）
  6. 结果列表:
     ├── 影片 → 跳转详情页(MovieDetailsAty)
     └── 资讯 → 跳转H5详情页
```

---

## 6. 消息/通知流程

```
友盟推送:
  1. 后台推送到达 → UmengNotificationService
  2. 解析推送payload:
     ├── type=1(直播) → 跳转直播间
     ├── type=2(消息) → 跳转消息列表
     └── 默认 → 打开App首页
  3. 消息列表:
     - 系统消息
     - 互动消息（点赞/评论/关注）
     - 标记已读: POST readAllMsg / readCategory
```

---

## 7. 淘口令解析流程

```
App 前台恢复 → 读取剪贴板内容
  → 匹配 "copy://" 前缀
  → 提取 movieId
  → 弹出确认对话框 "是否跳转到XXX？"
  → 确认 → 清空剪贴板 → 跳转 MovieDetailsAty
```

---

## 8. 实名认证流程

```
POST xzmovie2/rest/personal/addRealNameAuth
  请求: { code(验证码), idCard(身份证号), realName(真实姓名) }
  ↓
成功 → 保存到 UserInfoConst:
  - authTime
  - idCard
  - realName
```

---

## 9. 版本更新与安装流程

```
App 启动 / 手动检查 → UpVersionManager 检查更新
  ↓
API 请求获取 VersionBean:
  - version (远程版本号)
  - loadUrl (APK下载地址)
  - dec (更新说明)
  - forceId (强制更新标识: 0=普通, 1=强制)
  ↓
比对本地 versionCode
  ├── 强制更新(forceId==1) → 弹窗强制拦截，屏蔽返回键/外部点击
  └── 非强制更新 → 显示可取消更新弹窗 (取消时可记录 rejectUpdVersion 避免频繁打扰)

用户点击"立即更新"
  → 使用自定义下载机制下载 APK
  → 下载完成 → 调用系统安装器 (处理 Android 7.0+ FileProvider 及 8.0+ 未知应用安装权限)
```

---

## 10. 发布动态流程 (后台Service上传)

```
用户在动态页点击发动态 → SendDynamicActivity
  ↓
1. 填写内容: 纯文本 / 图片(多图) / 视频 + 关联话题/影片
2. 点击"发送":
   - 缓存当前输入到 SP(sendDynamicSaveData)，用于草稿恢复
   - 启动后台服务 `SendDynamicService` 并传入数据
   - 此时 Activity 直接 finish() 返回动态列表，实现“秒回”体验
  ↓
3. Service 开始后台执行 (SendDynamicService):
   ├── 需要上传媒体? → 先将图片/视频上传至 七牛云/COS 获取 URL
   ├── 无需/上传完成 → 构建最终发布参数
   └── POST 调用发布接口
  ↓
4. 服务端返回结果:
   - 成功: `EventBus.post(SendDynamicResultEvent(true))`
   - 失败: `EventBus.post(SendDynamicResultEvent(false))` + 允许存草稿并重试
  ↓
5. 列表页UI层监听到 EventBus 事件:
   → 刷新动态列表
   → 提示发布成功/失败
```
