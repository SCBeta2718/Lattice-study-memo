# Learning With Errors 困难问题

## 历史

- Oded(2005)，提出 LWE 的搜索版本，并证明它最坏情况下不比 SIS 和 GapSVP 简单
  - 量子算法归约到 GapSVP，要求 q 很大，与 n 是指数关系
- 2009，证明在特定情况下有传统的归约算法
- LWE 搜索问题不比 LWE 判定问题难

---
## 概述

- 搜索版本：已知向量 $b \in \Z_q^n$，从 $\Z_q^n$ 中随机选取若干向量 $a$，求满足所有 $a^Ts+e=b$ 的 $s$
  - 如果扰动 $e=0$，高斯消元即可
  - 如果 $e$ 满足正态分布
    - 令标准差 $\sigma=\alpha q \geq \sqrt n$，即噪声的绝对值期望为 $\sqrt n$
    > 存在 $O(\exp(\sigma^2))$ 的攻击算法
- 判定版本：
  - 给定一对 $(A=[a_1,a_2,...,a_m],b)$，要求区分它是按这一规则产生的还是随机生成的
  - 密码学中主要使用这个版本
- 与 SIS 的对比
    SIS | LWE
    :-: | :-:
    $Az=0, ans=z_{short}$|$(A,b=A^Ts+e)$ vs. $(A,b)_{rand}$
    『计算性』的，一个搜索问题|一个判定性问题
    存在多解|只有一个 $s$ 满足条件
    从 LWE 归约|存在量子算法可从 SIS 归约
    单向函数 <br> 抗碰撞哈希函数 <br> 数字签名 <br> 身份加密|公钥加密 <br> 茫然传输
    > **LWE $\rightarrow$ SIS**：考虑解 SIS 问题 $Az=0$，那么 $b^Tz=s^TAz+e^Tz=e^Tz=\epsilon$。如果 $b^Tz$ 确实很小即判定

---
## 判定性 LWE 问题
- 已知 $s' \in \Z_q^n$，如何判断它是一个 LWE 问题 $(A,b)$ 的解？
  - $e'=b-As'$ 应当是『小』的
  - 否则 $e'$ 应当分布在整个 $\Z_q$ 中
