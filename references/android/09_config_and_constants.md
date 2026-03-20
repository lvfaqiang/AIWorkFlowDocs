# 09 配置与常量

## 环境配置 (AppConfig.kt)

通过 Gradle `productFlavors` 的 `BuildConfig.ENV` 控制环境切换：

| 变体 | ENV值 | applicationId | 说明 |
|------|-------|--------------|------|
| prod | `"prod"` | `com.wasu.rrys` | 正式环境 |
| dev | `"dev"` | `com.wasu.rrys.test` | 测试环境 |

```
AppConfig.IS_ONLINE → BuildConfig.ENV == "prod"
```

---

## SharedPreferences 持久化字段（完整清单）

### 用户信息

| 字段名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `id` | String | `""` | 用户ID |
| `nick` | String | `""` | 昵称 |
| `headImgUrl` | String | `""` | 头像URL |
| `mobile` | String | `""` | 手机号 |
| `sign` | String | `""` | 个性签名 |
| `token` | String? | `null` | 登录Token |
| `isVip` | Boolean | `false` | 是否为VIP |
| `vipExpiredTime` | String | `""` | VIP过期时间 |
| `gender` | Int | `0` | 性别（1男/2女） |
| `coin` | Int | `0` | 人人币余额 |
| `zanNum` | Int | `0` | 获赞数 |
| `fansNum` | Int | `0` | 粉丝数 |
| `attentionNum` | Int | `0` | 关注数 |
| `desc` | String | `""` | 个人描述 |
| `birthdayYear` | String | `""` | 生日年 |
| `birthdayMonth` | String | `""` | 生日月 |
| `birthdayDay` | String | `""` | 生日日 |
| `provinceId` | String | `""` | 省份ID |
| `cityId` | String | `""` | 城市ID |
| `countryId` | String | `""` | 区县ID |

### 实名认证

| 字段名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `authTime` | String | `""` | 实名认证时间 |
| `idCard` | String | `""` | 身份证号 |
| `realName` | String | `""` | 真实姓名 |

### 设备与推送

| 字段名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `imToken` | String? | `null` | IM Token |
| `umToken` | String | `""` | 友盟推送Token |
| `umDeviceId` | String | `""` | 友盟设备ID |
| `promotionChannel` | String | `""` | 推广渠道 |
| `envAddress` | String | `""` | 自定义接口环境地址 |

### 应用状态

| 字段名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `showUserPrivate` | Boolean | `true` | 是否显示隐私协议（true=未同意） |
| `showTopCover` | Boolean | `true` | 是否显示顶部封面 |
| `isFirstInstall` | Boolean | `true` | 是否首次安装 |
| `isFirstOpenSlide` | Boolean | `true` | 是否首次打开滑动页 |
| `isNewRegisterUser` | Int | `-1` | 新注册用户标记（-1未知） |
| `lastVersionUpdType` | Int | `1` | 上次更新类型 |
| `rejectUpdVersion` | Boolean | `false` | 是否拒绝当前版本更新 |
| `notifyPermission` | Long | `0` | 通知权限请求时间戳 |
| `homeGlide` | Boolean | `true` | 首页引导 |
| `movieCommentGlide` | Boolean | `true` | 影评引导 |

### 播放与缓存

| 字段名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `normalBitrate` | String | `"蓝光1080P"` | 默认播放清晰度 |
| `selectBitrate` | String | `"蓝光1080P"` | 缓存下载清晰度（退出页面后跟随normalBitrate） |
| `movieQuality` | String | `""` | 影片画质 |
| `movieId` | String | `""` | 当前影片ID |
| `isClickMovieCheck` | Boolean | `false` | 是否点击了影片 |
| `isDownloadControl` | Boolean | `false` | 允许非Wi-Fi下载 |

### 投屏

| 字段名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `leboPoint` | Boolean | `true` | 乐播投屏标记 |
| `lastLeboDeviceUid` | String | `""` | 上次投屏设备UID |

### 社区互动

| 字段名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `searchHistoryData` | String | `""` | 搜索历史（JSON字符串） |
| `dynamicSaveData` | String | `""` | 动态保存数据 |
| `sendDynamicSaveData` | String | `""` | 发送动态保存数据 |
| `drpCode` | String | `""` | 分销推荐码(DRP) |
| `launcherAds` | String | `""` | 启动页广告数据 |

### 全局配置

