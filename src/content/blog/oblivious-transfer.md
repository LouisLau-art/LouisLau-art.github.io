---
title: '密码学复习笔记(3)：不经意传输 (OT) 协议'
description: 'Assignment 1 Task D 解析。如何在互不信任的情况下完成数据交换？这是 MPC 和投票协议的基石。'
pubDate: 'Dec 15 2025'
tags: ["Archive"]
---

在搞定了 ElGamal 之后，我们进入了密码学的高级领域。今天复习的重点是 **Oblivious Transfer (不经意传输，简称 OT)**。

这是一个非常反直觉的协议，也是 **Assignment 1 Task D** 的核心内容，更是 Assignment 2 后面多方计算 (MPC) 的基础。

## 1. 什么是 OT？

OT 解决的是“互不信任”的隐私问题。

**经典场景 (1-out-of-2 OT):**

- Alice 有两个秘密消息 $m_0$ 和 $m_1$。
- Bob 想看其中一个（比如 $m_b$），但他不想让 Alice 知道他看的是哪一个。
- Alice 也不想让 Bob 顺手把另一个消息也看了。

**Assignment 1 Task D 的场景 (Oblivious Evaluation):**

- Alice 有输入 $m$。
- Bob 有密钥 $k$。
- Alice 想要计算 $F(k, m) = H(m)^k$。
- **隐私要求：** Bob 不能知道 $m$ 是什么，Alice 不能知道 $k$ 是什么。

## 2. 协议推导 (Assignment 1 Task D)

这个题目本质上是利用 Diffie-Hellman 的指数运算规则来实现“盲化”计算。

### 角色设定

- **Alice:** 拥有消息 $m$。
- **Bob:** 拥有密钥 $k$。
- **目标:** Alice 获得 $H(m)^k$。

### 交互流程

1. **Alice (盲化/加锁):**
   Alice 选一个随机数 $\rho$ (rho)，把自己的消息 $H(m)$ 藏起来：
   $$ \hat{m} = H(m) \cdot g^\rho \bmod p $$
   Alice 把 $\hat{m}$ 发给 Bob。
   *注：Bob 只有 $k$，解不开 $\rho$，所以他不知道 $m$。*

2. **Bob (计算):**
   Bob 收到 $\hat{m}$ 后，用自己的密钥 $k$ 对所有东西进行指数运算。
   他计算两个值发回给 Alice：
   
   - $u = g^k$ (底数的加密)
   - $v = (\hat{m})^k$ (消息的加密)

3. **Alice (去盲/解锁):**
   Alice 收到 $u, v$。她知道自己的随机数 $\rho$。
   她想要消掉 $g^{\rho k}$ 这一项。
   
   推导如下：
   $$ v = (\hat{m})^k = (H(m) \cdot g^\rho)^k = H(m)^k \cdot g^{\rho k} $$
   
   Alice 计算 $u^\rho$：
   $$ u^\rho = (g^k)^\rho = g^{k \rho} $$
   
   **最终计算：**
   $$ \text{Result} = \frac{v}{u^\rho} = \frac{H(m)^k \cdot g^{\rho k}}{g^{k \rho}} = H(m)^k $$

### 为什么安全？

- **对 Bob 安全:** Bob 全程只看到 $\hat{m}$，这看起来就是个随机数，因为 $\rho$ 是 Alice 选的随机数，完美隐藏了 $m$。
- **对 Alice 安全:** Alice 拿到了 $H(m)^k$，也就是她想要的结果。但她只知道 $g^k$ (即 $u$)，根据离散对数难题 (DLP)，她无法反推出 Bob 的私钥 $k$。

---

## 3. OT 的扩展应用 (MPC)

虽然 Assignment 1 考的是计算 $H(m)^k$，但在 Tutorial 中，我们看到了 OT 更广泛的用法：**1-out-of-2 OT**。

在 Tutorial 的 **Garbled Circuit (混淆电路)** 部分，OT 是这样用的：

1. Alice 针对电路的每一根线准备两个密钥 $K_0$ (代表0) 和 $K_1$ (代表1)。
2. Bob 如果手里的输入是 bit $b$ (比如 1)，他就需要获得 $K_1$，但不能获得 $K_0$。
3. 双方运行 OT 协议，Bob 成功拿到了对应的密钥，且 Alice 不知道 Bob 拿走了哪个。

## 4. 总结

- **核心思想:** 利用**随机数**把消息“糊住”（Blinding），对方处理完后再把随机数“洗掉”（Unblinding）。
- **数学工具:** 指数运算的交换律 $(g^a)^b = (g^b)^a$。
- **考点预警:** 考试如果让你设计一个让双方都不泄露隐私的计算协议，**OT** 就是标准答案。

明天复习最后一个大难点：**Schnorr 签名与零知识证明 (ZKP)**，那是 Assignment 2 的重头戏。
