# 03 API 接口契约

## API 响应通用格式

所有 API 统一响应格式：

```json
{
  "code": 200,
  "message": "操作成功",
  "success": true,
  "timestamp": 1710000000000,
  "result": { ... },
  "request_id": "xxx"
}
```

> **注意**: 旧版 API 没有 `timestamp` 和 `request_id` 字段。
> 分页数据统一使用 `current/pages/size/total/records` 结构。

---

## API 模块总览（15个工具类 → 8个后端微服务）

| 工具类 | baseUrl模块 | 方法数 | 业务域 |
|--------|------------|--------|--------|
| LoginNetUtils | USER | 5 | 登录/三方绑定 |
| LoginNetUtilsV2 | COMMON+USER | 2 | 新版验证码+登录 |
| PersonNetUtils | USER+旧版 | 多 | 用户/追剧/播放记录/订单 |
| PersonNetUtilsV2 | USER | 4 | 新版用户信息/Token刷新/关注 |
| MovieNetUtils | MEDIA | 5 | 影片详情/剧集/推荐/状态/下载 |
| FilmLibraryNetUtils | MEDIA+HOME | 多 | 热门推荐/片库搜索/筛选 |
| HomeNetUtils | HOME+旧版 | 多 | 首页Tab/Banner/栏目 |
| ShortVideoNetUtils | ZIXUN+KANKAN | 多 | 短视频列表/评论/点赞/收藏/搜索 |
| NewsNetUtils | ZIXUN | 多 | 资讯列表/详情/点赞 |
| MovieDiscussNetUtils | COMMENT | 20+ | 影评CRUD/点赞/回复/举报/标签 |
| PayNetUtils | VIP+旧版 | 3 | 下单/支付/查询结果 |
| VipNetUtils | VIP | 1 | VIP商品列表 |
| RrCurrencyNetUtils | 旧版 | 1 | 人人币商品列表 |
| MessageNetUtils | 旧版 | 6 | 消息列表/已读/排行 |
| IDCardAuthNetUtils | USER | 1 | 实名认证 |

---

## 1. 登录模块

### 1.1 发送验证码（新版）
- **模块**: COMMON
- **方法**: `POST`
- **URL**: `v1/captcha/noa/send/sms`
- **请求**: `{ phone: String }`
- **响应**: `NewSendSmsResponse → { message: String }`

### 1.2 短信登录（新版）
- **模块**: USER
- **方法**: `POST`
- **URL**: `v1/users/noa/login/sms`
- **请求**: `{ phone: String, code: String, platform: String }`
- **响应**: `NewSmsLoginResponse → NewLoginResult`
  ```
  { token, user_id(id), phone, email, nickname(nick), avatar(headImg),
    intro, vip, vip_expired_time(vipExpiredTime), coin,
    fans_num(fansNum), attention_num(attentionNum), zan_num(zanNum),
    sex, birthday, address, forbidComment }
  ```

### 1.3 发送验证码（旧版）
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/noa/smsLogin`
- **请求**: `{ phone: String }`

### 1.4 短信登录（旧版）
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/noa/login`
- **请求**: `{ code: String, phone: String }`
- **响应**: `LoginBeanResult → LoginBean`

### 1.5 一键登录（号码认证）
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/noa/oneKeyLogin`
- **请求**: `PhoneNumRequest { md5, needLogin, opToken, operator, phoneOperator, token }`
- **响应**: `PhoneNumBeanResult → PhoneNumBean { loginResponse, phone }`

### 1.6 三方登录
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/noa/thirdPartyQuickLogin`
- **请求**: `{ thirdLoginType(0-6), accessToken, openId }`
  - 类型: 0=微信, 3=QQ, 6=苹果
- **响应**: `LoginBeanResult → LoginBean`

