# 基本的密码分析

## LLL 算法

- LLL 基的性质
  - 随着维度上升，施密特基的长度下降不会太快
  - $B=\widetilde BU$
    - 除了主对角线外，$U$ 中的每个元素绝对值都小于 $1/2$
    - $0.75|\widetilde B_i|^2 \leq |U_{i+1,i}\widetilde B_{i}+\widetilde B_{i+1}|^2$
      - $|\widetilde B_i| \leq {\sqrt 2}|\widetilde B_{i+1}|$
    - 一个 LLL 基的第一个向量 $B_1$ 最大为 $2^{(n-1)/2}\lambda_1$
      - $|B_1| = |\widetilde B_1| \leq 2^{1 \over 2}|\widetilde B_2| \leq 2^{2 \over 2}|\widetilde B_2| \leq 2^{3 \over 2}|\widetilde B_2| \leq ... \leq 2^{n-1 \over 2}|\widetilde B_n|$
      - $|B_1| \leq 2^{n-1 \over 2} \lambda_1$
- LLL 基的构造
  - 考虑 $U$ 是上三角阵，对 $U$ 做列变换相当于对特定位置每次加减一即完成构造
  - 对相邻两个基向量检查限制条件，不满足则交换

---
## 子集和问题

- **随机**取 $a_i \in \Z_M$，再取 $T=\sum_{sub(A)} a_i$，找到一个 $sub(A)$ 满足该条件
  - 传统算法（不考虑 $\Z_M$ 的循环性）：
    - 状压 dp：$O(2^n)$
    - 01 背包：$O(nM)$
  - $O(n^{\log n}) \leq M \leq O(2^n)$ 时，存在 $O(poly(n))$ 的算法（生日攻击）
  - $O(2^n) \leq M \leq O(2^{n \log n})$ 时，只有指数级算法
  - 当 $M \geq O(2^{n^2})$ 时存在多项式算法
- 子集和问题的 LLL 攻击
  - 令 $T=\sum a_ix_i \mod M， (x_i \in \{0,1\})$，$A'=[a_1,a_2,...,a_n,-T]$
  - 构造 $L_A=\{y \in \Z^{n+1}: \langle a,y\rangle=0 \mod M\}$
  - 显然 $x=[x_1,x_2,...,x_n,1] \in L_A$，且 $|x| \leq \sqrt{n+1}$
  - $|LLL(L_A)| \leq 2^{n \over 2}\lambda_1(L) \leq 2^{n \over 2}\sqrt{n+1}$
  - 如果 $|x'| \leq 2^{n \over 2}\sqrt{n+1}$ 唯一，则 $x=x'=LLL(L_A)$
    - 假设存在一个『坏』向量 $y=[y_1,y_2,...,y_n,k] \in L$，且 $|y| \leq 2^{n \over 2}\sqrt{n+1}$
    - $\sum a_i(y_i-kx_i)=0 \mod M$，且 $\exist y_i-kx_i \not ={0}$
    - 排除所有的 $y$：
      - 构造球 $S=\{|y| \leq 2^{n \over 2}\sqrt{n+1}, y \in \Z^{n+1}\}$
      - 对于任意的 $X \in \{0,1\}^n$ 与『坏』的 $Y=[y_1,y_2,...,y_n,k]$，考虑概率 $P(\sum a_i(y_i-kx_i)=0 \mod M)$ 
        - 注意 $a_i$ 随机
        - 如果 $M \in prime$，$P=1/M$
      - 通过布尔不等式合并概率：对于所有的 $(X,Y)$，$P=2^n|S|/M \simeq 2^{n+1 \over 2}\pi^{n \over 2}e^{n+3 \over 2}(n+3)^{-n-2}r^{n+1}/M=2^{n+1 \over 2}\pi^{n \over 2}e^{n+3 \over 2}(n+3)^{-n-2}(2^{n \over 2}\sqrt{n+1})^{n+1}/M=2^{(n+1)^2 \over 2}\pi^{n \over 2}e^{n+3 \over 2}{(n+1)^{n+1 \over 2} \over (n+3)^{n+2}}/M$
        > **布尔不等式**：并的概率不大于概率的和
        > 
        > **$\Gamma$ 函数**：
        > - $\Gamma (x)=\int_0^{+\infty}t^{x-1}e^{-t} dt \simeq \sqrt{2\pi/x}({x \over e})^x$
        > 
        > **n 维超球体积**：
        > - $V_n(r)={\pi^{n/2} \over \Gamma(n/2+1)}r^n \simeq {\pi^{n/2} \over \sqrt{2\pi/(n/2+1)}({n/2+1 \over e})^{n/2+1}}r^n=2^{n \over 2}\pi^{n-1 \over 2}e^{n+2 \over 2}(n+2)^{-n-1}r^n$
      - $M \gg 2^{(n+1)^2 \over 2}\pi^{n \over 2}e^{n+3 \over 2}{(n+1)^{n+1 \over 2} \over (n+3)^{n+2}}$
        - $M>9^{n+1}2^{(n+1)^2}$

---
## SIS 与 LWE

- 构造格 $L_A=\{y \in \Z^m: Ay\rangle=0 \mod q\}$
  - $\lambda_1(L) \leq \sqrt m |L|^{1 \over m}$
    > $|L|=\#(\Z^m/L)$
    > - 格的行列式等于格上任意基本平行多面体整点个数（行列式是面积！）
    - $|L| \leq q^n$
      - 如果 $A$ 各行线性无关，那么 $|L|=q^n$
    - $\lambda_1(L) \leq \sqrt m q^{n \over m}$
  - 同上，对于所有 $|Y|<r$，$P(Y \in L) \leq |S_r|/|L|=|S_r|/q^n=2^{m \over 2}\pi^{m-1 \over 2}e^{m+2 \over 2}(m+2)^{-m-1}r^mq^{-n}$
    - $\lambda_1>r \simeq q^{n/m}\sqrt{m/(2\pi e)}$
    - 此时用 LLL 可找到一个长度接近下界的向量
- 可归约到 LWE
- 在 $\infty-$范数下暂时没有好的攻击

---
## 实际应用

- 安全参数 $\alpha$
  - 当考虑短向量问题时（LLL），$\alpha=|s|/|L|^{1/m}$
  - 当考虑独立短向量问题时（LWE），$\alpha=\lambda_1/|s|$
- 性能
  $\sqrt[m] \alpha$ | 破解难度
  :-: | :-:
  1.02 | LLL
  1.01 | BKZ（优化 LLL）
  1.007 | 目前安全
  1.005 | 长期安全
  
   