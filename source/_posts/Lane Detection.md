---
title: Lane Detection
tags:
  - Edge Detection
  - CS131
categories:
  - Computer Vision
date: 2018-07-09 11:37:31
---

簡述Lane Detection

# Canny Edge Detector

利用Canny Edge Detector找edge

## Suppress Noise(降噪)
Smoothing
We first smooth the input image by convolving it with a Gaussian kernel. The equation for a Gaussian kernel of size \\((2k+1)\times(2k+1)\\) is given by:
$$h_{ij}=\frac{1}{2\pi\sigma^2}\exp{\Bigl(-\frac{(i-k)^2+(j-k)^2}{2\sigma^2}\Bigr)}, 0\leq i,j < 2k+1$$
## Finding gradients
The gradient of a 2D scalar function \\(I:\mathbb{R}^2\rightarrow{\mathbb{R}}\\) in Cartesian coordinate is defined by:
$$\nabla{I(x,y)}=\bigl[\frac{\partial{I}}{\partial{x}},\frac{\partial{I}}{\partial{y}}\bigr],$$
where,
$$
\frac{\partial{I(x,y)}}{\partial{x}}=\lim_{\Delta{x}\to{0}}\frac{I(x+\Delta{x},y)-I(x,y)}{\Delta{x}} \\
\frac{\partial{I(x,y)}}{\partial{y}}=\lim_{\Delta{y}\to{0}}\frac{I(x,y+\Delta{y})-I(x,y)}{\Delta{y}}.
$$

In case of images, we can approximate the partial derivatives by taking differences at one pixel intervals:
$$
\frac{\partial{I(x,y)}}{\partial{x}}\approx{\frac{I(x+1,y)-I(x-1,y)}{2}} \\
\frac{\partial{I(x,y)}}{\partial{y}}\approx{\frac{I(x,y+1)-I(x,y-1)}{2}}
$$

Note that the partial derivatives can be computed by convolving the image $I$ with some appropriate kernels $D_x$ and $D_y$:
$$
\frac{\partial{I}}{\partial{x}}\approx{I*D_x}=G_x \\
\frac{\partial{I}}{\partial{y}}\approx{I*D_y}=G_y
$$
Now, we can compute the magnitude and direction of gradient with the two partial derivatives:
$$
G = \sqrt{G_{x}^{2}+G_{y}^{2}} \\
\Theta = arctan\bigl(\frac{G_{y}}{G_{x}}\bigr)
$$

## Non-maximum suppression
現在圖片thick又blurry
所以我們要做的事情是把
**blurry img**變成**sharp img**
保留局部低度最大值然後把其他東西丟掉
	1. 把梯度方向移動到最近的45度倍 常見的8個方向
	2. 沿著梯度方向有正向和反向 比較這兩個方向的值 留大的 把小的丟棄
### Code
```py
def non_maximum_suppression(G, theta):
    """ Performs non-maximum suppression
    This function performs non-maximum suppression along the direction
    of gradient (theta) on the gradient magnitude image (G).
    
    Args:
        G: gradient magnitude image with shape of (H, W)
        theta: direction of gradients with shape of (H, W)
    Returns:
        out: non-maxima suppressed image
    """
    H, W = G.shape
    out = np.zeros((H, W))

    # Round the gradient direction to the nearest 45 degrees
    theta = np.floor((theta + 22.5) / 45) * 45

    ### BEGIN YOUR CODE
    theta = theta%360
    for i in range(1, H-1):
        for j in range(1,W-1):
            current_angle = theta[i,j]
            if current_angle == 0 or current_angle == 180:
                neighbors = [G[i, j-1], G[i, j+1]]
            elif current_angle == 45 or current_angle == 225:
                neighbors = [G[i-1, j-1], G[i+1, j+1]]
            elif current_angle == 90 or current_angle == 270:
                neighbors = [G[i-1, j], G[i+1, j]]
            elif current_angle == 135 or current_angle == 315:
                neighbors = [G[i-1, j+1], G[i+1, j-1]]
			
			# 和兩個鄰居互相比較
            if G[i,j] >= np.max(neighbors):
                out[i,j] = G[i,j]
            else:
                out[i, j] = 0 # 丟棄

    ### END YOUR CODE

    return out
```

	
## Double Thresholding
接下來剩下的edge-pixels有些是真的edge有些可能造成noise或變色
解決方法是 設定兩個 threshold high , low
if img > high : Strong edge
else if img > low and img < high : Weak edge

