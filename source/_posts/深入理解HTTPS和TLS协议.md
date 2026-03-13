---
title: 深入理解 HTTPS 和 TLS 协议
date: 2024-05-01
tags:
  - HTTPS
  - TLS
  - 网络安全
cover: https://images.unsplash.com/photo-1563013544-824ae1b704d3?w=800
---

HTTPS 是 HTTP 的安全版本，通过 TLS/SSL 协议加密传输数据。本文深入解析 HTTPS 的工作原理和 TLS 握手过程。

## 一、HTTPS 概述

### 为什么需要 HTTPS？

- **机密性**：防止数据被窃听
- **完整性**：防止数据被篡改
- **身份认证**：验证服务器身份

### HTTPS  vs HTTP

```
HTTP: TCP → HTTP
HTTPS: TCP → TLS → HTTP
```

## 二、TLS 协议详解

### TLS 1.2 握手过程

```
ClientHello                  →   支持的密码套件、随机数
                      ←   ServerHello
                           选择的密码套件、随机数
                      ←   服务器证书
                      ←   (可选) 服务器密钥交换
                      ←   (可选) 请求客户端证书
CertificateRequest          →
                      ←   ServerHelloDone
ClientKeyExchange           →   预主密钥（用公钥加密）
CertificateVerify           →   (可选) 签名
ChangeCipherSpec            →   加密算法确定
Finished                    →   加密的握手消息
                      ←   ChangeCipherSpec
                      ←   Finished
```

### 密钥交换算法

#### 1. RSA 密钥交换

```bash
# 客户端生成随机预主密钥，用服务器公钥加密后发送
PremasterSecret = random()
EncryptedPremasterSecret = RSA_Encrypt(PremasterSecret, ServerPublicKey)
```

#### 2. ECDHE 密钥交换（推荐）

```bash
# 使用椭圆曲线Diffie-Hellman密钥交换
# 支持前向安全性（Forward Secrecy）
ECDHE_RSA: 临时ECDHE密钥，RSA签名
ECDHE_ECDSA: 临时ECDHE密钥，ECDSA签名
```

### 密码套件示例

```
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
  ├── ECDHE: 密钥交换
  ├── RSA: 身份验证
  ├── AES_256_GCM: 对称加密
  └── SHA384: 消息认证
```

## 三、证书机制

### 证书链

```
Root CA (根证书)
  ↓
Intermediate CA (中间证书)
  ↓
Server Certificate (服务器证书)
```

### 证书验证过程

1. **有效性检查**：有效期、域名匹配
2. **吊销检查**：CRL / OCSP
3. **链验证**：逐级验证到根CA

### 自签名证书

```bash
# 生成私钥
openssl genrsa -out server.key 2048

# 生成CSR
openssl req -new -key server.key -out server.csr

# 自签名证书
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

## 四、TLS 1.3 改进

### 主要改进

1. **更快**：1-RTT握手（之前需要2-RTT）
2. **更安全**：移除不安全的密码套件
3. **简化**：密码套件减少到5个

### TLS 1.3 握手

```
ClientHello (ClientKeyShare)     →
                            ← ServerHello (ServerKeyShare, Certificate)
                            ← Finished
ClientKeyExchange, Finished      →
```

### 0-RTT 模式

```
ClientHello (EarlyData)         →
                            ← Finished (EarlyData)
```

⚠️ 注意：0-RTT模式存在重放攻击风险

## 五、Nginx 配置 HTTPS

```nginx
server {
    listen 443 ssl http2;
    
    ssl_certificate /path/to/server.crt;
    ssl_certificate_key /path/to/server.key;
    
    # TLS版本
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # 密码套件
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000" always;
    
    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
}
```

## 六、常见安全问题

### 1. 心脏出血（Heartbleed）

- OpenSSL 1.0.1-1.0.1f 漏洞
- 攻击者读取服务器内存

### 2. POODLE

- SSLv3 漏洞
- 禁用SSLv3即可防御

### 3. FREAK

- 出口限制导致的弱加密
- 确保不导出弱密钥

## 七、总结

| TLS版本 | 状态 | 特性 |
|---------|------|------|
| SSLv3 | 废弃 | 不安全 |
| TLS 1.0 | 废弃 | 不安全 |
| TLS 1.1 | 废弃 | 不安全 |
| TLS 1.2 | 推荐 | 安全、广泛支持 |
| TLS 1.3 | 推荐 | 更安全、更快 |

**最佳实践**：
- 使用 TLS 1.2 或 1.3
- 启用 HSTS
- 使用强密码套件
- 开启 OCSP Stapling
- 配置安全的重定向
