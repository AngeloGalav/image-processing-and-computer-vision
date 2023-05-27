# Pinhole camera and stereo vision intro
## Pinhole camera model
The simplest model we can think for an imaging device is the pinhole camera model. It has:
- A _tiny hole_ on one side
- A photosensitive material on the other (i.e. CMOS)
Light goes through hole and hits material. 
The image is achieved by drawing _straight rays_ from scene points through the hole up to the image plane
![[pinhole.png]]

### Perspective projection
![[projections.png]]

#### Coordinates systems
- Coordinates on the _image plane_ -> [u, v] (__image reference system__, IRS) (2D)
- Coordinates on the _focal plane_ -> [x, y, z] (__camera reference system__. CRS) (3D)
- $(0,0,0)$ is the _pinhole coordinates_, $C$. 

#### Conversion between systems
![[disegnino_del_prof.png]]
![[formulas.png]]

## Stereo vision
When mapping 3D coordinates into a 2D space, we are losing some information. 
One way to solve this is to have multiple cameras, say two cameras, and find correspondences between the points recorded by this 2 cameras. 

#### Assumptions:
- The axes of the focal plane _must_ be parallel. 
- The image planes must have the _same focal length_. 
In short:
- ==The cameras must be _perfectly aligned_ (which is impossible in the real world)==.

This also means that the difference between the 2 _reference frames_ is just a simple translation $b$, called __baseline__. 

$d = u_L - u_R$ defines the __disparity__. ==From disparity, we can derive depth==.   

The formula (__fundamental relationship in stereo vision__)
$$
z = \dfrac{b \cdot f}{d}
$$
There are a couple of issues though:
- No perfect alignment in depth in the real world. 
- You need to sense two images at the __very same moment__. 
- You can put the two cameras “as you want” but they must observe the same object. 

---
# Perspective projections and lenses
### Stereo Matching
- given a point $p_L$, I want to find $p_R$. 
To verify if two points are perfectly _aligned horizontally_, we simply draw an horizontal line that goes from $p_L$ to $p_R$ (__stereo matching__). 
![[stereo_matching.png]]
### Epipolar Geometry (epipolar line)
What if we need to find the correspondence between $p_R$ and $p_L$ and the 2 planes are _not aligned_? In that case, we could still find the other point by projecting $p_L$ onto $\pi_R$ (the right plane), and _search across the line_ (__epipolar line__).
![[epipolar_line.png]]
We still have some issues though, namely that this process is not efficient and that the cameras still are not perferctly aligned.

## Rectification 
What people do in practice is to convert _epipolar geometry_ to _standard geometry_ (Rectification / Warping / __Homography__)
- We compute a transformation to _both images_.

Given a point in one image L, we find another point in the other image R that is the projection of the same 3D point. 
- Beware: points closer to the camera have a larger disparity than points which are further away from the camera. 

#### Properties of perspective projections
- Ratios/parallelism (for lines that are not parallel to the image plane) are not preserved.
- Lengths are inverse to the depth ($z$).

In perspective projections, all lines that are not parallel to the image plane meet in a __vanishing point__.

## Lenses
We need lenses in order to overcome the drawbacks of the pinhole camera. 
$$
	\dfrac{1}{u} + \dfrac{1}{v} = \dfrac{1}{f}
$$
![[lenses.png]]
Here's what each variable represents:
- $P$ : scene point.
- $p$ : corresponding focused _image point_.
- $u$ : distance from $P$ to the lens. 
- $v$ : distance from $p$ to the lens.
- $f$ : focal length (parameter of the lens) 
- $C$ : centre of the lens 
- $F$ : focal point (or focus) of the lens

Our images are now brighter (and require less exposure times), but we pay in terms of DOF
-> we need to focus (position the image plane) properly!

---
# Cameras
## Circles of confusion
If the circles of confusion are smaller than the size of the photosensing elements of the camera, the image will still look on-focus. The range of distances of which the images appears on focus is also called DOF of the camera. 

### Diaphragram
The length of $v$ (aka the distance between the the image plane and the lens) is adjustable in cameras. 
- initial position, v=f  -> u=$\infty$
- then, $\uparrow v \implies \downarrow u$  

## Signal to noise ratio