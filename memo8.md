# Somewhat 同态加密

## 同态加密

- $Evaluate(f,Enc(x))=Enc(f(x))$
  - 通过函数 $Evaluate()$，加密前后对于 $f$ 是『同态』的
- 全同态加密就是对任何的 $f$ 都能做到同态加密
  - 在任意环上 $(ADD,MULT)$ 是图灵完备的，所以我们只需要实现加法、乘法的同态，即：
    - $Dec(ADD(c_1,c_2))=DEC(c_1)+DEC(c_2)$
    - $Dec(MULT(c_1,c_2))=DEC(c_1)DEC(c_2)$

---
## Polly Cracker 方案及其延伸

- Polly Cracker 方案
  - 将 $({0,1},XOR,AND)$ 环转换成多项式
  - 形式：
    - $Keygen()=(pk,sk)=(\{f\},[s_1,s_2,...,s_n])$
      - $f_i(sk)=0$
    - $Enc()$：
      - 从 $pk$ 中随机一个子集和 $g,g(sk)=0$
      - $cip=Enc_{sk}(m)=m+g$
    - $Dec()=cip(sk)$
  - 破解：获得大量 $0$ 的加密结果，高斯消元
- Noisy Polly Cracker 方案
  - 加入扰动：$f_i(sk)=2e$，即小偶数
  - 扰动会在运算过程中叠加
- SWHE
  - 方案：
    - $sk=p,2|p$
    - $cip=msg+pq+2e,e \ll p,q \gg p$
    - $msg=(cip \mod p) \mod 2$
  - 误差积累：
    - $c_1+c_2=m_1+m_2+p(q_1+q_2)+2(e_1+e_2) \mod p=m_1+m_2+2(e_1+e_2)$
    - $c_1c_2=m_1m_2+p(q_1m_2+q_2m_1+pq_1q_2)+2e_1(m_2+pq_2)+2e_2(m_1+pq_1)+4e_1e_2 \mod p=m_1m_2+(2e_1+m_1)(2e_2+m_2)$
  - 显然，当 $|f(m_1+2e_1,m_2+2e_2,...,m_t+2e_t)|<p/4$ 时，$f$ 可以进行同态加密

---
## 基于 LWE 的 SWHE

- LWE 的 Noisy Polly Cracker 表述
  - 首先生成 $s \sim U(\Z_q^n)$ 以及若干 $e_i \sim \chi$ 和 $f(x)=b+\sum f_ix_i \mod q$，使得 $f_i(s)=e_i$
    - $b=-\sum f_is_i+e$
- Regev 方案：
  - $sk \sim U(\Z_q^n),pk=f(x),f(s)=2e \ll q=2k+1$
  - $cip=msg+g$
  - $msg=(cip(sk) \mod q)\mod 2$
  - 在做乘法时需要重整为 $O(n)$ 项的多项式
    - 令 $x'_{ij}=x_ix_j,s'_{ij}=s_is_j,g'(x)=\sum {g_1}_i{g_2}_jx'_{ij}$
    - Key Switching
      - 将 $s'$ 和 $g'$ 二进制展开
      - 用一个公开的 $t$ 作为 $sk$ 『加密』$s'$，将结果 $h(x)$ 加在公钥上
      - 令 $d(x)=\sum g'_ih_i=g'(s')+\sum g'_ie'_i(g'_i \in {0,1})$
      - 密文长度仍然会以 $O(\log n)$ 的比例上升
  - 该方案下误差仍然会上升
- 基于理想格的 SWHE
  - 此时 $f$ 为一次函数
  - 重整时需要在环上进行二进制展开
  - 因为在模多项式环上，所以密文长度不会增加

---
## 基于 NTRU 的 SWHE

- 方案
  - $sk$ 是小奇数，$q$ 是奇数，在理想格下工作
  - $pk=\{f(x)=kx\},f_i(sk)=2e_i$
  - $cip=mx+g$
    - m 可以视作一个理想格下模 $2$ 的多项式
  - $msg=cip(s)=ms+g(s) \mod 2$
  > **NTRU 问题**：
  > 判定多项式 $f$ 是否是两个短多项式的商
- 密钥置换
  - 有时需要把来自多个私钥的数据互相运算，所以仍然需要密钥置换
  - 将 $a_{s,t}=se_{s,t}t^{-1}$ 加入公钥集
    - $e_{s,t}=2e^*+1$，其中 $e^* \sim \chi$
  - $cip'=ca_{s,t}$
    - $cip't=cip(a_{s,t}t)=cip(se_{s,t})=m+\epsilon$
- 误差是倍速增长的，但运算速度很快