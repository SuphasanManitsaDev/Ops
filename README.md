# 🛡️ Vault Quickstart with KV Secrets, Policy, and AppRole (ภาษาไทย)

คู่มือนี้ช่วยให้คุณตั้งค่า HashiCorp Vault ในโหมด Dev พร้อมใช้งาน Key-Value Secrets, กำหนดสิทธิ์ผ่าน Policy และ Login ด้วย AppRole แบบปลอดภัย

---

## 📦 1. ดูว่า Vault มี Secret Engine อะไรอยู่บ้าง

```bash
vault secrets list
```

### 🔍 ตัวอย่างผลลัพธ์

```
Path          Type       Description
----          ----       -----------
cubbyhole/    cubbyhole  per-token secret
secret/       kv         key/value secret storage
ooo/          kv         app-specific secret store
```

---

## 📁 2. ลิสต์ข้อมูลใน Secret Engine (Key-Value)

```bash
vault kv list secret/
vault kv list ooo/
```

### 🔍 ตัวอย่างผลลัพธ์

```
Keys
----
myapp/
mykey
```

---

## 📝 3. สร้างและอ่าน Secret

```bash
vault kv put secret/mykey foo=bar
vault kv get secret/mykey
```

### 🔍 ตัวอย่างผลลัพธ์

```
====== Metadata ======
created_time     2025-05-19T10:00:00Z
version          1

====== Data ======
Key     Value
---     -----
foo     bar
```

---

## 🛡️ 4. สร้าง Policy เพื่อควบคุมการเข้าถึง

สร้างไฟล์ชื่อ `myapp-policy.hcl`:

```hcl
path "secret/data/myapp/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}
```

จากนั้นเขียนเข้า Vault:

```bash
vault policy write myapp-policy myapp-policy.hcl
```

✅ สิทธิ์ที่กำหนด: ให้สามารถทำทุกอย่างกับ secret/data/myapp/*

---

## 🔐 5. เปิดและตั้งค่า Auth แบบ AppRole

```bash
vault auth enable approle
```

สร้าง Role และผูกกับ policy:

```bash
vault write auth/approle/role/myapp-role \
  token_policies="myapp-policy" \
  token_ttl=1h \
  token_max_ttl=4h
```

---

## 🆔 6. ดึง Role ID และสร้าง Secret ID

```bash
vault read auth/approle/role/myapp-role/role-id

vault write -f auth/approle/role/myapp-role/secret-id
```

---

## 🎫 7. Login ด้วย AppRole

```bash
vault write auth/approle/login \
  role_id="<role_id>" \
  secret_id="<secret_id>"
```

ผลลัพธ์: จะได้ Token (`client_token`) สำหรับใช้งาน

---

## ✅ 8. ใช้ Token เพื่อเข้าถึง Secret

```bash
export VAULT_TOKEN=s.xxxxxx

vault kv put secret/myapp/demo foo=bar
vault kv get secret/myapp/demo
```

---

## 🔄 9. ต่ออายุ Token

```bash
vault token renew
```

---

## 🚫 10. ยกเลิก Token

```bash
vault token revoke s.xxxxxx
```

---

## 📋 11. ดู policy ที่มีอยู่

```bash
vault policy list
```

## 🧾 12. อ่านเนื้อหา policy

```bash
vault policy read myapp-policy
```

---

## 🗑️ 13. ลบ policy หรือ role

```bash
vault policy delete myapp-policy
vault delete auth/approle/role/myapp-role
```

---

## ⚙️ 14. สรุป Flow การทำงาน

```
               [Vault Server - Dev Mode]
                         |
                 -------------------
                 |                 |
          [Secret Engine]     [Auth Method]
                 |                 |
          secret/myapp/*     approle/myapp-role
                 \                 /
                  \_______________/
                         |
                Login with Role ID + Secret ID
                         |
                    Get Token 🪪
                         |
             Use token to access secret 🔐
```

---

## 🧠 สรุปภาพรวม: เข้าใจการทำงานของ Vault ใน 5 ขั้นตอน

> ✅ "Secret → Policy → Role → Token → Access"

### 🗂️ 1. Secret
- ข้อมูลลับ เช่น `foo=bar`, `API_KEY=123`
- ถูกเก็บไว้ใน KV Engine เช่น `secret/myapp/api-key`

### 🛡️ 2. Policy
- กำหนดสิทธิ์ในการเข้าถึง path เช่น `secret/data/myapp/*`
- ระบุว่า read/write ได้ไหม, list ได้หรือเปล่า

### 🎭 3. Role
- ตัวแทนผู้ใช้หรือแอป (ใช้ Auth Method เช่น AppRole)
- ผูกกับ policy เพื่อควบคุมว่า role นี้มีสิทธิ์อะไร

### 🔑 4. Login ด้วย Role ID + Secret ID
- ใช้คู่ `role_id` และ `secret_id` เพื่อขอ Token
- Token ที่ได้มีสิทธิ์ตาม policy

### 🔐 5. ใช้ Token เข้าถึง Secret
- ใช้ token กับคำสั่งเช่น `vault kv get secret/myapp/api-key`
- Token มีอายุ (TTL) และสามารถ renew หรือ revoke ได้

---

✅ สรุปแบบเข้าใจง่าย:
> "สร้าง Secret → สร้าง Policy → ผูก Policy กับ Role → Login ด้วย Role ID + Secret ID → ได้ Token มาใช้เข้าถึง Secret"