| 字段名 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `isGreyDisplay` | Boolean | `false` | 灰色显示模式 |
| `forbidComment` | Boolean | `false` | 禁止评论 |
| `forbidUserModify` | Boolean | `false` | 禁止修改个人资料 |
| `clearUserMsg` | Boolean | `true` | 清除用户消息 |
| `sysMessageCount` | Int | `0` | 系统消息计数 |
| `sysInviteMessageCount` | Int | `0` | 系统邀请消息计数 |

---

## 用户状态管理 (UserInfoConst)

全局单例对象，管理当前登录用户的内存状态。从 SharedPreferences 加载初始值。

### 登录判断

```
isLogin() → userID非空 && userID≠"" && userID≠"0" && token非空 && token≠"null"
```

### VIP 判断

```
isVip() → isVipUser == true
isVipForever() → isVipUser == true && vipExpiredTime 为空
```

### 登出流程

```
1. clearUserData()  — 清空内存变量 + SP中所有用户字段
2. LoginUtils.deleteHistory()  — 清除搜索历史
3. LoginUtils.deleteVideoHistory()  — 清除观看历史
4. EventBus.post(false, EventConfig.LOGIN_STATE)  — 广播登出事件
```

---

## EventConfig 事件常量（完整清单，90+）

### 支付相关

| 常量 | 值 | 说明 |
|------|----|------|
| `PAY_SUCCESS` | `"pay_success"` | 支付成功 |
| `PAY_STATUS` | `"pay_status"` | 支付结果状态 |
| `BRAND_PAY_SUCCESS` | `"brand_pay_success"` | 品牌厅支付成功 |
| `BUY_EPISODE` | `"buy_episode"` | 购买电视剧剧集 |
| `BUY_EPISODE_NEW` | `"buy_episode_new"` | 购买剧集(新) |
| `SHOW_PAY_DIALOG` | `"show_pay_dialog"` | 打开支付弹窗 |
| `PAY_RED_ENVELOPE_SUCCESS` | `"pay_red_envelop_success"` | 红包支付成功 |
| `CHAT_PAY_RED_ENVELOPE_SUCCESS` | `"chat_pay_red_envelop_success"` | 聊天红包支付成功 |
| `MONEY_PAY_SUCCESS` | `"money_pay_success"` | 收益提现成功 |

### 登录与用户

| 常量 | 值 | 说明 |
|------|----|------|
| `LOGIN_STATE` | `"login_state"` | 登录状态变更 |
| `MAIN_LOAD_USER_INFO` | `"main_load_user_info"` | 首页获取用户信息 |
| `USERINFO_STATE` | `"user_info_state"` | 用户信息完善状态 |
| `RONGYUN_OTHER_LOGIN` | `"rongyun_other_login"` | 异地登录检测 |

### 播放器控制

| 常量 | 值 | 说明 |
|------|----|------|
| `MOVIE_DRAG_PROGRESS` | `"movie_drag_progress"` | 进度条拖动 |
| `VOD_CONTROLLER` | `"vod_controller"` | 播放器控制器 |
| `SWITCH_TRAILER` | `"switch_trailer"` | 切换预告片 |
| `REFRESH_TV` | `"refresh_tv"` | 刷新电视剧播放 |
| `REFRESH_MOVIE` | `"refresh_movie"` | 刷新电影播放 |
| `REFRESH_EPISODE` | `"refresh_episode"` | 刷新剧集播放 |
| `REFRESH_EPISODE_LIST` | `"refresh_episode_list"` | 刷新剧集列表 |
| `EPISODE_LIST` | `"episode_list"` | 剧集列表 |
| `FULLSCREEN_CLICK_EPISODE` | `"fullscreen_click_episode"` | 全屏点击剧集 |
| `CURRENT_MOVIE` | `"current_movie"` | 更新正在播放标题 |
| `REFRESH_BITRATE` | `"refresh_bitrate"` | 更新码率 |
| `GESTURE_MOVIE_PROGRESS` | `"gesture_movie_progress"` | 手势滑动进度 |
| `BUG_CHANGE_QUALITY_NEXT_ON_PLAY_END` | `"bug_change_quality_next_on_play_end"` | 切换清晰度bug修复标记 |

### 弹幕

