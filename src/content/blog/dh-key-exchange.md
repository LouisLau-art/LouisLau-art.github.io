---
title: '🚀密码学复习笔记(1)：Diffie-Hellman 密钥交换'
description: '期末考试复习记录。如何在一个有人偷听的房间里告诉对方秘密数字？DH 协议的核心逻辑梳理。'
pubDate: 'Dec 15 2025'
tags: ["Archive"]
---

最近正在备战 **Modern Cryptography** 期末考试。密码学中一个经典问题是：**Alice 和 Bob 如何在公共信道（可能被第三方窃听）上安全地协商一个只有他们两人知道的密钥？**  

1976年，**Whitfield Diffie** 和 **Martin Hellman** 提出了解决方案——**Diffie-Hellman 密钥交换协议**。本文梳理其核心逻辑。

---

## **1. 核心数学概念**

在深入协议之前，需掌握两个关键数学概念：

#### **(1) 模运算 (Modular Arithmetic)**  
表示将一个数除以另一个数（模数）后取余数，记作：  
> **A mod p**  

*例如：* `10 mod 3 = 1`（因为 10 ÷ 3 = 3 余 1）

#### **(2) 离散对数问题 (Discrete Logarithm Problem, DLP)**  
给定：  
- 大素数 **p**  
- 底数 **g**  
- 值 **A = gᵃ mod p**  

**求解指数 *a* 极其困难**，即使已知 *g*, *p*, *A*。  

> ✅ **这是 DH 协议安全性的基石**。当 *p* 足够大（如 2048 位）时，当前计算机无法在合理时间内破解。

---

## **2. Diffie-Hellman 协议流程**

**前提：** Alice 与 Bob 预先约定两个**公开参数**：  
- 一个大素数 **p**  
- 一个底数 **g**（通常为 *p* 的原根）

### **协议步骤（流程图解）**

| 步骤 | Alice (发送方)                     | Bob (接收方)                       | Eve (窃听者)          |
|------|-------------------------------------|-------------------------------------|-----------------------|
| **1. 选择私钥** | 私密选择随机整数 **a** <br> *(仅 Alice 知道)* | 私密选择随机整数 **b** <br> *(仅 Bob 知道)* | **无信息** |
| **2. 计算公钥** | 计算公钥：**A = gᵃ mod p** | 计算公钥：**B = gᵇ mod p** | ... |
| **3. 交换公钥** | **将 A 发送给 Bob** | **将 B 发送给 Alice** | **截获 A 和 B** |
| **4. 计算共享密钥** | 用 Bob 的公钥计算：**S = Bᵃ mod p** | 用 Alice 的公钥计算：**S = Aᵇ mod p** | **无法计算 S** 😭 |

> 💡 **关键点**：  
> - Alice 计算的 *S = Bᵃ mod p = (gᵇ mod p)ᵃ mod p = gᵃᵇ mod p*  
> - Bob 计算的 *S = Aᵇ mod p = (gᵃ mod p)ᵇ mod p = gᵃᵇ mod p*  
> **两者得到的 *S* 完全相同！**  

---

## **3. Python 代码演示**

以下代码模拟 DH 协议过程（使用小质数 *p=23* 便于演示）：

```python
def power(base, exp, mod):
    """快速幂算法：计算 (base^exp) % mod"""
    res = 1
    base = base % mod
    while exp > 0:
        if exp % 2 == 1:
            res = (res * base) % mod
        base = (base * base) % mod
        exp //= 2
    return res

# 0. 设置公开参数
p = 23   # 大素数（实际使用需为 2048 位以上）
g = 5    # 底数

print(f"[*] 公开参数: p = {p}, g = {g}")

# 1. Alice 和 Bob 各自选择私钥（保密）
a_private = 6   # Alice 的私钥
b_private = 15  # Bob 的私钥

# 2. 计算公钥
A_public = power(g, a_private, p)  # A = g^a mod p
B_public = power(g, b_private, p)  # B = g^b mod p

print(f"[*] Alice 发送公钥 A = {A_public}")
print(f"[*] Bob   发送公钥 B = {B_public}")

# 3. 计算共享密钥
s_alice = power(B_public, a_private, p)  # S = B^a mod p
s_bob   = power(A_public, b_private, p)  # S = A^b mod p

print(f"[-] Alice 计算的共享密钥: {s_alice}")
print(f"[-] Bob   计算的共享密钥: {s_bob}")

# 验证密钥是否一致
if s_alice == s_bob:
    print("[✅] 密钥协商成功！两人拥有相同的共享密钥。")
else:
    print("[❌] 密钥协商失败！")
```

**运行结果示例**：  
```
[*] 公开参数: p = 23, g = 5
[*] Alice 发送公钥 A = 8
[*] Bob   发送公钥 B = 19
[-] Alice 计算的共享密钥: 2
[-] Bob   计算的共享密钥: 2
[✅] 密钥协商成功！两人拥有相同的共享密钥。
```

> ⚠️ **注意**：实际应用中 *p* 必须是**大素数（≥2048 位）**，否则协议不安全。

---

## **4. 为什么 Diffie-Hellman 是安全的？**

**攻击者 Eve 的困境**：  
Eve 截获了公开参数 *p*, *g* 及双方的公钥 *A* 和 *B*。  
她想计算共享密钥 *S = gᵃᵇ mod p*，**必须先求解 *a* 或 *b***。  

**即需解方程**：  
> *gˣ ≡ A (mod p)*  

这正是 **离散对数问题 (DLP)**！  

- **当 *p* 很小时**（如本例 *p=23*），Eve 可通过穷举快速破解。  
- **当 *p* 足够大**（如 2048 位），**当前没有已知算法能在合理时间内破解**。  
  → **即使 Eve 拦截所有通信，也无法得知共享密钥 *S***。

> 🔒 **安全假设**：*计算离散对数困难*。

---

## **复习总结**

- **DH 协议** 完美解决了**在不安全信道上安全协商密钥**的问题。  
- **核心思想**：利用**模幂运算**与**离散对数难题**，确保双方独立计算出相同的共享密钥，而第三方无法推导。  
- **应用基础**：DH 是 **ElGamal 加密**、**Schnorr 签名** 及现代 TLS/SSL 协议（如 ECDHE）的理论基础。  

**下一步**：深入学习基于 DH 的公钥加密方案及数字签名！ 💪