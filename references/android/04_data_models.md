# 04 数据模型

## 模型文件清单

| 文件 | 模型数 | 业务域 |
|------|--------|--------|
| MovieBean.kt | 30+ | 影片详情/剧集/演员/搜索/推荐/筛选/上新 |
| UserBean.kt | 20+ | 登录/注册/三方绑定/Token/用户信息/关注/粉丝 |
| HomeFragBean.kt | 15+ | 首页Tab/Banner/栏目/剧集/清晰度 |
| PersonBean.kt | 20+ | 用户/订单/追剧/播放记录/实名/人人币 |
| ShortVideoBean.kt | 20+ | 短视频/评论/资讯/搜索 |
| PayBean.kt | 4 | 支付/订单/交易详情 |
| VipBean.kt | 3 | VIP商品 |
| CommonDataBean.kt | 10+ | 通用数据(版本更新/启动广告/剧集URL等) |
| DiscussBean.kt | 10+ | 影评/评分 |
| NewsBean.kt | 5+ | 资讯/新闻 |
| MessageBean.kt | 5+ | 消息 |
| GlobalConfigBean.kt | 2 | 全局配置 |
| LoginDataMapper.kt | — | 登录数据映射工具 |
| RrCurrencyBean.kt | 2 | 人人币 |

---

## 1. 影片核心模型

### DetailsInfo（旧版影片详情）

| 字段 | 类型 | 说明 |
|------|------|------|
| `movieId` | String? | 影片ID（@SerializedName("id")） |
| `movieName` | String? | 影片名 |
| `movieNameEn` | String? | 英文名 |
| `movieAlias` | String? | 别名 |
| `movieType` | String? | 类型: "0"电影/"1"电视剧/"2"动漫/"3"综艺/"4"纪录片/"5"短剧 |
| `coverHorizontal` | String? | 横版封面URL |
| `coverVertical` | String? | 竖版封面URL |
| `posterHorizontal` | String? | 横版海报URL |
| `posterVertical` | String? | 竖版海报URL |
| `gifHorizontal` | String? | 横版GIF |
| `gifVertical` | String? | 竖版GIF |
| `description` | String? | 简介 |
| `shortDesc` | String? | 一句话描述 |
| `starValue` | Float? | 评分值 |
| `time` | String? | 年代 |
| `country` | String? | 国别 |
| `language` | String? | 语言 |
| `themeTags` | String? | 题材标签（逗号分隔） |
| `directorTags` | String? | 导演标签 |
| `actorTags` | String? | 演员标签 |
| `scriptTags` | String? | 编剧标签 |
| `resolution` | String? | 分辨率 |
| `totalNum` | Int? | 总集数 |
| `updateNum` | Int? | 已更新集数 |
| `playNumber` | Int? | 播放次数 |
| `shareNum` | Int? | 分享数 |
| `enable` | String? | 状态: "0"不可用/"1"可用 |
| `isDownload` | Int? | 是否可下载 |
| `isFollow` | Boolean? | 是否追剧 |
| `openDiscuss` | Int? | 是否开启影评 |
| `displayNowFansScore` | String? | 是否显示影迷评分 |
| `offTime` | String? | 下架时间 |
| `mainMovieItemList` | List\<EpisodeBean\>? | 正片剧集列表 |
| `previewMovieItemList` | List\<EpisodeBean\>? | 预告片列表 |
| `playRecord` | PlayRecord? | 播放记录 |
| `actorTagModelList` | List\<MovieActor\>? | 演员列表(带图片) |
| `directorTagModelList` | List\<MovieActor\>? | 导演列表 |
| `scriptTagModelList` | List\<MovieActor\>? | 编剧列表 |
| `pictures` | String? | 海报（逗号分隔URL） |
| `mediaIsVip` | Boolean? | 是否需要VIP |
| `commentNumber` | Int? | 评论数 |
| `isHuashu` | Boolean? | 是否华数内容 |

### 辅助方法
- `isMovie()` → movieType == "0"
- `isPurchased(selectNo)` → 检查指定集是否已购买
- `isFree(selectNo)` → 检查指定集是否免费
- `isVip()` → 当前用户是否VIP

---

### EpisodeBean（剧集）

| 字段 | 类型 | 说明 |
|------|------|------|
| `movieItemId` | String? | 剧集ID（@SerializedName("id")） |
| `movieId` | String? | 所属影片ID |
| `movieItemName` | String? | 剧集名（@SerializedName("name")） |
| `episode` | Int? | 集数 |
| `movieLength` | Int? | 时长(秒) |
| `movieUrlInfoList` | List\<MovieUriInfo\>? | 多清晰度的播放地址 |
| `priceType` | String? | 收费类型: "0"免费/"1"收费/"2"单集解锁 |
| `originalPrice` | Int? | 原价(分) |
| `price` | Int? | 现价(分) |
| `isPurchased` | Boolean? | 是否已购买 |
| `sort` | String? | 排序 |
| `zanNum` | Int? | 点赞数 |
| `isZan` | Boolean? | 是否已点赞 |

