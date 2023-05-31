## Intrinsic Parameter Matrix
![[intrisic_parameter_matrix.png]]
The part of the matrix highlighted in this picture is called the __intrinsic parameter matrix__, and it is the leftmost part of $P_{int}$. 
It is usually referred to as $𝑨$ or $𝑲$ and it _models the characteristics of the image sensing device_ (camera). It contains the four parameters that [[2023-04-20 - 3D to 2D recap, Intrinsic and Extrinsic parameters, projection space#Final, intrinsic parameters|we already introduced]].
- It is always an upper right _triangular matrix_ (it will always have some 0s in the lower left corner).
A notation that we use is this: $[A|0] = P_{int}$. 

##### Skew (trad. storta) [skippable...]
A more general model would include a 5th parameter, known as __skew__, to account for possible _non-orthogonality between the axes of the image sensor_. 

The skew would be $A[1,2]$, but it is usually 0 in practice.

This encodes any possible skew between 
- the sensor axes due to the sensor not being mounted perpendicular to the optical axis, or 
- it may be because of issues in the manufacturing process (in the sensor the pixel rows and columns are not perfectly perpendicular).

## Extrinsic Parameter Matrix
If we consider the general case of 3D points expressed in a world projective space, the relationship between CRF and WRF can also be expressed with one 4x4 matrix $𝑮$, the __extrinsic parameter matrix__.
![[ext_parms_matrix.png]]

## Perspective projection matrix
![[perspective_projection_matrix.png]]
$𝑷$ is known as the __perspective projection matrix__ (PPM), and has dimension 3x4. 

## Canonical perspective projection
𝑷 is a 3x4 matrix with _full rank_. The most basic PPM is
$$
P ≡ [𝑰 |0]
$$
This form is useful to understand the core operations carried out by perspective projection, i.e. _scaling_ lateral coordinates ($𝑥, 𝑦$) according to the distance from the camera ($𝑧$). 
The above form is usually referred to as __canonical or standard PPM__.

A general PPM can then be factorized as $𝑷 ≡ 𝑨 [𝑰 | 𝟎] 𝑮$, i.e. it can be thought of as carrying out _three fundamental operations in sequence_:
1. __Convert__ coordinates from _WRF to CRF_ 
2. Perform _canonical perspective projection_, i.e. divide by the third coordinate 
3. Apply _camera specific transformations_: i.e. where is my reference frame etc... (intrisic params)

From it, we get another useful factorization of a generic PPM as 
![[useful_factorization.png]]

>[!approfondimento]
>Why does $[I|0]$ work as a PPM? Well, because: $𝑨 [𝑰 | 𝟎] = [A|0]$

## Lens distortion
The PPM is based on the pinhole camera model.
However, _real lenses_ introduce _distortions_ w.r.t. to the pure pinhole model (in particular, for cheap and/or short focal length lenses).
We often need to model also the effects caused by the optical distortion induced by lenses 
- _Lens distortion_ is modelled through _additional parameters_ that _do not alter_ the form of the _PPM_.
![[lens_disto.png]]

When introducing these new aspects, we do not need to change the camera model, we stay with the pinhole camera model. Infact, we never once changed the model from the pinhole camera model. 

>[!approfondimento]
>How do we know that it is not our perspective projection model that generates this distortions, and is in fact the lens? 
>Well, perspective projections preserve _colinearity_ (it preserves lines), though _it does not preserve parallelism_ (the angles change). 
>In reality, different position of the lenses will bend light differently.

## Modeling lens distortion
When have 2 main deviation from the results obtained by a pinhole camera model:
1. The most significant deviation from the _ideal pinhole model_ is known as __radial distortion__ (lens “curvature”). It depends on the image radius -> is _more evident the further_ you are. 
2. Second order effects are induced by __tangential distortion__ (“misalignments” of _optical components_ and/or defects). 

Lens distortion is modelled through a non-linear transformation which maps ideal (i.e., undistorted) image coordinates into the observed (i.e., distorted) image coordinates.

![[lens_distortion_coords.png]]

$(x, y)$ is the actual output of the image, while $x_{undistorted}$ refers to the undistorted, real image that we would have with a ideal model.
The _output is influenced_ by the __distance__ $r$ from the __distortion centre__, which is usually assumed to correspond with the _piercing point_ $c = [0,0]^T$, $r = \sqrt{(x_{undist})^2 + (y_{undist})^2)}$.

>[!WARNING]
>As you can see, we're not using $(u,v)$ as notation for expressing the image coordinates, but rather we are using $(x,y)$. 
>This is because, while we're technically on the image plane, our coordinates are not pixelized yet. 

### Types of lens distortions
- __Barrel distortion__ is a lens defect that causes _straight lines_ to _bend outwards_ and is most noticeable along the edges 
	- It is associated with wide-angle lenses 
- __Pincushion distortion__ is the opposite of barrel distortion, it causes _straight lines_ to _curve inward_ from the edges to the middle 
	- Telephoto lenses are most commonly associated with pincushion distortion
![[radial_barrel_distortion.png]]

### Modelling lens distortion 2
The _radial distortion function_ $L(r)$ is defined for positive r only and such that $L(0) = 1$. This non-linear function is typically approximated by its _Taylor series_ (up to a certain approximation order)
$$
L(r) = 1 + k_1r^2 + k_2r^4  + k_3r^6
$$
The _tangential distortion vector_ is instead approximated as follows:
![[tan_dist.png]]
The parameters $p_1$ and $p_2$ weight the _amount of distortion_ that we have in the model. 

The _radial distortion coefficients_ $𝑘_1 , 𝑘_2 , … , 𝑘_n$ together with the _two tangential distortion coefficients_ $𝑝_1$ and $𝑝_2$ form the set of the lens distortion parameters, which extends the set of __intrinsic parameters__ required to build _a realistic camera model_.

### Lens distortion within the image formation process
Lens distortion is modelled as a non-linear mapping _taking place after canonical perspective projection_ onto the image plane. Afterwards, the intrinsic parameter matrix allow applying an affine transformation which maps continuous image coordinates into pixel coordinates.
Accordingly, the image formation flow with lenses can be summarized as follows:
![[list_of_stuff.png]]
Why are we handling lens distortion before the intrisic parameters of the camera sensor?
Well, it's because that's what happens in practice -> otherwise, positioning the object in the WRF would be affected by inaccuracies. 

## Complete camera model
![[complete_camera_model.png]]
![[camera_calib_2.png]]
Given (a lot of) pairs $(𝑚^{(𝑖)} , 𝑀_𝑊^{(𝑖)} )$, estimate the parameters $M_c$ for a specific camera and setup.

>[!WARNING]
>Since lens distortion depends on your specific sensor, they are thought of intrinsic parts. 

Now that we have full camera model, we'll need to solve this:
- Given (a lot of) pairs $(𝑚^{(𝑖)} , 𝑀_𝑊^{(𝑖)} )$, _estimate the parameters_ for a specific camera and setup.

----
# Calibration
We have now defined a camera model thoroughly explaining the image formation and digitization process. Such a model is the _PPM_ $𝑷$ (+ _distortion coefficients_), which in turn can be decomposed into 3 independent elements: 
- __intrinsic parameter matrix__ (𝑨) 
- __rotation matrix__ (𝑹) 
- __translation vector__ (𝒕) 

_Camera calibration_ is the process whereby all parameters defining the camera model are (as accurately as possible) estimated for a specific camera device. 
Depending on the application, either the PMM only or also its independent components $(A, R, t)$ need to be estimated.

## Calibration patterns
Camera calibration approaches can be split into two main categories: 
- Those relying on a _single image_ of a 3D calibration object 
	- featuring several (at least 2) planes containing a known pattern 
- Those relying on _several_ (at least __3__) _different images_ of one given planar pattern.

In practice, it is difficult to build accurate targets containing multiple planes, while an accurate planar target can be attained rather easily Implementing a camera calibration software requires a significant effort 
- the main Computer Vision toolboxes include specific functions (OpenCV, Matlab CC Toolbox)

## Zhang's method
![[zhang_method.png]]
## Zhang's step 1 - Acquire 𝑛 images of a planar pattern with 𝑚 internal corner
### Calibration pattern
Given a chessboard pattern, we know: 
- The _number of internal corners_ of the pattern, usually odd along one dimension and even along the other to _remove rotation ambiguities_.
	- (we want the patterns at the edges to be very different from each other). 
- The _size of the squares_ that form the pattern (in mm, cm...)
Internal corners can be detected easily by standard algorithms (e.g. the __Harris corner detector__)
![[camera_calib_noamb.png]]
### 3D coordinates
The World Reference Frame can be conveniently defined.
In an unambiguous pattern, the WRF:
- _has its origin always_ in the same corner (e.g., the one next to the dark square on the right of the chessboard if both dark squares are on top) 
- has plane $𝑧 = 0$ given by the pattern itself => the third coordinate is always 0 
- and the $𝑥, 𝑦$ axes are _aligned to the chessboard_ (e.g., 𝑥 along the short side and 𝑦 along the long one).
	- and ==they must be consistent==!!!!
Given such rules and the known square side, it is possible to define 3D coordinates for all corners in an image of the pattern.
![[image_chessboard.png]]

## Extrinsic parameters
The World Reference Frame is different for each calibration image. _The $[𝑹 \ 𝒕]$ are estimated wrt the World Reference System attached to the target_, which _moves with the pattern_. Therefore, __we estimate as many extrinsic matrices as the number of images used for calibration__ (usually 10 to 20 images of the pattern).
![[checkboard_1.png]]

The WRF remains the same in any direction of the pattern, it doesnt change if I move the camera. 

Can it be the same PPM for the two images?
- NO! SInce we have a different relationship between the input, the same 3D point and the camera.

What changes in these PPMs is the relative position of the camera. 
- Some things _remains unchanged_: $A$, $k_1... k_n$, $p_1$ and $p_2$ remain the same (intrinsic stuff). 

## Zhang step 2 -> For each image compute an initial guess homography $𝐻_𝑖$

>[!WARNING]
>For this step, we are assuming that we have no lenses, so we are ignoring their effect. 

### 𝑃 as a Homography
Due to the choice of the WRF associated with calibration images, in each of them we consider only 3D points with $𝑧 = 0$. 
Accordingly, the PPM for points on the pattern can _be simplified_ to a __3x3 matrix__ (remember, normally the PPM has 3x4 dimensions):
![[P_as_homography.png]]
Such a transformation, denoted here as $𝑯$, is known as __homography__ and represents a general transformation between projective planes.
- Essentially, $H$ can be seen as a _relation_ between the _pattern plane_ and the _image plane_. 
$𝑯$ can be thought of as a simplification of $𝑷$ in case the imaged object is planar.

## Estimating $𝐻_𝑖$ (DLT algorithm)
Given the image of a pattern with $𝑚$ corners, we can write 3 linear equations for each corner $𝑗$ where: 
- 3D coordinates are known due to the WRF definition 
- 2D coordinates are known due to corners having been detected in the $i$-th image ◦
- the unknowns are the 9 elements in $𝐻_i$

There an would be $H_i$ for _each image_ ($n$). 

![[equations.png]]

Stacking $3𝑚$ such equations for the $𝑚$ corners we get a system of equations, but… How do we solve a system of equations in a projective space?
- In fact, we do not have $=$ signs on our equation, but rather _equivalence signs_, since as we know points in a projective space are equivalent _up to a certain scale_. 
	- [[2023-04-20 - 3D to 2D recap, Intrinsic and Extrinsic parameters, projection space#Projective Space|Remember?]]

When are two 3D points _equivalent_ in $ℙ^2$?
![[two_points_equivalent.png]]
As we know, in projective space 2 points are equivalent they stay on the same line, and... 
- Two points lay on the same line if their _cross product_ is the _zero vector_:

![[estimating_algorithm.png]]

Given $𝑚$ corners, we can create a _homogeneous_, _overdetermined linear system of equations_:
![[bho.png]]

In standard linear algebra, there's no solution to that (we can only compute solution if the matrix is squared/full rank). 

To avoid the trivial solution 𝒉 = 𝟎 we look for solutions with an additional constraint, e.g., $||𝒉|| = 1$.

#### Singular Value Decomposition
The solution $𝒉^∗$ is found by minimizing the norm of the vector $𝑳h$:
![[svd.png]]
What I get is a solution ($h*$) that is not 0, but it will be close to it. -> my final point won't perfectly be on the line, but it will be close.  

It is known from linear algebra that the solution to such problem can be found via __Singular Value Decomposition__ of $𝐿$. In particular, the ==solution== is $𝒉^∗ = 𝒗_𝟗$, i.e., the last column of $V$:
![[svd_2.png]]
