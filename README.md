<div align="center">

# Syntropy POW API

[English](./README.md) | [简体中文](./README.ZH_HANS.md)

---

A Proof of Work (POW) API where users earn coffee rewards by solving SHA-512 challenges.
</div>

## Features

- 🔐 **Account System**: Register and login with password/OTP
- 🍪 **Cookie-based Auth**: Auto-login with persistent cookies (400 days)
- ⛏️ **POW Mining**: Get UUIDs and solve SHA-512 challenges
- ☕ **Coffee Rewards**: Earn exponential rewards based on leading zeros

---

## API Endpoints

### POST /reg
Register a new account.

**Request:** No body

**Response:**
```json
{
  "success": 1,
  "aid": "uuid",
  "password": "hashed_password",
  "otp": "hashed_otp",
  "message": "Account created! Cookie set for auto-login"
}
```

> ⚠️ Save your password and OTP. They are only returned once!

---

### POST /login
Login and set authentication cookie.

**Supported login methods:**

| Method | Parameters | Cookie Set |
|--------|-----------|------------|
| Basic | `aid` | `aid` |
| Password | `aid` + `password` | `aid` + `password` |
| Full | `aid` + `password` + `otp` | `aid` + `password` + `otp` |

**Request:**
```
aid=uuid&password=hashed_password&otp=hashed_otp
```

**Response:**
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
Clear authentication cookies.

**Request:** No body

**Response:**
```json
{
  "success": 1,
  "message": "Logout success! Cookie cleared"
}
```

---

### GET /issue
Get UUIDs for mining. Aid is auto-read from cookie.

**Query Parameters:**
- `n` (optional): Number of UUIDs (max 25, default 10)

**Response:**
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
Submit proof of work and earn coffee rewards.

**Request:**
```
powdata=SYNPOW;{uuid};{aid};{nonce}
```

**Response:**
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
Get account status. Aid is auto-read from cookie.

**Response:**
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

## Reward Calculation

```
reward = 2 ^ leading_zeros
```

1. Hash your data: `SHA512(SYNPOW;{uuid};{aid};{nonce})`
2. Count leading zeros in the hash
3. Reward = 2^zero_count

### Examples

| Leading Zeros | Reward |
|---------------|--------|
| 2 | 4 coffee |
| 3 | 8 coffee |
| 4 | 16 coffee |
| 5 | 32 coffee |
| 6 | 64 coffee |

**Minimum requirement:** At least 2 leading zeros

---

## Quick Start

### 1. Register
```bash
curl -X POST https://api.com/reg -c cookies.txt
```

### 2. Login (optional)
```bash
curl -X POST https://api.com/login \
  -d "aid=uuid&password=hash&otp=hash" \
  -c cookies.txt
```

### 3. Get UUIDs
```bash
curl https://api.com/issue?n=5 -b cookies.txt
```

### 4. Mine POW (Client-side)
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

### 5. Submit POW
```bash
curl -X POST https://api.com/brew \
  -d "powdata=SYNPOW;uuid;aid;12345" \
  -b cookies.txt
```

### 6. Check Status
```bash
curl https://api.com/stat -b cookies.txt
```