### 1.7 三方绑定手机号
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/noa/thirdBindPhoneLogin`
- **请求**: `{ thirdLoginType, accessToken, code, openId, phone }`

---

## 2. 用户模块

### 2.1 获取用户信息（新版）
- **模块**: USER
- **方法**: `GET`
- **URL**: `v1/users/info`
- **参数**: `userId`
- **响应**: `NewUserInfoResponse → NewUserInfoResult`

### 2.2 刷新Token（新版）
- **模块**: USER
- **方法**: `POST`
- **URL**: `v1/users/token/refresh`
- **响应**: `NewRefreshTokenResponse → NewRefreshTokenResult`

### 2.3 修改用户信息
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/modifyUserInfo`
- **请求**: `{ nick, intro, sex, birthday, address }`

### 2.4 修改头像
- **方法**: `POST (文件上传)`
- **URL**: `xzmovie2/rest/personal/modifyAvatar`
- **请求**: `file: 图片文件`

### 2.5 换绑手机
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/changePhone`
- **请求**: `{ code: String, newPhone: String }`

### 2.6 意见反馈
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/addFeedback`
- **请求**: `{ content: String, phone: String }`

### 2.7 注销账号
- **方法**: `GET`
- **URL**: `xzmovie2/rest/personal/logout`
- **响应**: `LogoutResult`

### 2.8 获取关注列表（新版）
- **模块**: USER
- **方法**: `GET`
- **URL**: `v1/users/follow/list`
- **参数**: `userId, pageNo, pageSize`
- **响应**: `NewFollowUserListResponse → [NewFollowUserItem]`

### 2.9 关注/取消关注用户
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/attention` / `deleteAttention`
- **请求**: `{ userId }`

---

## 3. 影片模块

### 3.1 获取影片详情
- **模块**: MEDIA
- **方法**: `GET`
- **URL**: `api/v1/rest/movie/detail/noa/movieDetail`
- **参数**: `movieId`
- **响应**: `MovieDetailNewResult → MovieDetailNewData`
  ```
  { id, movieName, movieNameEn, movieAlias,
    coverHorizontal/Vertical, posterHorizontal/Vertical,
    gifHorizontal/Vertical, star, starValue, movieType(Int类型),
    country, language, time, themeTags, directorTags, actorTags,
    scriptTags, description, shareDesc, shareContents,
    pictures(List), playNum, totalNum, updateNum, resolution,
    isFollow, isDownload, enable, openDiscuss, displayNowFansScore,
    offTime, movieItemLastUpdateTime,
    mainMovieItemList, previewMovieItemList,
    actorTagModeList, directorTagModeList, scriptTagModeList,
    playRecord, commentNumber, isHuashu }
  ```

### 3.2 获取剧集播放URL
- **模块**: MEDIA
- **方法**: `GET`
- **URL**: `api/v1/rest/movie/detail/noa/movieItemUrl`
- **参数**: `itemUrlId`
- **响应**: `MovieEpoUrlListNewResult → MovieUriItemInfo`

### 3.3 获取推荐列表
- **模块**: MEDIA
- **方法**: `GET`
- **URL**: `api/v1/rest/movie/detail/noa/recommend`
- **参数**: `movieId`
- **响应**: `RecommendNewResult → [RecommendNewItem]`

### 3.4 检查影片状态
- **模块**: MEDIA
- **方法**: `GET`
- **URL**: `api/v1/rest/movie/detail/noa/valid`
- **参数**: `movieId`
- **响应**: `MovieStatusNewResult → { enable: Boolean }`

### 3.5 获取全部下载链接
- **模块**: MEDIA
- **方法**: `POST`
- **URL**: `api/v1/rest/movie/detail/noa/play-url/batch`
- **请求**: `{ itemUrlIdList: List<String> }`
- **响应**: `MovieEpoUrlDownloadListResult → [MovieUriDownloadInfo]`

---

## 4. 首页模块

### 4.1 获取首页索引数据（新版）
- **模块**: HOME
- **方法**: `GET`
- **URL**: `v1/home/noa/index`
- **响应**: `HomeIndexResult → HomeIndexData { tabItems, recommendBanner, mediaGroup }`

### 4.2 获取首页Tab列表（旧版）
- **方法**: `POST`
- **URL**: `xzmovie2/rest/top/type/noa/listAllTopType`

### 4.3 获取Tab内容（旧版）
- **方法**: `POST`  
- **URL**: `xzmovie2/rest/movie/noa/listTypeData`
- **请求**: `{ topTypeId }`

---

## 5. 片库模块

### 5.1 热门推荐
- **模块**: MEDIA
- **方法**: `POST`
- **URL**: `api/v1/rest/movie/lib/noa/hot`
- **请求**: `{ ignoreMovieIdList, pageNo, pageSize }`

### 5.2 筛选条件获取
- **模块**: MEDIA
- **方法**: `GET`
- **URL**: `api/v1/rest/movie/lib/noa/searchItem`
- **参数**: `movieType`

### 5.3 按条件搜索
- **模块**: MEDIA
- **方法**: `POST`
- **URL**: `api/v1/rest/movie/lib/noa/list` (新) / `xzmovie2/rest/movie/noa/search` (旧)
- **请求**: `{ sort, type, theme, time, language, country, streamingMedia, content, pageNo, pageSize }`

### 5.4 上新列表
- **模块**: MEDIA
- **方法**: `GET`
- **URL**: `api/v1/rest/movie/lib/noa/new/movie`
- **参数**: `pageNo, pageSize`

---

## 6. 播放记录模块

### 6.1 保存播放记录
- **模块**: MEDIA
- **方法**: `POST`
- **URL**: `api/v1/rest/personal/record/play`
- **请求**: `{ movieItemId, duration }` (duration单位: 秒)

### 6.2 获取当前播放记录
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/movie/playrecord`
- **请求**: `{ movieItemId }`

