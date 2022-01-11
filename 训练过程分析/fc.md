全连接FC
===============

### 参数

I:input:(C,)
O:output:(N,)
W:weight:(N,C)
B:bias:(N)

### 计算公式

$$
\begin{aligned}
  O(n)
  &=\sum_{c=0}^{C-1}{I(c)*F(n,c)} + b(n)
\end{aligned}
$$