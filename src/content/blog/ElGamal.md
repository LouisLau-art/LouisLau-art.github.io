---
title: '密码学复习笔记(2)：ElGamal 加密与同态性质'
description: '从 DH 密钥交换进阶到公钥加密。为什么两个密文乘起来，解密后等于明文相乘？这是 Assignment 1 的核心考点。'
pubDate: 'Dec 15 2025'
tags: ["Archive"]
---

在上一篇笔记中，我们复习了 Diffie-Hellman (DH) 密钥交换。今天复习的内容是它的“进阶版”—— **ElGamal 加密算法**。

这是 Assignment 1 Task E 的重点，也是理解后续零知识证明的基础。简单来说，ElGamal 就是把 DH 协商出来的那个“共享秘密”，直接用来“遮盖”明文。

## 1. 算法流程：从 DH 到 ElGamal

我们还是假设公开参数：大素数 $p$ 和底数 $g$。

### 🔑 密钥生成 (KeyGen)

和 DH 一样，Alice 选一个私钥，算一个公钥。

- **私钥 (sk):** $a$
- **公钥 (pk):** $h = g^a \bmod p$

### 🔒 加密 (Encryption)

假设 Bob 要给 Alice 发消息 $m$。
Bob 不能直接用公钥 $h$ 加密，他需要引入一个随机数（Ephemeral Key）来保证每次加密的结果都不一样（IND-CPA 安全的要求）。

1. Bob 选一个随机数 $r$。
2. 计算 **$c_1 = g^r \bmod p$** （这就像 DH 里 Bob 发给 Alice 的那个公钥）。
3. 计算共享秘密 $s = h^r = (g^a)^r = g^{ar}$。
4. 用共享秘密“遮盖”明文：**$c_2 = m \cdot s = m \cdot h^r \bmod p$**。
5. **最终密文:** $(c_1, c_2)$。

### 🔓 解密 (Decryption)

Alice 收到 $(c_1, c_2)$。

1. Alice 利用私钥 $a$ 恢复共享秘密：
   $$s = c_1^a = (g^r)^a = g^{ar}$$
2. Alice 除去遮盖，还原明文：
   $$m = c_2 \cdot s^{-1} \bmod p$$

---

## 2. 乘法同态 (Multiplicative Homomorphism)

这是 **Assignment 1 Task E** 必考的性质。

**问题：** 如果我有两个密文，分别是 $m_1$ 和 $m_2$ 的加密结果。如果我把这两个密文“乘”起来，解密后会得到什么？

**推导过程：**

假设公钥是 $h$。

- 密文 1 (对应 $m_1$): $C_A = (g^{r_1}, \quad m_1 \cdot h^{r_1})$
- 密文 2 (对应 $m_2$): $C_B = (g^{r_2}, \quad m_2 \cdot h^{r_2})$

我们将它们对应部分相乘（Component-wise multiplication）：

1. **第一部分相乘:**
   $$C_{new\_1} = g^{r_1} \cdot g^{r_2} = g^{r_1 + r_2}$$

2. **第二部分相乘:**
   $$C_{new\_2} = (m_1 \cdot h^{r_1}) \cdot (m_2 \cdot h^{r_2}) = (m_1 \cdot m_2) \cdot h^{r_1 + r_2}$$

**结论：**
新的密文 $C_{new} = (g^{R}, M \cdot h^{R})$，其中 $R = r_1+r_2$， $M = m_1 \cdot m_2$。
这依然是一个合法的 ElGamal 密文！

这意味着：**$Dec(Enc(m_1) \cdot Enc(m_2)) = m_1 \cdot m_2$**。
这就是**乘法同态**性质。

---

## 3. 思考：能做成“加法同态”吗？

如果我们想要 $Dec(C_{new}) = m_1 + m_2$，该怎么办？

**Trick:** 把消息 $m$ 放在指数上。

- 修改加密算法为：$c_2 = g^m \cdot h^r$

这样乘起来时：
$$g^{m_1} \cdot g^{m_2} = g^{m_1 + m_2}$$
指数确实相加了。

**但也带来了致命缺陷 (Drawback)：**
解密时，Alice 算出的是 $g^{m_1+m_2}$。她需要从这个结果反推出 $m_1+m_2$。
这正是**离散对数问题 (DLP)**！如果 $m$ 很大，Alice 自己也解不出来。
所以这种加法同态方案只能用于 $m$ 非常小（比如投票计数）的场景。

---

## 4. 总结

1. **ElGamal** = **Diffie-Hellman** + **掩码操作**。
2. **密文结构**：$(g^r, m \cdot h^r)$，包含随机数 $r$，所以是概率性加密 (Probabilistic Encryption)。
3. **同态性质**：原版 ElGamal 是**乘法同态**的。

下一篇笔记将整理 **Oblivious Transfer (OT) 协议**，那是 MPC (多方安全计算) 的基石。