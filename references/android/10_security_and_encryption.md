# 10 安全与加密

## 安全架构总览

```
客户端安全层
├── mTLS 双向认证      → 客户端证书 + 私钥（Arkana混淆存储）
├── NDK 加密层         → C++ 混淆存储密钥/IV/加密AppId
├── Token 认证         → Bearer Token + UserID Header
├── 证书固定           → 自定义 TrustManager
└── 崩溃防护           → 第三方SDK崩溃拦截
```

---

## 1. mTLS 双向认证 (ClientCertificateHelper)

### 工作原理

客户端在 TLS 握手时向服务端出示客户端证书，实现双向身份校验：

```
Client                          Server
  |── ClientHello ──────────────→|
  |←── ServerHello + ServerCert ─|
  |←── CertificateRequest ──────|  ← 服务端要求客户端证书
  |── ClientCertificate ────────→|  ← 客户端出示证书
  |── CertificateVerify ────────→|  ← 客户端用私钥签名证明身份
  |── Finished ─────────────────→|
  |←── Finished ─────────────────|
```

### 证书来源

证书和私钥通过 **Arkana** 工具混淆后编译进应用：

- 配置文件: `.arkana.yml`
- 原始值存储: `.env` 文件（不提交到版本库）
- 编译后: `Secrets.Global.cLIENT_CERTIFICATE` / `Secrets.Global.cLIENT_PRIVATE_KEY`

### 证书信息

| 项目 | 值 |
|------|-----|
| 证书类型 | X.509 |
| 私钥算法 | RSA 2048-bit |
| 编码格式 | PEM (Base64) |
| 域名匹配 | `*.yyets.com.cn` |
| 有效期 | 2026-03-02 ~ 2036-02-28 |
| 颁发者 | 自签名（rrys 内部 CA） |
| KeyStore 密码 | `rrys2026` |
| Key 别名 | `client` |

### 实现流程

```
1. 从 Arkana Secrets 获取 PEM 文本
2. 将 \\n 转换为真正的换行符
3. 解析 X.509 证书
4. 解析 PKCS8 格式的 RSA 私钥
5. 创建 KeyStore，将证书和私钥存入
6. 初始化 KeyManagerFactory
7. 初始化 TrustManagerFactory（使用系统默认信任链）
8. 创建 SSLContext (TLS协议)
9. 为 OkHttpClient 配置 sslSocketFactory
```

### 容错

证书配置失败时不会崩溃，回退到默认无客户端证书的 HTTPS 连接。

---

## 2. NDK 加密层 (security.cpp)

### 编译配置

- CMake 3.18.1
- 项目名: `security`
- 输出: `libsecurity.so` (共享库)
- 路径: `app/src/main/cpp/`

### JNI 接口

| Java 方法 | JNI 函数 | 说明 |
|-----------|---------|------|
| `SecurityHelper.getEncryptedAppIdBytesNative(env: String)` | `Java_com_chuanying_xianzaikan_utils_SecurityHelper_getEncryptedAppIdBytesNative` | 获取加密的 AppId 字节数组 |
| `SecurityHelper.getKeyStringNative()` | `Java_com_chuanying_xianzaikan_utils_SecurityHelper_getKeyStringNative` | 获取 AES 密钥 |
| `SecurityHelper.getIVStringNative()` | `Java_com_chuanying_xianzaikan_utils_SecurityHelper_getIVStringNative` | 获取 AES IV 向量 |

### 密钥存储

AES 密钥和 IV 以混淆的 char 数组形式存储在 native 层，只有通过 JNI 调用才能获取：

- **KEY**: 由 `KEY_PART_1` + `KEY_PART_2` 拼接（32字符）
- **IV**: 由 `IV_PART_1` + `IV_PART_2` 拼接（20字符）

### 加密 AppId

AppId 以加密后的字节数组存储，按环境区分：

| 环境 | 加密数据长度 | 校验和 |
|------|------------|--------|
| prod | 16字节 | 0x28 |
| dev | 16字节 | 0xB1 |

### 安全特性

1. **数据完整性校验**: 使用 XOR 校验和验证数据未被篡改
2. **参数校验**: null 检查防止 JNI 崩溃
3. **内存检查**: ByteArray 分配失败时抛出 OOM 异常
4. **异常检测**: SetByteArrayRegion 后检查 JNI 异常
5. **默认值**: 未知环境默认使用 prod 配置

---

## 3. Token 认证机制

### 认证头注入

在 OkHttp 拦截器中自动注入：

```
Authorization: Bearer {token}
C-ACCESS-TOKEN: {token}
movienow-userid: {userId}
```

### 401 处理

详见 02_network_layer.md 的 "401 未授权处理流程" 章节。

### Token 存储

- **内存**: `UserInfoConst.token` 全局变量
- **持久化**: `SharedPreferencesUtils.token` 字段
- **清除**: `UserInfoConst.clearUserData()` 同时清除内存和SP

---

## 4. 签名配置

| 配置项 | 值 |
|--------|-----|
| 签名文件 | `rrzjKeystore.jks`（仓库根目录） |
| Key 别名 | `key0` |
| Store 密码 | `@#yjkj$%` |
| Key 密码 | `@#yjkj$%` |
| ABI | armeabi-v7a, arm64-v8a |

Release 和 Debug 使用相同签名配置。

---

## 5. 第三方 SDK 崩溃防护

```kotlin
// 自定义 UncaughtExceptionHandler
// 拦截条件: NullPointerException + message 包含 "interceptor" 和 "returned null"
// 处理: 记录日志但不崩溃（离线状态下第三方SDK的已知问题）
// 其他崩溃: 交给 Bugly 处理
```

---

## 6. 防盗链

README 中记录的腾讯云点播防盗链密钥: `93YOLwJHewGxMsNM8jKO`

（注: 当前播放器已迁移到火山引擎，此密钥可能仅用于旧内容兼容）

---

## 7. 复刻要点

| 安全机制 | 复刻方式 |
|---------|---------|
| mTLS | 新平台需配置相同的客户端证书和私钥 |
| NDK加密 | 需要在目标平台实现等效的安全存储（如 iOS Keychain） |
| AES 加解密 | 需要获取完整的 KEY/IV 值实现对应的 AES 解密 |
| Token 认证 | 按 02_network_layer 中的 Header 规范实现 |
| 签名 | 每个平台使用自己的签名机制 |
