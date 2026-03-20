# 05 数据库 Schema

## 数据库信息

| 项目 | 值 |
|------|-----|
| ORM 框架 | GreenDao 3.x |
| 数据库名 | `xianzaicache.db` |
| 当前版本 | 3 |
| 实体表数 | 2 |

---

## 表 1: CACHE_INFO_BEAN（下载缓存信息）

存储离线下载的影片缓存元数据。

### 字段定义

| 序号 | 列名 | 类型 | 约束 | 说明 |
|------|------|------|------|------|
| 0 | `_id` | INTEGER | PK, AUTOINCREMENT | 自增主键 |
| 1 | `MOVIE_NAME` | TEXT | nullable | 影片名称 |
| 2 | `MOVIE_ITEM_NAME` | TEXT | nullable | 剧集名称 |
| 3 | `SORT_NUM` | INTEGER | NOT NULL | 排序序号 |
| 4 | `DOWNLOAD_STATE` | INTEGER | NOT NULL | 下载状态（参考 Constants.STATE_*） |
| 5 | `MOVIE_ID` | TEXT | nullable | 影片ID |
| 6 | `MOVIE_ITEM_ID` | TEXT | nullable | 剧集ID |
| 7 | `IS_DOWNLOAD` | INTEGER | NOT NULL | 是否已下载 |
| 8 | `HORIZONTAL_COVER` | TEXT | nullable | 横版封面URL |
| 9 | `MOVIE_TYPE` | INTEGER | NOT NULL | 影片类型（0电影/1电视剧/2动漫/3综艺/4纪录片/5短剧） |
| 10 | `MOVIE_VIDEO_TYPE` | TEXT | nullable | 清晰度（如"1080P"） |
| 11 | `MOVIE_URL` | TEXT | nullable | 下载URL |
| 12 | `DOWNLOAD_PROGRESS` | INTEGER | NOT NULL | 下载进度(%) |
| 13 | `DOWNLOAD_SPEED` | INTEGER | NOT NULL | 下载速度 |
| 14 | `SIZE` | INTEGER(long) | NOT NULL | 文件大小(字节) |
| 15 | `PLAY_PATH` | TEXT | nullable | 本地播放路径 |
| 16 | `WATCH_PROGRESS` | INTEGER | NOT NULL | 观看进度(秒) |
| 17 | `FILM_LENGTH` | INTEGER | NOT NULL | 影片时长(秒) |
| 18 | `THEME_TAGS` | TEXT | nullable | 题材标签 |
| 19 | `TIME` | TEXT | nullable | 年代 |
| 20 | `DOWNLOAD_NUM` | INTEGER | NOT NULL | 下载集数 |
| 21 | `PRICE_TYPE` | TEXT | nullable | 收费类型（v2新增） |
| 22 | `IS_PURCHASED` | INTEGER | NOT NULL, DEFAULT 0 | 是否购买（v2新增） |
| 23 | `MOVIE_ITEM_URL_ID` | TEXT | nullable | 播放URL ID（v3新增） |

### 索引

```sql
CREATE UNIQUE INDEX IDX_CACHE_INFO_BEAN_MOVIE_ID_MOVIE_ITEM_ID 
  ON CACHE_INFO_BEAN (MOVIE_ID ASC, MOVIE_ITEM_ID ASC);
```

> 每部影片每集只保存一条记录。

---

## 表 2: DAN_MU_BEAN（弹幕缓存）

缓存影片的弹幕数据，避免每次播放都重新请求。

### 字段定义

| 序号 | 列名 | 类型 | 约束 | 说明 |
|------|------|------|------|------|
| 0 | `_id` | INTEGER | PK, AUTOINCREMENT | 自增主键 |
| 1 | `MOVIE_ID` | TEXT | nullable | 影片ID |
| 2 | `LIST` | TEXT | nullable | 弹幕数据（JSON序列化的 `List<DanmukuData>`） |

### 类型转换器

`LIST` 字段使用 `ListConverter` 进行 JSON ↔ List 转换：
- 存储: `List<DanmukuData>` → JSON String
- 读取: JSON String → `List<DanmukuData>`

---

## 版本迁移历史

| 版本 | 变更 |
|------|------|
| v1 | 初始版本，创建 `CACHE_INFO_BEAN` 和 `DAN_MU_BEAN` |
| v2 | `CACHE_INFO_BEAN` 新增 `PRICE_TYPE`(TEXT) 和 `IS_PURCHASED`(INTEGER DEFAULT 0) |
| v3 | `CACHE_INFO_BEAN` 新增 `MOVIE_ITEM_URL_ID`(TEXT) |

### 迁移 SQL

```sql
-- v1 → v2
ALTER TABLE CACHE_INFO_BEAN ADD COLUMN PRICE_TYPE TEXT;
ALTER TABLE CACHE_INFO_BEAN ADD COLUMN IS_PURCHASED INTEGER NOT NULL DEFAULT 0;

-- v2 → v3
ALTER TABLE CACHE_INFO_BEAN ADD COLUMN MOVIE_ITEM_URL_ID TEXT;
```

---

## 数据库初始化

在 `MovieApplication.onCreate()` 中初始化：

```
CacheOpenHelper(context, "xianzaicache.db")
  → DaoMaster(database)
    → DaoSession
      ├── CacheInfoBeanDao
      └── DanMuBeanDao
```

### 使用方式

```
// 获取 DaoSession
val daoSession = MovieApplication.getDaoSession()

// 获取 Dao
val cacheDao = daoSession.cacheInfoBeanDao
val danmuDao = daoSession.danMuBeanDao

// 查询
cacheDao.queryBuilder()
    .where(CacheInfoBeanDao.Properties.MovieId.eq("xxx"))
    .list()

// 插入或更新
cacheDao.insertOrReplace(cacheInfoBean)

// 删除
cacheDao.delete(entity)
```

---

## 其他持久化方式

除 GreenDao 外，项目还使用：

| 方式 | 用途 | 详见 |
|------|------|------|
| SharedPreferences | 用户状态/设置/配置 | 09_config_and_constants.md |
| SpUtil (live模块) | 直播相关配置 | CommonAppConfig.java |
| 文件系统 | 下载的视频文件 | 见文件路径常量 |