### Code
```py
def double_thresholding(img, high, low):
    """
    Args:
        img: numpy array of shape (H, W) representing NMS edge response
        high: high threshold(float) for strong edges
        low: low threshold(float) for weak edges
    Returns:
        strong_edges: Boolean array representing strong edges.
            Strong edeges are the pixels with the values above
            the higher threshold.
        weak_edges: Boolean array representing weak edges.
            Weak edges are the pixels with the values below the
            higher threshould and above the lower threshold.
    """

    strong_edges = np.zeros(img.shape)
    weak_edges = np.zeros(img.shape)

    ### YOUR CODE HERE
    strong_edges = img > high
    weak_edges = (img < high) & (img > low)

    ### END YOUR CODE

    return strong_edges, weak_edges
```


## Edge tracking
上一步的Strong edge 就直接保留了
而Weak edge呢 則要看他是不是和Strong edge相鄰來決定
相鄰則留否則不留


# Extracting region of interest (ROI)
儘管Canny edge detector得到邊緣
但我們現在只要lane 所以要把其他東西去掉
算法很簡單
設我們有興趣的地方為**Mask**
`Roi = Edges * Mask`

# Fitting lines using Hough transform
目前結果仍是一堆點 所以我們要把他連成線
![](https://render.githubusercontent.com/render/math?math=%5Crho%20%3D%20x%5Ccdot%7Bcos%5Ctheta%7D%20%2B%20y%5Ccdot%7Bsin%5Ctheta%7D&mode=display)
![](https://render.githubusercontent.com/render/math?math=%5Ctheta%5Cin%7B%5B-%5Cpi%2C%20%5Cpi%5D%7D&mode=inline) , ![](https://render.githubusercontent.com/render/math?math=%5Crho%5Cin%7B%5Cmathbb%7BR%7D%7D&mode=inline)
計算這些點在 Hough Space的值 然後選最大的那些點
## Code
```py
def hough_transform(img):
    """ Transform points in the input image into Hough space.
    Use the parameterization:
        rho = x * cos(theta) + y * sin(theta)
    to transform a point (x,y) to a sine-like function in Hough space.
    Args:
        img: binary image of shape (H, W)
        
    Returns:
        accumulator: numpy array of shape (m, n)
        rhos: numpy array of shape (m, )
        thetas: numpy array of shape (n, )
    """
    # Set rho and theta ranges
    W, H = img.shape
    diag_len = int(np.ceil(np.sqrt(W * W + H * H)))
    rhos = np.linspace(-diag_len, diag_len, diag_len * 2.0 + 1)
    thetas = np.deg2rad(np.arange(-90.0, 90.0))
    # Cache some reusable values
    cos_t = np.cos(thetas)
    sin_t = np.sin(thetas)
    num_thetas = len(thetas)
    # Initialize accumulator in the Hough space
    accumulator = np.zeros((2 * diag_len + 1, num_thetas), dtype=np.uint64)
    ys, xs = np.nonzero(img)
    # Transform each point (x, y) in image
    # Find rho corresponding to values in thetas
    # and increment the accumulator in the corresponding coordiate.
    ### YOUR CODE HERE

    # it might be better to work with a sparse matrix representation (as opposed to dense representation)
    for y,x in zip(ys, xs):
        for t_idx, theta in enumerate(thetas):
            rho = x*cos_t[t_idx] + y * sin_t[t_idx]
            # Finding our idx in the linear space
            rho_idx = int(rho +diag_len)
            accumulator[rho_idx, t_idx]+=1

    ### END YOUR CODE

    return accumulator, rhos, thetas
```