## 公式

**大括号公式**
$$
\begin{cases}
min(dp[i-1] [j-1]，dp[i] [j-1]，dp[i-1] [j]) + 1 \ \ \ \ \ \ \ \ \ ij位置字符不相等\\
min(dp[i-1] [j-1]，dp[i] [j-1] + 1，dp[i-1] [j]+ 1) \ \ ij位置字符相等
\end{cases}
$$
**联排公式**
$$
\begin{aligned}
\hat{b} &=\frac{1}{y_{j}}-X_{j}^{T} \hat{W} \\
&=y_{j}-X_{j}^{T} \hat{W} \\
&=y_{j}-\sum_{i=1}^{n} \hat{\alpha}_{i} y_{i} X_{j}^{T} X_{i}
\end{aligned}
$$
**符号**
$$
y_{i}^{(b)}=B N\left(x_{i}\right)^{(b)}=\gamma \cdot\left(\frac{x_{i}^{(b)}-\mu\left(x_{i}\right)}{\sqrt{\sigma\left(x_{i}\right)^{2}+\epsilon}}\right)+\beta
$$
**居左**
$$
\begin{aligned}
&\min _{W, b} J(W)=\min _{W, b} \frac{1}{2}\|W\|^{2} \\
&\text { s.t. } \quad y_{i}\left(X_{i}^{T} W+b\right) \geq 1, i=1,2, \ldots n .
\end{aligned}
$$

**因为所以**：$\because and \therefore$

**箭头**：[如何用LaTeX打出各种箭头？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/263896738)







