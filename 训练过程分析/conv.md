卷积Conv
===================

### 参数

P:padding:(pt,pb,pl,pr) \
S:stride \
K:kernel
W:weight:( N, C, K , K)
B:bias:(N)

I:input:(C,HI,WI) \
O:output:(N,HO,WO) \
$\frac{dL}{dO}$:(N,HO,WO) \
$\frac{dL}{dI}$:(C,HI,WI) \
$\frac{dL}{dW}$:(N,C,K,K) \
$\frac{dL}{dB}$:(N,) 


##### 缩写
O:Output I:Input IP:Input with padding

### 计算公式

#### 前向传播

$$
\begin{aligned}
O(n,x,y)&=
  \sum_{c=0}^{C-1}{
      \sum_{j=0}^{K-1}{
        \sum_{j=0}^{K-1}{
          F(n,c,i,j)*IP(c,yS+j,xs+i) +B(n)
        }
    }
  } \\
  &=\sum_{c=0}^{C-1}{
    \sum_{j=0}^{K-1}{
      \sum_{j=0}^{K-1}{
        W(n,c,i,j)*I(c,yS+j-pt,xs+i-pl)
      }
    }
  }
\end{aligned}
$$
其中，第二行由于去掉了padding，可能发生溢出

#### 反向传播

* stride = 1

$$
\begin{aligned}
  \frac{dL}{dI(c,y,x)}
    &=\frac{dL}{dO}*\frac{dO}{dI(c,y,x)} \\
    &=\sum_{n=0}^{N-1}{
      \sum_{j=0}^{H-1}{
        \sum_{i=0}^{W-1}{
          \frac{dL}{dO(n,j,i)}*\frac{dO(n,j,i)}{dI(c,y,x)}
        }
      }
    } , 其中大部分I(c,y,x)对O没有贡献 \\
    &=\sum_{n=0}^{N-1}{
      \sum_{j=0}^{H-1}{
        \sum_{i=0}^{W-1}{
          \frac{dL}{dO(n,j,i)}*W(n,c,y-j+pt,x-i+pl)
        }
      }
    } \\
    &  令j'=y-j+pt,i'=y-i+pl \\
    &=\sum_{n=0}^{N-1}{
      \sum_{j'=0}^{K-1}{
        \sum_{i'=0}^{K-1}{
          \frac{dL}{dO(n,y-j'+pt,x-i'+pt)}*W(n,c,j',i')
        }
      } 
    } \\
    & 令j=K-1-j',i=K-1-i' \\
    &=\sum_{n=0}^{N-1}{
      \sum_{j=0}^{K-1}{
        \sum_{i=0}^{K-1}{
          \frac{dL}{dO(n,j+y+(pt-K+1), i+x+(pl-K+1))}*W(n,c,K-1-j,K-1-i)
        }
      }
    } \\
    &令\frac{dL}{dO}的padding为( K-1-pt , HI-HO+pt , K-1-pl , WI-WO+pl) , 得到 \frac{dL}{dOP} \\
    &=\sum_{n=0}^{N-1}{
      \sum_{j=0}^{K-1}{
        \sum_{i=0}^{K-1}{
          \frac{dL}{dOP(n,j+y, i+x)}*W(n,c,K-1-j,K-1-i)
        }
      }
    }
\end{aligned}
$$

$$
\begin{aligned}
  \frac{dL}{dI(n)}
  &=\sum_{n=0}^{N-1}{conv(\frac{dL}{dOP(n)} , W^T(n,c))}
\end{aligned}
$$
即$\frac{dL}{dI}$是$\frac{dL}{dOP}$与W旋转180°按输出维度的组合卷积并求和

#### 权重更新


* 对bias梯度

$$
\begin{aligned}
\frac{dL}{dB(n)}&=
    \sum_{h}^{HO-1}{
        dO(n,h,w)
    }
\end{aligned}
$$

* 对weight梯度

$$
\begin{aligned}
\frac{dL}{dW(n,c,y,x)}
&=\frac{dL}{dO}*\frac{dO}{dW(n,c,y,x)} \\
&=\sum_{j=0}^{H-1}{
  \sum_{i=0}^{W-1}{
    \frac{dL}{dO(n,j,i)}*\frac{dO(n,j,i)}{dW(n,c,y,x)}
  }
}\\
&=\sum_{j=0}^{H-1}{
  \sum_{i=0}^{W-1}{
    \frac{dL}{dO(n,j,i)}*IP(n,c,y,x)
  }
}\\
\end{aligned}
$$

因此，
$$
\begin{aligned}
\frac{dL}{dW(n,c)} = conv(\frac{dL}{dO(n)} , IP(c) )
\end{aligned}
$$


