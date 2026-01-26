---
title: '密码学复习笔记(4)：Schnorr 协议与零知识证明 (ZKP)'
description: 'Assignment 2 核心攻略。如何证明“我知道秘密”却不泄露秘密？深入解析 Sigma 协议与 Nonce 重用攻击。'
pubDate: 'Dec 15 2025'
tags: ["Archive"]
---

这是密码学复习的重头戏，也是 **Assignment 2** 和 **Tutorial** 中最硬核的部分。

今天我们复习 **Schnorr 身份认证协议**，它是 **零知识证明 (Zero-Knowledge Proof, ZKP)** 的经典案例，也被称为 **Sigma 协议**（因为协议流程图形状像希腊字母 $\Sigma$）。

## 1. 场景设定

- **Prover (P, 证明者):** 拥有私钥 $x$。
- **Verifier (V, 验证者):** 知道公钥 $y = g^x \bmod q$。
- **目标:** P 要向 V 证明“我知道 $x$”，但**绝不能把 $x$ 告诉 V**。

## 2. 协议标准流程 (必背!)

这是一个三步交互过程 (Commit - Challenge - Response)。

### 第一步：Commitment (承诺)

Prover 选一个随机数 $k$ (Nonce)，计算承诺值 $t$ 发给 Verifier。
$$ t = g^k \bmod q $$
*(就像把 $k$ 锁在箱子里扔过去)*

### 第二步：Challenge (挑战)

Verifier 随机选一个挑战数 $c$，发给 Prover。
*(V: "既然你知道秘密，那回答我这个问题...")*

### 第三步：Response (响应)

Prover 计算响应值 $z$，发回给 Verifier。
$$ z = k + c \cdot x \bmod q $$
*(注意：这里 $k$ 保护了 $x$，因为 $k$ 是随机的，所以 $z$ 看起来也是随机的)*

### 第四步：Verify (验证)

Verifier 收到 $z$ 后，验证下面的等式是否成立：
$$ g^z \stackrel{?}{\equiv} t \cdot y^c \bmod q $$

**为什么能验证通过？(Completeness)**
$$ \text{右边} = t \cdot y^c = g^k \cdot (g^x)^c = g^k \cdot g^{xc} = g^{k+cx} = g^z = \text{左边} $$

---

## 3. 核心考点：针对 Schnorr 的攻击

Tutorial 里花了大量篇幅讲 **"Insecure"** 的情况。这是考试计算题的来源。

### 攻击场景：Nonce Reuse (随机数重用)

如果 Prover 在两次不同的证明中，**使用了同一个随机数 $k$**（导致承诺 $t$ 也一样），会发生什么？

**已知数据 (Eve 在旁路观察):**

1. 第一次交互: $(t, c_1, z_1)$
2. 第二次交互: $(t, c_2, z_2)$
3. 公开参数: $g, y$

**攻击推导 (解线性方程组):**
根据协议定义，我们有两个方程：

1. $z_1 = k + c_1 \cdot x$
2. $z_2 = k + c_2 \cdot x$

因为 $k$ 是相同的，我们可以把两式相减消去 $k$：
$$ z_1 - z_2 = (c_1 - c_2) \cdot x $$

**直接解出私钥 $x$:**
$$ x = \frac{z_1 - z_2}{c_1 - c_2} \bmod q $$

**结论：** 只要 $k$ 重用，私钥立马暴露。这解释了为什么 Tutorial 里反复强调随机数的重要性。这也叫 **Soundness (可靠性)** 的反面案例。

---

## 4. 零知识 (Zero-Knowledge) 的直观理解

既然 $z = k + c \cdot x$，Verifier 拿到了 $z$，会不会反推 $x$？
不会。因为方程里还有一个未知数 $k$。

**模拟器 (Simulator) 思想：**
为了证明这个协议是“零知识”的（即没泄露任何信息），我们需要构造一个**模拟器**。
如果我没有私钥 $x$，但我可以**控制时间倒流**（或者先选 $c$ 和 $z$），我也能伪造出一份完美的交互记录 $(t, c, z)$：

1. 先瞎编一个 $c$ 和 $z$。
2. 倒推计算 $t = g^z \cdot y^{-c}$。
3. 输出 $(t, c, z)$。

这份伪造的记录和真实的记录在数学分布上是一模一样的。既然没有私钥也能造出这种记录，说明**这种记录本身不包含私钥的任何信息**。

---

## 5. 总结

1. **Schnorr 流程:** $t=g^k \to c \to z=k+cx$。验证 $g^z = t \cdot y^c$。
2. **致命弱点:** 随机数 $k$ 绝对不能重用，否则通过 $(z_1-z_2)/(c_1-c_2)$ 就能算出私钥。
3. **Assignment 2:** 可能会让你设计类似的 Sigma 协议，或者证明某个变体是不安全的（套用上面的攻击方法即可）。

下一篇，我们将进入最后的难关：**多方安全计算 (MPC) 与混淆电路**。
