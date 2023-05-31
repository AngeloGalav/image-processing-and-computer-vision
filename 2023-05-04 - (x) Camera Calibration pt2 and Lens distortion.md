## Zhang Part 3 -> Refine each $𝐻_𝑖$ by minimizing the reprojection error
### What error should we minimize?
![[optmiizi_h.png]]
When finding the homography $H$, we wanted to find an H that optimizing on top of $\tilde w_j$. 
We are not getting a perfect H, since we are finding it in LSQ sense and we're neglecting the lens distortion. And most definetly, we are not going to find it using a simple linear model. 
Our $\tilde w_j$ will map something close to $m_j$, but it is not quite the exact value yet.

There's another problem though:
- $H_i$ is a 3x3 matrix (like we saw). 
- $w_j$ (with no tilde) has 4 coordinates [x, y, z, 1] (with the tilde, it has 3 coordinates)

But, the notation is done like this in order to make it clear that we are in the pixel space: it's a multiplication resulting in a 2x1 vector, since I divide by a 3rd coordinate ($h_3$) each row of the matrix (in order to escape from the perspective space). 
- This is because we're trying to project the image into the _Euclidean space_. 

We can try to improve $H$ by making this distance (between $H_iw_j$ and $m_j$) as short as possible -> We want to minimize $||m_j - H_iw_j||$. 
- By solving [[2023-04-27 - PPM, handling lenses, intro to camera calibration#Estimating $𝐻_𝑖$ (DLT algorithm)|the linear system of equation]], we are trying to minimize the _algebraic error_, while when we are minimizing $||m_j - H_iw_j||$ we are minimizing the _geometric error_.

##### In short...
Given the initial guess for $𝑯_𝒊$ , we can refine it by a non-linear minimization problem:
![[refinement_H.png]]
which can be solved for by using an iterative algorithm, like the Levenberg-Marquardt algorithm (similar to SGD). 
This additional optimization step corresponds to the minimization of the reprojection error (typically referred to as geometric error) measured for each of the 3D corners of the pattern by comparing the _pixel coordinates predicted by the estimated homography_ to the _pixel coordinates_ of the corresponding corner extracted in the image ($m_j$). 

The error minimized to estimate the initial guess when solving the linear system is instead referred to as _algebraic error_ or distance. Solutions based on minimization of the algebraic error may not be aligned with our intuition, yet there exist a unique solution, which is cheap to compute. Hence, they are a good starting point for a geometric, non-linear minimization, which effectively minimize the distance we care about.

>[!approfondimento]
>Why are we doing the 2-step error minimization (algebraic + geometric) instead of just using the geomtric minization?
> This is because iterative methods are very sensistive to initial parameter values (i.e. the first parameter of the iteration) etc...
> - The algebraic error serves to find a good initial guess essentially.  

## Zhang's Method 4 -> Estimation of the intrinsic parameters (initial guess of A)
Up to now, if we got $n$ images we have computed an homography between the world pattern and the pattern between the $i$ images. Why can't we stop right now?

With calibration, we want to estimate _perspective projection matrix_ and _the parameters_ (intrisinc and extrinsic).
- The $H$ (homography) projection matrix is very close to the perspective projection matrix, we just need another to find $P$ from $H$.

To get the 4th column, we need $A$, which is an _intrisic parameter_. But all images acquired for calibration (if taken with the same camera) share the same _intrinsic parameters_.  

We can establish the following relations between them and the extrinsic and intrinsic parameters:
![[ext_intr.png]]
$H_i$ we get from the images, which gives a 3x3 matrix H. 
But why is there a $k$ parameters (which is not the parameter k for lens distortion)? 
- I have to estimate $k$, which can be seen as a _proportional factor_. This is because, as always, of the relation of equivalence in the proportional space. 

In the final part of this equation (the one with the 2 equivalences), I know only the $h_{ik}$, the other parameters are unknown. But...
$r_{i1}, r_{i2}$ are column vectors of a _rotation matrix_, so they are _orthogonal_, so -> 
- the $k$ is not important then!
- And...
Since the column vectors of each $𝑹_𝒊$ are orthonormal, we get the following constraints:
![[ok1.png]]
Oh great, now I only need to find $A^TA^{-1}$ (since the other $h$s are known) 😀.

Also, since the column vectors of $R$ are orthonormal, _their norm is $k$_, so:
![[norm_conseq.png]]
I can equate the 2 parts of the linear system. 

By stacking these two constraints for each image, we get a homogeneous system of equations which can be solved again by SVD if $𝑛 ≥ 3$ images are collected (_6 unknowns_ since $𝑨^{−𝑻}𝑨^{−𝟏}$ is symmetric).
-> $2n$ _constraints_. 

This is also the reason __why__ _we need at least_ $n = 3$ images for camera calibration. 

## Zhang's Method 5 -> Estimation of the extrinsic parameters ($R_i$ and $t_i$) given $A$ and $H_i$
Once $𝑨$ has been estimated, it is possible to compute $𝑹_𝒊$ and $𝒕_𝒊$ (for each image) given $𝑨$ and the previously computed homography $𝑯_𝒊$:
![[extr.png]]
As $𝒓_{𝒊𝟏}$ is a unit vector, the normalization constant can be computed as $k = ||A^{-1}h_{i1}||$.
Then, the same constant can be used to compute:
![[nice_ext.png]]
There's no guarantee that $r_{i2}$ and $r_{i1}$ wil be unit vectors, or orthogonal (because of noise and other errors). So, we need to _enforce_ it.
However, SVD of $𝑹_𝒊$ allows _to find the closest orthonormal matrix to it_ by substituting $𝑫$ with $𝑰$.

These estimations of $R_i$ and $t_i$ are not perfect [..], we just need some good guesses in order for the iteration for converge. 

## Lens distortion coefficients - Zhang's Method 6
- Compute an initial guess for distortion parameters $𝑘$ and $p$.

So far, we have neglected lens distortion and calibrated a _pure pinhole model_. The coordinates predicted by the homographies starting from points in the WRFs correspond to the ideal (undistorted) pixel coordinates of the chessboard corners $𝑚_{𝑢𝑛𝑑𝑖𝑠𝑡}$. The measured coordinates of the corners in the images are the real (distorted) coordinates $𝑚$.

The Original Zhang's model only takes into account the _radial distortion_, and estimates coefficients $𝑘_1$ , $𝑘_2$ of the radial distortion function:
![[radial_dist.png]]
Where $L(r)$ is the Taylor series, and r is the distance from the image center (aka the norm between $x_{undist}$ and $y_{undist}$)

The lens distortion model is applied _before pixelization_, so as we can see the formula doesn't not use $u$ and $v$.
$x_{undist}$ and $y_{undist}$ are the ideal undistorted coordinates. 
To compute $u$ and $v$, we need $x_{undist}$ and $y_{undist}$, so it's a chicken and the egg problem. To solve this, we need to use an approximation. 

### Metric image coordinates
Recall: lens distortion takes place _before_ we change metric image coordinates to pixel coordinates. But we measure and predict pixel coordinates.

We can transform back pixel coordinates $\begin{bmatrix}  u\\  v\end{bmatrix}$ to metric image coordinates  $\begin{bmatrix}  x\\  y\end{bmatrix}$ thanks to the estimated intrinsic matrix $A$: 
![[matrix_image_coords.png]]
The same transformation holds between $𝑢_{𝑢𝑛𝑑𝑖𝑠𝑡}$, $𝑣_{𝑢𝑛𝑑𝑖𝑠𝑡}$ and $𝑥_{𝑢𝑛𝑑𝑖𝑠𝑡}$, $𝑦_{𝑢𝑛𝑑𝑖𝑠𝑡}$
Then, the distortion equation in pixel coordinates become:
![[transformation.png]]

### Lens distortion coefficients
![[lens_distortion.png]]
We get a linear, non-homogeneous system of linear equations 𝑫𝒌 = 𝒅 in the unknowns 𝒌 =  𝑘1 𝑘2 𝑇 . With 𝑚 corners in 𝑛 images we get 2𝑛𝑚 equations in 2 unknowns, which can be solved in a least square sense, i.e., minimizing 𝑫𝒌 − 𝒅 2 , by computing the pseudo-inverse matrix $𝑫^†$ as
![[formula_cazzo.png]]

## Refinement by non-linear optimization - Zhang’s Method 7
- Refine all parameters $𝐴, 𝑅_𝑖 ,𝑡_𝑖 , 𝑘, 𝑝$ by minimizing the reprojection error
Now that we have everything, we now need to refine all the approximations that we have. 

Final non-linear refinement of the estimated parameters. As for homographies, the procedure highlighted so far seeks to minimize an _algebraic error_, without any real physical meaning.
A more accurate solution can instead be found by a so called _Maximum Likelihood Estimate (MLE)_ aimed at _minimization of the geometric_ (i.e. reprojection) error.
We use all the values estimated so far as initial guesses. Under the hypothesis of i.i.d. (_independent identically distributed_) noise, the MLE for our models is obtained by minimization of the error:
![[refinemnet.png]]

# End of Camera Calibration - What do we do now? WARPING!
## Warping to compensate lens distortion
Lens distortion makes the system non-linear and cumbersome to use. Once a camera has been calibrated, we can _warp_ the images it takes so to simulate a system _without lens distortion_. 
_Warping_ refers to transformations of the _spatial domain_ of images, while _filtering_ refers to changes _in the values of images_:
![[warping.png]]So, in short, warping does not alter the content of the image, but it changes the positions of the pixels (it changes the spatial domain). 
On the other hand, filtering changes the color the pixel. 

### Image Warping
If we have a function that computes point in image $I’$ starting from point in image $I$, we can copy the value:
![[warping_2.png]]
Can be just a rotation:
![[warping_3.png]]
Or it can be a full homography:
![[warping_4.png]]
- find the corners with, for example, Harris. 
- choose 4 ideal corners.
- compute the _homography_. 
This is called __Forward Mapping__.

### Forward Mapping
After applying the warping function in general we get _continuous coordinates_, not discrete ones. Possible choices to make it discrete: truncate, neareast neighbor (i.e. rounding),...
![[forward_mapping.png]]

##### Folds and holds
Regardless of the choice of the mapping, due to rounding 
- more than one pixel can go to ==one position== (_folds_) 
- some pixels of the destination image may not be hit (_holes_)
![[forward_mapping_2.png]]

### Backward mapping
We can avoid these problems if we take a value for each coordinate in the output image by using the inverse mapping.
Yet, the coordinate are still continuous values. Which mapping strategy? 
- Truncate 
- Nearest Neighbour 
- _Interpolate between the 4 closest point_ (bilinear, bicubic, etc…)
![[backward_mapping.png]]
This solves holes by design, since we are trying to find a correspondant for each pixel.  

### Bilinear Interpolation
![[bilinear_interpolation.png]]

### Undistort warping