| 常量 | 值 | 说明 |
|------|----|------|
| `DANMU_CONTROLLER` | `"danmu_controller"` | 弹幕开关控制 |
| `DANMU_INPUT_STATUS` | `"danmu_input_status"` | 弹幕输入窗状态 |
| `KNOWLEDGE_DANMU` | `"knowledge_danmu"` | 冷知识列表转弹幕 |
| `CLICK_COLD_KNOWLEDGE` | `"click_cold_knowledge"` | 点击冷知识 |

### 投屏

| 常量 | 值 | 说明 |
|------|----|------|
| `CALL_SELECT_LELINK` | `"call_select_lelink"` | 选择投屏设备 |
| `CONNECT_LEBO` | `"connect_lebo"` | 连接投屏 |
| `LEBO_SHARPNESS_SELECT` | `"lebo_sharpness_select"` | 投屏清晰度选择 |
| `LEBO_LINK_PLAY_PAUSE` | `"lebo_link_play_pause"` | 投屏播放/暂停 |
| `LEBO_VIEW_SHOW` | `"lebo_view_show"` | 投屏占位View显示 |
| `NOTIFY_SAVE_CUR_MOVIEID` | `"notify_service_save_movieid"` | 保存当前投屏影片ID |

### 社区与短视频

| 常量 | 值 | 说明 |
|------|----|------|
| `VIDEO_LIST_ITEM_FRAGMENT_CHANGE` | `"video_list_item_fragment_change"` | 切换fragment暂停短视频 |
| `VIDEO_LIST_ITEM_PLAY_ONE` | `"video_list_item_play_one"` | 切换单个播放 |
| `VIDEO_COLLECTION_STATUS` | `"video_collection_status"` | 短视频收藏状态 |
| `SHORT_VIDEO_COLLECT_CLICK` | `"short_video_collect_click"` | 短视频收藏 |
| `SHORT_VIDEO_SHARE_CLICK` | `"short_video_share_click"` | 短视频分享 |
| `SHORT_VIDEO_LIKE_CLICK` | `"short_video_like_click"` | 短视频点赞 |
| `SHORT_VIDEO_LONG_PRESS` | `"short_video_long_press"` | 短视频长按倍速 |
| `VIDEO_STATUS_REFRESH` | `"video_status_refresh"` | 视频状态刷新 |

### 下载缓存

| 常量 | 值 | 说明 |
|------|----|------|
| `REFRESH_CACHE_FINISH` | `"refresh_cache_finish"` | 下载完成 |
| `REFRESH_TASK_CACHING` | `"refresh_task_caching"` | 正在下载 |

### 首页与导航

| 常量 | 值 | 说明 |
|------|----|------|
| `MAIN_HEAD_SWITCH_STAGE` | `"main_head_switch_stage"` | 首页切换顶部沉浸图 |
| `MAIN_HAS_HEAD_STATE` | `"main_has_head_state"` | 当前是否在顶部 |
| `TAB_DOUBLE_REFRESH` | `"tab_double_refresh"` | 双击tab刷新 |
| `NOTIFY_SLIDE_REFRESH` | `"notify_slide_refresh"` | 关闭广告后刷新 |
| `NET_WORK_CHANGE` | `"net_work_change"` | 网络切换事件 |
| `FULL_WINDOW` | `"full_window"` | 全屏切换 |
| `SWITCH_FULL_VIEW_HEIGHT` | `"switch_full_view_height"` | 横竖屏UI更新 |

### 影评与评论

| 常量 | 值 | 说明 |
|------|----|------|
| `COMMENT_SUCCESS` | `"comment_success"` | 评论成功 |
| `CREATE_MOVIE_REVIEW_H5` | `"create_movie_review_h5"` | H5创建影评 |
| `CREATE_MOVIE_REVIEW_AV` | `"create_movie_review_av"` | AV页创建影评 |
| `CREATE_MOVIE_REVIEW_MOVIE_DETAILS` | `"create_movie_review_moviedetails"` | 详情页创建影评 |
| `EDIT_MOVIE_MOMENT` | `"edit_movie_moment"` | 写评语 |

### 其他

