<div align="center">

# Syntropy POW API
通过解决 SHA-512 挑战来赚取咖啡奖励的 Proof of Work (POW) API。

[English](./README.md) | [简体中文](./README.ZH_HANS.md)
</div>

## 功能特点

- 🔐 **账户系统**：使用密码/OTP 注册和登录
- 🍪 **Cookie 认证**：持久化 Cookie 自动登录（400天）
- ⛏️ **POW 挖矿**：获取 UUID 并解决 SHA-512 挑战
- ☕ **咖啡奖励**：基于前导零数量获得指数级奖励

---

## API 端点
> ⚠️ POST body数据格式应为URLSearchParams，不是JSON
### POST /reg
注册新账号，并将账号完整信息保存到cookie

**请求：** 无请求体

**响应：**
```json
{
  "success": 1,
  "aid": "uuid",
  "password": "hashed_password",
  "otp": "hashed_otp",
  "message": "Account created! Cookie set for auto-login"
}
```

> ⚠️ 请保存好你的密码和 OTP，它们只返回一次！

---

### POST /login
登录并设置认证 Cookie。

**支持的登录方式：**

| 方式 | 参数 | Cookie 内容 |
|------|------|------------|
| 基础 | `aid` | `aid` |
| 密码 | `aid` + `password` | `aid` + `password` |
| 完整 | `aid` + `password` + `otp` | `aid` + `password` + `otp` |

**请求：**
```
aid=uuid&password=hashed_password&otp=hashed_otp
```

**响应：**
```json
{
  "success": 1,
  "aid": "uuid",
  "has_password": true,
  "has_otp": true,
  "message": "Login success! Cookie set with: aid, password, otp"
}
```

---

### POST /logout
清除认证 Cookie。

**请求：** 无请求体

**响应：**
```json
{
  "success": 1,
  "message": "Logout success! Cookie cleared"
}
```

---

### GET /issue
获取用于挖矿的 UUID。aid 自动从 Cookie 读取。

**查询参数：**
- `n`（可选）：UUID 数量（最大 25，默认 10）
- `aid`（或从cookie自动获取）：账号id

**响应：**
```json
{
  "success": 1,
  "uuids": ["uuid1", "uuid2", ...],
  "total_unsolved": 10,
  "prefix": "SYNPOW",
  "message": "SHA512(`{prefix};{one of uuids};{your account id};{nonce}`), reward = 2^leading_zeros coffee"
}
```

---

### POST /brew
提交工作量证明并获得咖啡奖励。

**请求：**
```
powdata=SYNPOW;{uuid};{aid};{nonce}&aid={账号id} #或从cookie获取
```

**响应：**
```json
{
  "success": 1,
  "coffee": 16,
  "reward": 16,
  "leading_zeros": 4,
  "exponent_base": 2,
  "message": "☕ Coffee brewed successfully! +16 coffee (2^4)",
  "pow_hash": "0000a1b2c3d4..."
}
```

---

### GET /stat
获取账户状态。aid 自动从 Cookie 读取。

**查询参数：**
- `aid`（或从cookie自动获取）：账号id

**响应：**
```json
{
  "success": 1,
  "aid": "uuid",
  "coffee": 16,
  "unsolved_count": 5,
  "unsolved_uuids": ["uuid1", "uuid2", ...]
}
```

---

## 奖励计算

```
奖励 = 2 ^ 前导零数量
```

1. 计算哈希：`SHA512(SYNPOW;{uuid};{aid};{nonce})`
2. 统计哈希值开头的零的数量
3. 奖励 = 2^前导零数量

### 示例

| 前导零数量 | 奖励 |
|-----------|------|
| 2 | 4 杯咖啡 |
| 3 | 8 杯咖啡 |
| 4 | 16 杯咖啡 |
| 5 | 32 杯咖啡 |
| 6 | 64 杯咖啡 |

**最低要求：** 至少 2 个前导零

---

## 快速开始

### 1. 注册
```bash
curl -X POST https://api.com/reg -c cookies.txt
```

### 2. 登录（可选）
```bash
curl -X POST https://api.com/login \
  -d "aid=uuid&password=hash&otp=hash" \
  -c cookies.txt
```

### 3. 获取 UUID
```bash
curl https://api.com/issue?n=5 -b cookies.txt
```

### 4. 挖矿（客户端）
```javascript
async function mine(uuid, aid) {
  let nonce = 0;
  while (true) {
    const data = `SYNPOW;${uuid};${aid};${nonce}`;
    const hash = await SHA512(data);
    const zeros = countLeadingZeros(hash);
    if (zeros >= 2) {
      return { powdata: data, hash, zeros };
    }
    nonce++;
  }
}

const result = await mine(uuid, aid);
// result.powdata = "SYNPOW;uuid;aid;12345"
```

### 5. 提交 POW
```bash
curl -X POST https://api.com/brew \
  -d "powdata=SYNPOW;uuid;aid;12345" \
  -b cookies.txt
```

### 6. 查看状态
```bash
curl https://api.com/stat -b cookies.txt
```