### 6.3 获取播放历史列表
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/listPlayRecord`
- **请求**: `{ pageNo, pageSize }`

### 6.4 删除播放记录
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/deletePlayRecord`
- **请求**: `{ movieItemIdList }`

---

## 7. 追剧/收藏模块

### 7.1 追剧/取消追剧
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/followMovie`
- **请求**: `{ isFollow: Boolean, movieId: String }`

### 7.2 关注追剧列表（新版）
- **模块**: USER
- **方法**: `POST`
- **URL**: `v1/users/follow/movie/list`
- **请求**: `{ userId, movieType, pageNo, pageSize }`

### 7.3 删除追剧
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/deleteFollow`
- **请求**: `{ itemIds: List<String>, channel }`

### 7.4 点赞/取消点赞
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/zan`
- **请求**: `{ isZan: Boolean, movieItemId }`

---

## 8. 支付模块

### 8.1 获取VIP商品列表
- **模块**: VIP
- **方法**: `GET`
- **URL**: `v1/vip/noa/product/list`
- **响应**: `VipProductBeanResult → VipProductResult { list, isSubscribeUser, isBoughtVip }`

### 8.2 创建支付订单
- **方法**: `POST`
- **URL**: `xzmovie2/rest/order/createOrder`
- **请求**: `PayRequest { goodsId, goodsType(0=VIP), payChannelType(0=支付宝/1=微信/2=苹果IAP), payload?, transactionId? }`
- **响应**: `PayResult → PayBean { url?, tradeNo?, payload?, payParams? }`

### 8.3 查询支付结果
- **方法**: `POST`
- **URL**: `xzmovie2/rest/order/getOrderPayResult`
- **请求**: `{ tradeNo: String }`
- **响应**: `TradeRecordBeanResult → TradeDetailBean`

### 8.4 解锁剧集（消费人人币）
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/consumeMovieItem`
- **请求**: `{ movieItemIdList: List<String> }`

