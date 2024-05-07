## 一个小Trick看Python矩阵运算加速

### 1.Background

我们想求两个矩阵的欧几里得距离，定义如下：
$$
d_e\left( \left( \begin{array}{l}
	x_{11}&		x_{12}&		x_{13}\\
	...&		...&		...\\
	x_{n1}&		x_{n2}&		x_{n3}\\
\end{array} \right) ,\left( \begin{array}{l}
	y_{11}&		y_{12}&		y_{13}\\
	...&		...&		...\\
	y_{n1}&		y_{n2}&		y_{n3}\\
\end{array} \right) \right) =\left( \begin{array}{l}
	\sum{\left( x_{1i}-y_{1i} \right) ^2}&		\sum{\left( x_{1i}-y_{2i} \right) ^2}&		\sum{\left( x_{1i}-y_{ni} \right) ^2}\\
	...&		...&		...\\
	\sum{\left( x_{ni}-y_{1i} \right) ^2}&		\sum{\left( x_{ni}-y_{2i} \right) ^2}&		\sum{\left( x_{ni}-y_{ni} \right) ^2}\\
\end{array} \right)
$$
类似的还可以定义曼哈顿距离，
$$

d_e\left( \left( \begin{array}{l}
	x_{11}&		x_{12}&		x_{13}\\
	...&		...&		...\\
	x_{n1}&		x_{n2}&		x_{n3}\\
\end{array} \right) ,\left( \begin{array}{l}
	y_{11}&		y_{12}&		y_{13}\\
	...&		...&		...\\
	y_{n1}&		y_{n2}&		y_{n3}\\
\end{array} \right) \right) =\left( \begin{array}{l}
	\sum{\left| x_{1i}-y_{1i} \right|}&		\sum{\left| x_{1i}-y_{2i} \right|}&		\sum{\left| x_{1i}-y_{ni} \right|}\\
	...&		...&		...\\
	\sum{\left| x_{ni}-y_{1i} \right|}&		\sum{\left| x_{ni}-y_{1i} \right|}&		\sum{\left| x_{ni}-y_{1i} \right|}\\
\end{array} \right) 

$$

### 2.Coding

如果对性能没有要求，容易写出下面的代码来计算两个矩阵的欧式距离：

```python
def eulidean_distance_matrix(x,y):
    num_samples = x.shape[0]
    dis_matrix = np.empty((num_samples,num_samples))
    for i, xi in enumerate(x):
        for j, yj in enumerate(y):
            diff = xi - yi
            dis_matrix[i][j] = diff @ diff
     return dis_matrix
```

但是，这样逐个求值会导致计算速度非常慢，因此，我们的基本思路是尽量少写或者不写循环。

给出下面两种优化写法：

```python
 def eulidean_distance_matrix(x,y):
      diff = x[:, np.newaxis,:] - y[np.newaxis, : ,]
      return (diff * diff).sum(axis = 2)
```



```python
 def eulidean_distance_matrix(x,y): 
      x2 = (x * x ).sum(axis = 1)[:,np.newaxis]
      y2 = (y * y ).sum(axis = 1)[np.newaxis,:]
      xy = x @ y.T

      return np.abs(x2 + y2 - 2 * xy)
```

当然了， 还可以使用numba库来加速。