- 随机自归约性
  - 如果某一算法有一定概率解决 LWE 搜索问题，那我们就能不断生成等价的 LWE 问题将概率放大
  - 把解 $s$『平移』至 $s+t$
    - $(A,b=A^Ts+e) \rightarrow (A,b'=A^T(s+t)+e=b+A^Tt)$
- LWE 问题的合并
  - 如果存在若干个共享 $A$ 的判定性 LWE 问题 $(A,b_1=A^Ts_1+e_1,b_2=A^Ts_2+e_2,...,b_t=A^Ts_t+e_t)$，可以将它组合成一个 LWE 问题：
    - $(A,B=A^T[s_1,s_2,s_3,...,s_t]+[e_1,e_2,...,e_t])$
    - 证明：$B_{ij}=A_i^Ts_j+{e_i}_j$
- 搜索/判定等价性
  - 搜索 $\rightarrow$ 判定（假设 $q=poly(n),q \in prime$）
    - 遍历 $s_i=0,1,...,q-1$，将问题平移到 $s_i=0$
    - 取随机的向量 $r \in \Z_q^n$，$A_i \leftarrow A_i-r$，解判定性 LWE 问题 $(A',b)$
      - 注意 $b=A^Ts+e=A'^Ts+r^Ts_i+e$，$r^Ts_i$ 随机，所以仅当 $s_i=0$ ，即原 $s_i$ 为解时判定成立
    - $O(nq)$
- 如果已知 $s$ 满足正态分布，那么 LWE 仍然是困难的
  - 寻找 $e$ 和寻找 $s$ 等价
    - 取一些 $A$ 中的向量和其对应的值，构成 $(\bar A,\bar b)$
    - 对于 $A$ 中每个向量 $a_i$，及其对应值 $b_i$，令 $a_i'=-\bar A^{-1}a_i$，$b_i'=b_i+\bar b^Ta_i'$
      - $b_i'=b_i+\bar b^Ta_i'=a_i^Ts+e_i+s^T\bar Aa_i'+\bar e^Ta_i'=a_i^Ts+e_i-s^T\bar A\bar A^{-1}a_i-e^Ta_i'=e_i-\bar e^Ta_i$

---
## LWE 公钥加密方案

- 原始加密方案
  - Alice 随机选择（或者从第三方获得）矩阵 $A \in \Z_q^{n \times m}$，选择私钥 $s \in \Z_q^n$，构造公钥 $b=A^Ts+e$
  - Bob 希望向 Alice 发送信息 $C \in \{0,1\}$，他将随机生成 $x \in \{0,1\}^m$，计算 $\vec u=Ax,u'=b^Tx+Cq/2$，发送 $(\vec u,u')$
  - Alice 解密：$u'-s^T\vec u=b^Tx+Cq/2-s^TAx=s^TAx+e^Tx-s^TAx+Cq/2=Cq/2+e^Tx \simeq Cq/2$
    - 通过接近 $0$ 或 $q/2$ 区分
- 对偶加密方案
  - Alice 随机选择（或者从第三方获得）矩阵 $A \in \Z_q^{n \times m}$，随机生成 $x \in \{0,1\}^m$，构造公钥 $u=Ax$
  - Bob 希望向 Alice 发送信息 $C \in \{0,1\}$，他将选择随机的 $s \in \Z_q^n$，计算 $\vec b=A^Ts+e,b'=u^Ts+e'+Cq/2$，发送 $(\vec b,b')$
  - Alice 解密：$b'-x^T\vec b \simeq Cq/2$、
> **参数**：注意 $x$ 的定义域大小为 $2^m$，密文值域为 $q^n$，所以要满足 $m \geq n \log_2 q$
- 原始方案时空复杂度：
  - 空间：$A \sim O(n^2\log q),s \sim O(n),b \sim O(n\log n)$
  - 时间：$T_{Enc}(n)=O(n^2\log q),T_{Dec}(n)=O(n^2\log q)$
- 优化
  - Alice 随机选择（或者从第三方获得）矩阵 $A \in \Z_q^{n \times n}$，**以与 $e$ 同分布**选择私钥 $s \in \Z_q^n$，构造公钥 $u=A^Ts+e_A$
  - Bob  希望向 Alice 发送信息 $C \in \{0,1\}$，他同样**以与 $e$ 同分布**选择 $r \in \Z_q^n$，再随机生成 $x \in \{0,1\}^n$，计算 $\vec b=Ar+e_B,b'=u^Tr+e+Cq/2$，发送 $(\vec b,b')$
  - Alice 解密：$b'-s^Tb=e+Cq/2+e_A^Tr-s^Te_B \simeq Cq/2$
  - 通过同分布规避了 $x$ 的大小问题，优化到 $A(n)=T(n)=O(n^2)$

---
## LWE 伪随机函数

> **伪随机函数（PRF）**：
> - 一个函数族 $F=\{F_s: \{0,1\}^k \rightarrow D\}$，$s$ 是随机数的种子
> - 对于任意的 $F_{s_0} \in F$，对于任意的一系列询问 $\{X\}$，不能区分 $\{F_{s_0}(X)\}$ 和随机映射 $U(X)$

> **合成器**：
>  - 一个函数 $S:D \times  D \rightarrow D$
>  - 对于任意的 $m=poly(n)$，随机选择 $2m$ 个值 $a_1,a_2,...,a_m \in D$ 与 $b_1,b_2,...,b_m \in D$，那么 $S(a_i,b_j)$ 组成的矩阵应当是随机的矩阵
>    - 注意这里所有的 $a_i$ 和 $b_i$ 都是**随机**选择的
- 基于合成器的 PRF 构造
  - $k=1$：$F_{(s_0,s_1)}(x)=s_x$
  - 假设在 $k=k_0$ 下存在 PRF $F_{k_0}$，则可以构造如下 PRF 使得它能够处理 $k=2k_0$ 的情况：
    - 从 $F_k$ 中选择两个函数 ${F_{k_0}}_l,{F_{k_0}}_r$，构造 $F_{2{k_0}}=S({F_{k_0}}_l(x_{hbit}),{F_{k_0}}_r(x_{lbit}))$
  - 合成器深度是 $O(\log k)$ 的
- 基于 LWE 的合成器构造
  - LWR 问题
    - 不考虑 $e_{ij}$，而是将 $a_i^Ts_j$ 舍入到 $\Z_q$ 的循环子群 $\Z_p$
      - 类似于解密时舍入到 $0$ 或 $q/2$
    - 本质是 LWE，因为舍入的误差是在小范围随机的
  - 令 $m=n$，即 $a \in \Z_q^n,s \in \Z_q^n,e \in \Z_q$
  - 合并 $a_1,a_2,...,a_n$ 与 $s_1,s_2,...,s_n$ 构成的 $n^2$ 个 LWR 问题
    - $Syn(A,S)=[A^TS]_{\Z_p}$
  - 注意该合成器会丢失信息（$Z_q \rightarrow \Z_p$），所以 PRF 构造不能太多，并且每层递归需要选取递增的模数 $p|q$
- 基于 DDH 问题的 PRF 构造
  > **DDH 问题**
  > - 给定 $g,g^a,g^b \in G$（$G$ 是交换群），要求判定某个 $T \in G$ 是随机元素还是 $g^{ab}$
  - $F_{g,s}(x)=g^{\prod s_i^{x_i}}$
  - 改造成 LWE 问题
    - $F_{A,S=[S_1,S_2,...,S_k]}(x)=[A^T \prod S_i^{x_i} \mod q]_{Z_p}$
      - 参数：$A \sim U(\Z_q^{n \times n}),S \sim N(\Z_q^{n \times n}),q>p,q/p=poly(n)$