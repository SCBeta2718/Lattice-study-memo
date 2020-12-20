# 基于格的陷门函数及其应用

## 基于格的原象抽样陷门函数构造

> **陷门函数**：
> - 存在一个函数 $f:D \rightarrow D'$ 与 $f^{-1}$，$f$ 是公开的，而 $f^{-1}$ 则是私密的，即陷门。
> - 显然 $f$ 是伪随机的
> - 对于只知道 $f$ 的攻击者来说，求 $f$ 的逆是困难的，但 $f^{-1}$ 提供了更多的信息可以高效求 $f$ 的逆
>
> **陷门置换**：
> - 要求 $f$ 是一个置换的陷门函数
>
> **从陷门置换构造数字签名**：
> - $Keygen()=(pk=f,sk=f^{-1})$
> - $Sign(msg)=f^{-1}(H(msg))$
> - $Verigy(msg,sig)=[f(sig)=H(msg)]$
> 
> **原象抽样陷门函数**
> - 要求 $D' \in D$，且定义域满足一定概率分布
> - 此时 $f$ 有多个逆，$f^{-1}$ 获得其中某个逆的概率满足定义域的概率分布
> - 原象抽样陷门函数可以用和陷门置换一样的方法构造数字签名
- 改造 GGH 方案
  - 用 SIS 问题构造格：$L=\{x|Ax=0 \mod q\}$ 与它的一个较好的基 $B$
  - $pk=f(x)=Ax \mod q,(x \sim N_{5\lambda_n}^n)$
    - 显然 $f(x)$ 是均匀的
    - 由于 $x$ 按正态分布取得，所以绝大多数 $x$ 会落在较小区域内
      - 对于攻击者，$f^{-1}(x)$ 就是 SIS 问题 $A(x-A^{-1}u)=0$
  - 假设给定 $u=Ax$，那么 $x$ 应当满足离散正态分布
  - $f^{-1}$ 的构造
    - 方案一：概率最近平面算法
      - 对于点 $u$ 和『好』格基 $B$，以 $u$ 为原点构造格 $L$ 的陪集 $L_u$
      - 对于『好』格基 $B$ 的每一个向量 $b_i$，将原点倒序正交投影到较近的与 $b_i$ 平行的平面上
        - 具体而言，按到原点距离的正态分布决定正交投影的概率
      - 时间复杂度 $O(n^3)$
    - 方案二：拼合两个正态分布，使得结果球对称
        > **协方差矩阵**：
        > - $n$ 维随机变量 $[x_1,x_2,...,x_n]$ 的协方差矩阵为：$[Cov(x_i,x_j)=E(x_ix_j)-E(x_i)E(x_j)]$
        > - 协方差矩阵是半正定的
        >   - $y^T[E(x_i-E(x_i))E(x_j-E(x_j))]y \\
            =[E(x_i-E(x_i))\sum y_jE(x_j-E(x_j))]y \\
            =\sum y_iE(x_i-E(x_i))\sum y_jE(x_j-E(x_j)) \\
            =(\sum y_iE(x_i-E(x_i)))^2 \geq 0$
        > - 可理解为一种广义的『方差』
        >
        > **高维正态分布与协方差矩阵**：
        > - $f_{\mu,\Sigma}(x)={1 \over \sqrt{(2\pi)^n|\Sigma|}}\exp(-{1\over 2}(x-\mu)^T\Sigma(x-\mu))$
        >   
        > - 对于球对称的正态分布，协方差矩阵是单位矩阵的方差倍
        >   - 当图像关于原点球对称时，分布函数退化为 $f_s(x)=s^{-n}\exp(-{\pi \over s^2} ||X||_2^2)$，其中 $s=\sigma\sqrt{2\pi}$
        >   - 可得协方差矩阵的几何意义为：在一组基 $S$ 上取高维正态分布，$\Sigma=SS^T$
        > - 正态分布协方差矩阵正定
        > - 高维正态分布协方差矩阵可加
      - 对于点 $u$ 和『好』格基 $B$，以 $u$ 为原点构造格 $L$ 的陪集 $L_u$
      - 如果 $N(0,\Sigma_1)$ 为在格基 $B$ 上取得的正态分布
        - $y^T\Sigma_2y=y^T(\sigma^2-\Sigma_1)y=\sigma^2-y^T\Sigma_1y \geq 0$（正定性）
        - 则 $\sigma^2>||\lambda(\Sigma_1)||_{\infty}=||\lambda(BB^T)||_{\infty}$
        - 由此我们获得一个矫正分布 $N'(0,\sigma^2E-\Sigma_1)$，在此分布下生成一个点 $P$ 并在陪集 $L_u$ 中舍入
          - 只需要乘 $A^{-1}$ （可预处理）并取近似即可
        - 时间复杂度 $O(n^2)$