### 8.5 解锁全集
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/consumeMovie`
- **请求**: `{ movieId: String }`

### 8.6 获取订单列表（新版）
- **模块**: VIP
- **方法**: `GET`
- **URL**: `v1/vip/order/list`
- **参数**: `pageNo, pageSize`

---

## 9. 短视频模块

### 9.1 短视频列表
- **方法**: `POST`
- **URL**: `xzmovie2/rest/shortVideo/noa/list`
- **请求**: `{ pageNo, pageSize }`

### 9.2 短视频评论列表
- **方法**: `POST`
- **URL**: `xzmovie2/rest/shortVideo/noa/commentPage`
- **请求**: `{ shortVideoId, pageNo, pageSize }`

### 9.3 短视频评论
- **方法**: `POST`
- **URL**: `xzmovie2/rest/shortVideo/comment`
- **请求**: `{ shortVideoId, comment }`

### 9.4 短视频点赞/收藏/关注
- **方法**: `POST`
- **URL**: `xzmovie2/rest/shortVideo/zan` / `collect` / `authorAttention`
- **请求**: `{ shortVideoId/userId, status: Boolean }`

### 9.5 短视频搜索
- **方法**: `POST`
- **URL**: `xzmovie2/rest/shortVideo/noa/search`
- **请求**: `{ keyword, pageNo, pageSize }`

---

## 10. 影评与评论模块（20+接口）

> **注意**: 现已支持基于 `channel` (`short_video`/`media`/`article`/`news`) 的新版统一评论架构。

### 10.1 获取统一评论列表（新版）
- **方法**: `GET`
- **URL**: `api/v1/rest/comment/list`
- **参数**: `channel, itemId, pageNo, pageSize, sort`
- **响应**: `NewCommentListResponse → NewCommentListResult { comments, total, size, pages }`

### 10.2 获取子评论列表（新版）
- **方法**: `POST`
- **URL**: `api/v1/rest/comment/child/list`
- **请求**: `{ commentId, pageNo, pageSize }`
- **响应**: `NewChildCommentListResponse → NewChildCommentListResult`

### 10.3 发布评论（新版）
- **方法**: `POST`
- **URL**: `api/v1/rest/comment/add`
- **请求**: `PublishCommentRequest { channel, is_anonymous, item_id, content, score, parent_id }`
- **响应**: `PublishCommentResponse`

### 10.4 统一评论/动态点赞与取消（新版）
- **方法**: `POST`
- **URL**: `api/v1/rest/personal/add/zan` / `api/v1/rest/personal/delete/zan`
- **请求**: `AddZanRequest { channel, itemId }`

### 10.5 我的影评列表（新版）
- **方法**: `POST`
- **URL**: `api/v1/rest/comment/user/media/list`
- **请求**: `{ userId, pageNo, pageSize }`
- **响应**: `NewCommentUserListResponse → NewCommentUserListResult`

### 10.6 获取影片专属影评（旧版）
- **模块**: COMMENT
- **方法**: `GET`
- **URL**: `api/v1/rest/discuss/noa/list`
- **参数**: `movieId, pageNo, pageSize, sortType`

### 10.7 发表影评（旧版带图片）
- **模块**: COMMENT
- **方法**: `POST`
- **URL**: `api/v1/rest/discuss/save`
- **请求**: `{ data(JSON), images(文件数组) }` — multipart上传

### 10.8 影评举报与标签
- **URL**: `api/v1/rest/discuss/report` (举报)
- **URL**: `api/v1/rest/discuss/noa/tags` (获取标签)

---

## 11. 消息模块

### 11.1 获取消息列表
- **方法**: `POST`
- **URL**: `xzmovie2/rest/msg/listMsg`
- **请求**: `{ category, pageNo, pageSize }`

### 11.2 标记全部已读
- **URL**: `xzmovie2/rest/msg/readAllMsg`

### 11.3 排行榜列表
- **URL**: `xzmovie2/rest/rank/noa/list`

---

## 12. 实名认证

### 12.1 实名认证
- **模块**: USER
- **方法**: `POST`
- **URL**: `xzmovie2/rest/personal/addRealNameAuth`
- **请求**: `{ code, idCard, realName }`

---

## 13. 全局配置

### 13.1 获取全局配置
- **模块**: STATIC
- **方法**: `GET`
- **URL**: `v1/global_config`
- **响应**: `GlobalConfigResult → GlobalConfigBean { isGreyDisplay, forbidComment, forbidUserModify }`