### 播放权限判断逻辑 (isBuy)
```
1. VIP用户 → 可观看所有
2. 免费内容(priceType=="0") → 所有人可看
3. 已购买 → isPurchased==true && 对应清晰度的isPurchased==true
```

---

### MovieUriInfo（播放地址）

| 字段 | 类型 | 说明 |
|------|------|------|
| `itemUrlId` | String? | URL ID（用于请求真实播放地址） |
| `movieUrl` | String? | 播放URL（需二次请求获取） |
| `movieVideoType` | String? | 清晰度名称（如"1080P"） |
| `size` | Long? | 文件大小(字节) |
| `isPurchased` | Boolean? | 该清晰度是否已购买 |

### 清晰度映射
| movieVideoType | 显示名 | 是否高清（需付费） |
|---------------|--------|-----------------|
| 4K | 超高清4K | ✅ |
| 2K | 超高清2K | ✅ |
| 1080P | 超清1080P | ✅ |
| 720P | 高清720P | ❌ |
| 480 | 标清480 | ❌ |

---

### PlayRecord（播放记录）

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | String? | 记录ID |
| `userId` | String? | 用户ID |
| `movieId` | String? | 影片ID |
| `movieItemId` | String? | 剧集ID |
| `movieItemDuration` | Int? | 剧集总时长(秒) |
| `record` | Int? | 已播放进度(秒) |
| `isLastPlay` | Boolean? | 是否最后播放的剧集 |

---

## 2. 用户模型

### LoginBean（登录结果）

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | String? | 用户ID |
| `token` | String? | Token |
| `nick` | String? | 昵称 |
| `headImg` | String? | 头像 |
| `phone` | String? | 手机号 |
| `email` | String? | 邮箱 |
| `sex` | String? | 性别: "0"未知/"1"男/"2"女 |
| `vip` | Boolean? | 是否VIP |
| `vipExpiredTime` | String? | VIP过期时间(null=永不过期) |
| `coin` | Int? | 人人币 |
| `fansNum` | Int? | 粉丝数 |
| `zanNum` | Int? | 获赞数 |
| `attentionNum` | Int? | 关注数 |
| `intro` | String? | 简介 |
| `qqBindStatus` | Int? | QQ绑定: 1=已绑/0=未绑 |
| `birthday` | String? | 生日 |
| `address` | String? | 地址 |
| `authTime` | String? | 实名认证时间 |
| `idCard` | String? | 身份证号 |
| `realName` | String? | 真实姓名 |
| `forbidComment` | Boolean? | 是否禁评 |

---

## 3. 首页模型

### HomeIndexData（新版首页）

| 字段 | 说明 |
|------|------|
| `tabItems` | Tab项列表（每个Tab含banner和mediaList） |
| `recommendBanner` | 推荐Banner |
| `mediaGroup` | 媒体分组（含焦点影片和列表） |

### HomeIndexMedia

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | Int? | 影片ID |
| `name` | String? | 名称 |
| `mediaType` | Int? | 类型 |
| `horizontalCover` | String? | 横版封面 |
| `verticalCover` | String? | 竖版封面 |
| `star` | String? | 评分 |
| `totalEpisode` | Int? | 总集数 |
| `shortIntro` | String? | 简介 |
| `isVip` | Boolean? | VIP标记 |
| `updateNum` | Int? | 更新集数 |

---

## 4. 影片类型映射

```
首页Tab position → 显示名称:
  1=电影, 2=电视剧, 3=短剧, 4=综艺, 5=动漫, 6=纪录片

后端 API movieType → 显示名称:
  0=电影, 1=剧集, 2=动漫, 3=综艺, 4=纪录片, 5=短剧

首页Tab position → API movieType:
  1→0(电影), 2→1(剧集), 3→3(综艺), 4→2(动漫), 5→4(纪录片)
```

---

## 5. VIP 商品模型

### VipProductBean

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | String? | 商品ID |
| `name` | String? | 商品名称 |
| `firstBuyName` | String? | 首购名称 |
| `month` | Int? | 月数(null=无期限) |
| `originalPrice` | Int? | 原价(分) |
| `price` | Int? | 现价(分) |
| `vipTag` | String? | 标签(如"推荐") |
| `limitPeople` | Int? | 限量人数(null=不限) |
| `isAutoRenewable` | Boolean | 是否自动续费 |
| `enable` | String? | 是否可用 |

### VipProductResult

| 字段 | 说明 |
|------|------|
| `list` | 商品列表 |
| `isSubscribeUser` | 当前用户是否订阅用户 |
| `isBoughtVip` | 当前用户是否购买过VIP |

---

## 6. 支付模型

### PayRequest

| 字段 | 值 | 说明 |
|------|-----|------|
| `goodsId` | 商品ID | 来自VIP商品列表 |
| `goodsType` | 0 | 0=VIP |
| `payChannelType` | 0/1/2 | 0=支付宝, 1=微信, 2=苹果IAP |
| `payload` | String? | 苹果IAP payload |
| `transactionId` | String? | 苹果IAP交易ID |

### TradeDetailBean（订单详情）