| 常量 | 值 | 说明 |
|------|----|------|
| `PLAY_GIFT_SVGA` | `"play_gift_svga"` | 播放SVG礼物动画 |
| `POSTER_TRANCE_DATA` | `"poster_trance_data"` | 传递海报数据 |
| `MOVIE_GO_DETAILS` | `"movie_go_details"` | 进入影片详情 |
| `TRAILER_GO_DETAILS` | `"trailer_go_details"` | 进入预告片详情 |
| `MOVIE_SHARE_MINE_HAIBAO` | `"movie_share_mine_haibao"` | 分享专属海报 |
| `DEVICE_PERMI` | `"device_permi"` | 设备授权 |
| `NEW_USER_TICKET` | `"new_user_ticket"` | 新人优惠券 |
| `COUNT_DOWN_REFRESH` | `"count_down_refresh"` | 倒计时刷新 |
| `PLAY_END_APPOINTMENT` | `"play_end_appointment"` | 试看结束 |
| `UM_GET_DEVICE_TOKEN` | `"get_um_device_token"` | 友盟DeviceToken获取 |
| `REFRESH_CALLBACK` | `"refresh_callback"` | 通用刷新回调 |
| `ORDER_DETAIL_CALLBACK` | `"order_detail_callback"` | 订单详情返回 |
| `CALL_SELECT_USERCODE` | `"call_select_usercode"` | 选择观影码 |

---

## Constants.java 通用常量

### 下载状态机

| 常量 | 值 | 说明 |
|------|-----|------|
| `STATE_NONE` | 0 | 无状态 |
| `STATE_ADD` | 1 | 加入下载 |
| `STATE_WAITING` | 2 | 等待中 |
| `STATE_PAUSED` | 3 | 已暂停 |
| `STATE_DOWNLOADING` | 4 | 下载中 |
| `STATE_SUCCESS` | 5 | 下载完成 |
| `STATE_ERROR` | 6 | 下载失败 |
| `STATE_PLAY` | 7 | 可播放 |
| `STATE_CANCEL` | 8 | 已取消 |
| `STATE_PAUSE_ALL` | 9 | 全部暂停 |
| `STATE_RECOVER_ALL` | 10 | 全部恢复 |
| `STATE_RESUME` | 11 | 恢复下载 |
| `STATE_DELETE` | 12 | 删除 |
| `STATE_NO_NETWORK` | 13 | 无网络 |
| `STATE_DELETE_ALL` | 14 | 删除全部 |
| `STATE_DELETE_SELECT` | 15 | 删除选中 |
| `STATE_DELETE_FILE_SELECT` | 16 | 删除选中文件 |
| `MAX_DOWNLOAD_TASK` | 3 | 最大并行下载数 |

### 支付常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `PAY_TYPE_ALI` | `"ali"` | 支付宝 |
| `PAY_TYPE_WX` | `"wx"` | 微信 |
| `PAY_BUY_COIN_ALI` | `"Charge.getAliOrder"` | 支付宝购币 |
| `PAY_BUY_COIN_WX` | `"Charge.getWxOrder"` | 微信购币 |
| `PACKAGE_NAME_ALI` | `"com.eg.android.AlipayGphone"` | 支付宝包名 |
| `PACKAGE_NAME_WX` | `"com.tencent.mm"` | 微信包名 |
| `PACKAGE_NAME_QQ` | `"com.tencent.mobileqq"` | QQ包名 |

### 影片类型

| 常量 | 值 | 说明 |
|------|-----|------|
| `TYPE_MOVIE` | 0 | 电影 |
| `TYPE_TV` | 1 | 电视剧 |
| `TYPE_CACHE` | -1 | 正在缓存 |
| `VALID_PLAY` | 1 | 有效期内可播放 |

### 性别

| 常量 | 值 |
|------|-----|
| `SEX_MALE` | 1 |
| `SEX_FEMALE` | 2 |

### 文件存储路径

| 路径 | 说明 |
|------|------|
| `{外部缓存}/xianzai/video/` | 视频存储 |
| `{外部缓存}/xianzai/tieZhi/` | 贴纸下载 |
| `{外部缓存}/xianzai/music/` | 音乐下载 |
| `{外部缓存}/xianzai/camera/` | 拍照图片 |
| `{外部缓存}/gif/` | GIF缓存 |

---

## Manifest 配置项

| 配置项 | 用途 |
|--------|------|
| `SERVER_HOST` | 正式环境域名 |
| `SERVER_HOST_TEST` | 测试环境域名 |
| `SERVER_GRAY` | 灰度环境域名 |
| `JPUSH_APPKEY` | 极光推送AppKey |
| `TencentMapSDK` | 腾讯地图AppKey |
| `UMENG_CHANNEL` | 友盟渠道名 |
| `UMENG_APPKEY` | 友盟AppKey: `5db1093d570df3b928000295` |
| `buglyAppId` | Bugly AppId: `0d026bf359` |
