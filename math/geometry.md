
[toc]

[math](./math.md)


* general coordinates
    * scaling
        $$
        S_v = 
            \begin{bmatrix} v_x & 0 & 0 \\ 0 & v_y & 0 \\ 0 & 0 & v_z \\ \end{bmatrix}
            \begin{bmatrix} p_x \\ p_y\\ p_z\\ \end{bmatrix} 
            = \begin{bmatrix} v_xp_x \\ v_yp_y\\ v_zp_z\\ \end{bmatrix}
        $$
* homogeneous coordinates(齐次坐标系)
    * background
        * normal point
            * euclidean coordinates: $(x, y)$
            * homogeneous coordinates: $(x, y, 1)$
        * inifinty
            * euclidean coordinates: $(\infty, \infty)$
            * homogeneous coordinates: $(x, y, 0)$
        * scale invariant(尺度不变性，齐次的含义)
        * meaning
            * distinguish point $(x, y, 1)$ and vector $(x, y, 0)$
            * enable affine transformation
    * scaling
        $$
        S_v = 
            \begin{bmatrix} v_x & 0 & 0 & 0 \\ 0 & v_y & 0 & 0 \\ 0 & 0 & v_z & 0 \\ 0 & 0 & 0 & 1 \\ \end{bmatrix}
            \begin{bmatrix} p_x \\ p_y\\ p_z\\ 1\\ \end{bmatrix} 
            = \begin{bmatrix} v_xp_x \\ v_yp_y\\ v_zp_z\\ 1\\ \end{bmatrix}
        $$
* gradient of images
    * gradient of a point: 
        $$gradf(x,y)=\frac{{\partial}f}{{\partial}x}\vec{i}+\frac{{\partial}f}{{\partial}y}\vec{j}$$
    * vector of this gradient
        $$\bigtriangledown{f(x,y)}=[\frac{{\partial}f}{{\partial}x}, \frac{{\partial}f}{{\partial}y}]^T$$
    * magnitude of this gradient
        $$mag(\bigtriangledown{f(x,y)})=\sqrt{\frac{{\partial}f}{{\partial}x}^2+\frac{{\partial}f}{{\partial}y}^2}$$
    * angle of this gradient
        $$\phi(x,y)=arctan|\frac{{\partial}f}{{\partial}x}/\frac{{\partial}f}{{\partial}y}|$$