| 字段 | 类型 | 说明 |
|------|------|------|
| `orderNo` | String | 订单号 |
| `orderStatus` | String | 0=待支付/1=已支付/2=已取消 |
| `originalPrice` | Double | 原价 |
| `payment` | Double | 实付 |
| `payChannelType` | String | 支付方式 |
| `goodsType` | String | 商品类型 |
| `createTime` | String | 创建时间 |
| `finishTime` | String | 支付时间 |
| `startTime` | String | 开始时间 |
| `expiredTime` | String | 过期时间 |

---

## 7. 短视频模型

### ShortVideoBean

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | String? | 短视频ID |
| `content` | String? | 内容描述 |
| `cover` | String? | 封面URL |
| `videoLength` | Int? | 时长(秒) |
| `videoWidth` / `videoHeight` | Int? | 宽高 |
| `movieUrlInfoList` | List\<MovieUriInfo\>? | 播放地址列表 |
| `authorName` / `authorHeadImg` | String? | 作者信息 |
| `userId` | String? | 作者ID |
| `refMovieId` | String? | 关联影片ID |
| `zanNum` / `zanStatus` | Int?/Boolean? | 点赞 |
| `collectNum` / `collectionStatus` | Int?/Boolean? | 收藏 |
| `commentNum` | Int? | 评论数 |
| `shareNum` | Int? | 分享数 |
| `attentionStatus` | Boolean? | 是否关注作者 |
| `playNumber` | Int? | 播放次数 |

### CommentBean（评论）

| 字段 | 类型 | 说明 |
|------|------|------|
| `commentId` | String? | 评论ID |
| `comment` | String? | 评论内容 |
| `userName` | String? | 用户名 |
| `userHeadImg` | String? | 头像 |
| `userId` | String? | 用户ID |
| `commentTime` | String? | 评论时间 |
| `ipLocation` | String? | IP归属地 |
| `zanNum` | Int | 点赞数 |
| `zanStatus` | Boolean? | 是否点赞 |
| `commentReplyCount` | Int | 回复数 |
| `replyCommentList` | List\<CommentBean\>? | 回复列表 |

---

## 8. EventBus 事件Bean

| Bean | 字段 | 说明 |
|------|------|------|
| `FilmEvent` | movieId, name, ... | 影片相关事件 |
| `EpisodeEvent` | episodeIndex, ... | 剧集切换事件 |
| `VideoEvent` | ... | 视频状态事件 |
| `FollowEvent` | userId, status | 关注/取关事件 |
| `LikeDynamicEvent` | id, status | 动态点赞事件 |
| `DeleteDynamicEvent` | id | 删除动态事件 |
| `CollectDynamicEvent` | id, status | 收藏动态事件 |
| `ShareDynamicEvent` | id | 分享动态事件 |
| `ChangePageEvent` | page | 页面切换事件 |
| `TopicEvent` | topicId | 话题事件 |
| `LeBoConnectEventBus` | device | 投屏连接事件 |
| `LeBoServiceEventBus` | status | 投屏服务事件 |
| `RefreshUserInfoEvent` | | 刷新用户信息 |
| `RefreshNotificationUnreadEvent` | count | 通知未读数 |
| `RefreshDynamicCommentEvent` | | 刷新动态评论 |
| `SendDynamicResultEvent` | result | 发送动态结果 |
| `CreateReviewEventBusBean` | reviewId | 创建影评 |

---

## 9. 新版统一评论模型 (CommonDataBean.kt)

### NewCommentItem (统一评论数据结构)

> **注意**: 该模型字段支持蛇形命名(下划线)与驼峰命名双兼容，此处列出主要字段。

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | Int? | 评论ID |
| `channel` | String? | 频道 (`short_video`\|`media`\|`article`\|`news`) |
| `item_id` | String? | 关联内容ID |
| `author_id` / `user_id` | Int? | 作者ID |
| `author_name` | String? | 作者昵称 |
| `author_avatar` | String? | 作者头像 |
| `content` | String? | 评论内容 |
| `score` | String? | 评分 (0-10) |
| `parent_id` | Int? | 父评论ID (0为一级评论) |
| `reply_id` | Int? | 回复目标评论ID |
| `zan_count` / `zanCount` | Int? | 点赞数 |
| `reply_count` / `replyCount`| Int? | 回复数 |
| `is_zan` | Boolean? | 当前用户是否点赞 |
| `is_anonymous` | Boolean? | 是否匿名发布 |
| `ip_location` | String? | IP归属地 |
| `created_at` | String? | 创建时间 |
| `children` / `replyList` | List\<NewCommentItem\>?| 子评论/回复列表 |

### 辅助模型定义

| 模型类 | 作用 |
|--------|------|
| `CommentChannel` | 枚举类：SHORT_VIDEO, MEDIA, ARTICLE, NEWS |
| `NewCommentListResult` | 列表数据容器：包含 `comments`(列表)、`total`(总数)、`pages`(总页数) |
| `NewCommentUserItem` | 用户个人影评模型，扩展了 `media_name`, `media_score` 字段 |
| `PublishCommentRequest`| 发布评论请求体体：包含 `channel`, `is_anonymous`, `item_id`, `content`, `score`, `parent_id` 必选参数 |