---
## 通过 LWE 问题构建基于身份的加密方案（IBE）

> **基于身份的加密方案**：
> - $Setup()=(mpk,msk)$：可信任第三方生成主公钥 $mpk$ 和主私钥 $msk$
> - $Keygen(id)=sk$：用户让可信任第三方通过身份 $id$ 生成私钥 $sk$
> - $Enc(mpk,id,msg)=cip$：只需知道主公钥 $mpk$ 和对方的身份 $id$ 即可加密
> - $Dec(sk,cip)=msg$：通过自己的私钥解密

- 从之前的对偶 LWE 公钥加密出发
  - $mpk=A \sim U(\Z_q^{n \times m})$
  - $sk=Keygen(id)=f^{-1}_A(id)$
    - 按之前的讨论，此处会使用 $msk$
  - $cip=Enc(mpk,id,msg=C \in \{0,1\})=(\vec b,b')=(A^Ts+e,id^Ts+e'+Cq/2)$
    - $s \in U(\Z_q^n)$
  - $msg=Dec(sk,cip)=b'-sk^T\vec b \simeq Cq/2$
- 参数：
  - $A \sim U(\Z_q^{n \times m}),q=poly(n),m \geq O(n\log n),q=2^k$
  - 构造一个『规整的』矩阵 $G$
    - $g=[1,2,4,...,2^{k-1}] \in \Z_q^k$
      - 对 $g$ 做 LWE：
        - $b=gs+e=[s+e_0,2s+e_1,...,2^{k-1}s+e_{k-1}] \mod q$
        - $e$ 很小，并且高位会被舍去，所以 $b$ 的差分可以确定 $s$ 的某一位
    - 令 $G=diag(\underbrace{g,g,...,g}_k) \in \Z_q^{n \times nk}$
    - $A=[\bar A|G]\left[\begin{matrix}E & -R \\0 & E\end{matrix}\right]=[\bar A|G-\bar AR]$
      - $\bar A \sim U(\Z_q^{n \times \bar m}),R \sim N(\Z_q^{\bar m \times nk})$ 
      - 定义 $R$ 就是 $A$ 的陷门
    - 性质：
      - 对 $A$ 求 LWE（$b=A^Ts+e$）
        - $[R|E]b=[R|E]A^Ts+[R|E]e=G^Ts+[R|E]e$
          - 要求 $[R|E]e \in [-q/4,q/4)$，才能对结果进行纠错，获得 $G^Ts$
  - $f$ 和 $f^{-1}$ 的构造
    - $f_G(x)=Gx \mod q$
    - $f^{-1}_G(x)$
      - 对 $G$ 分块解 LWE
    - $x=Gf^{-1}_G(x)=Af_A^{-1}(x)=[\bar A|G-\bar AR]f_A^{-1}(x)$
      - $A[R|E]^T=G$
      - $f_A^{-1}(x)=[R|E]^Tf^{-1}_G(x)+P$
        - $P$ 是一个类似前述正态分布的修正扰动
- 结果：
  - $Setup()=(mpk,msk)=((A,u),R)=(([\bar A|-\bar AR],u),R)$
  - $Keygen(id=H)=(pk,sk)=(A_H,f^{-1}_{A_H}(u)),A_H=A+[0|HG]=[\bar A|HG- \bar AR]$
    - $f^{-1}_{A_H}(u)$ 用前述的算法计算
  - $Enc(),Dec()$：对偶公钥加密