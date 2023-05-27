# Images
An imaging device (i.e. a camera or other sensors) gathers the light reflected by 3D objects to create a 3D representation of the scene.
- CV vs. CG: in CV we try to reconstruct a scene/image or get information from it, in CG we have a model of the scene, and we're trying to make a realistic approximation of it.

In computer vision we basically try to invert such a process, so as to _infer knowledge on the objects from one or more digital images_. 

Image formation and acquisition process includes: 
- The geometric relationship between scene points and image points.
	- Meaning, real points in space. 
- The radiometric relationship between _the brightness of image points_ and the _light emitted by scene points_. 
	- We're probably not going to cover it. 
- The image digitization process
	- How we transform light in a matrix of pixel?

## Pinhole camera model
The _simplest model_ we can think for a camera/imaging device is a box, a box with a _tiny hole_ in the front, named __pinhole__, and somehow, on the other part, on the back of this box, we have a _photosensitive material_, like an analog film or a CMOS, something that is sensitive to light. 
![[pinhole.png]]
Geometrically, the image is achieved by drawing _straight rays_ from _scene points_ through the _hole_ up to the image plane.

- The light will hit the object, which will be in turn reflected on the material. 
- We can see that the object is projected upside-down on the film (in particular, reversed on both axes). 

While its a good approximation of the geometry of image formation in most modern imaging devices, ==useful images can hardly be captured by means of a pinhole camera==. 
- In particular, this model is not sufficent since _we dont have a lens_. 

### Perspective projection
The geometric model of image formation in a pinhole camera is known as __Perspective Projection__. 
![[projections.png]]
In addition:
- $m$ is also called a _projection_. 
- the _focal length_ defines the distance between the _image plane_ and the _optical center_ (and thus the focal plane). 

We want to find a relationship between 3D and 2D points. 

#### Coordinates in the image plane
Before being able to relate a point in the world to a point on the image plane, we need to define a coordinate system. 

![[planes.png]]
 The coordinates defined on the image plane are the __image reference system__, while the coordinates on the focal plane is the __camera reference system__ (it's called this way since it is somehow attached to my camera, but it is only mathematical obviously).  

- $x, y$ are the coordinates of the CRS, while u,v are IRS coordinates. 
- $(0,0,0)$ is the _pinhole coordinates_, $C$. 
- They are _parallel_: $u,x$ are horizontal, while $v,y$ are vertical. 
- The equations to map _scene points_ into their corresponding _image points_ are defined as follows:
![[fromulas1.png]]

What we can derive from this equation
![[disegnino_del_prof.png]]
![[formulas.png]]
As we know, in the pinhole camera model our image is inverted upside down, and obviously, we do not want to happen to our captured image. ==How do we get rid of the _up-down_ and _left-right_ inversions?== -> _Change of sign!_ 
- the image plane can be thought of as lying in front rather than behind the optical center (in real world we do not get flipped images)
![[removing_minus.png]]
In the picture above, the purple arrow represents the _image plane_ after "_flipping_", that is, <u>removing the minus</u>. 

The $z$ is the distance from the real element/object to the camera, and represents __depth__.
In addition, as we've just seen, it allows to _scale_ the coordinates from the _focal plane_ to the _image plane_.   

#### How do they scale?
==Image coordinates are a scaled version of scene coordinates (function of _depth_)==
- the $f$ can change if we move the image, like what happens when we zoom. 

The __Field of View__ (FOV) gets _bigger_ with a _short focal length_ and the rays are shorter, while if the focal length is larger (and so the FOV is restritcted) the images _are closer_ (and bigger!). 
- The farther the point the smaller the coordinates (object distant from the camera)
- The larger the _focal length_, the _bigger_ the object is in the image (and viceversa). 
![[fov_example.png]]
We scale the world inversely with respect to the depth ($z$) -> big depth equals small image.

A CV algorithm should be __scale-unbiased__ (scale-invariance?), meaning that is not affected by loss of information due to scaling. 

>[!WARNING]
> DO NOT CONFUSE SCALE WITH SIZE, or the professor will get angry. 

#### Loss of information

![[different_depths_and_points.png]]

The image formation process deals with mapping a 3D space onto a 2D space => _loss of information_.

The mapping is not a _bijection_: a given scene point is mapped into a unique image point 
- a given _image point_ is mapped onto a _3D line_ (i.e. the line through the point, $m$, and the optical centre, $C$).

You cannot reconstruct from a 2D image a 3D scene: recovering the 3D structure of a scene from a single image is an __ill-posed problem__ (the solution is not unique).
- For an image point we can only state that its corresponding scene point lays on a line, but cannot disambiguate a specific 3D point along such a line (i.e. we know nothing about the distance to the camera).

Can there be a solution to this?

### Stereo vision
The human vision is a stereo vision system. 
Solution: <u>use multiple images</u> (at least two) => stereo vision

By using stereo images, and somehow _mimicking the human vision system_, we can try and infer the _precise depth_ of a _point in the space_.
![[stereo_vision1.png]]

![[stereo_vision2.png]]
Nevertheless, this process only works if I can track the same point in the real world from two different cameras. Let $p_L$ and $p_R$ be the projection of the same on the left and right camera respectively:
- I need some sort of algorithm which can find __correspondences__ (between points), since _I dont know that $p_r$ and $p_l$ are the same point_. 
-  Given correspondences, 3D information can be recovered easily by __triangulation__ 
	- __triangulation__ is the process of determining the location of a point by _forming triangles_ to the point from known points

## Standard stereo geometry 
__Assumptions__: 
- Parallel (x, y, z) axes 
- Same focal length => _co-planar image planes_ 
- The two cameras are __perfectly aligned__ in _depth_ (this is _impossible_ in the real world). 

![[stereo_vision3.png]]

The transformation between the two reference frames is just a __translation__ ($b$), usually horizontal. $b$ is also called a _baseline_.
 
![[translation_baseline.png]]
$P_L$ and $P_R$ are two (correspondent) points on the left camera system and the right camera system respectively. 

There are a couple of issues though:
- No perfect alignment in depth in the real world. 
- You need to sense two images at the __very same moment__. 
- You can put the two cameras “as you want” but they must observe the same object. 

Let's try to sum-up mathematically and geometrically what we've seen. 

Consider that:
- $v_L = v_R$ are the same since the cameras are aligned.  
- $u_L$ and $u_R$ are different because of the baseline $b$. 
- $d = u_L - u_R$ defines the __disparity__. ==From disparity, we can derive depth==.   

![[fundamental_relationship_in_stereo_vision.png]]
The formula 
$$
z = \dfrac{b \cdot f}{d}
$$
is considered the ==__fundamental relationship in stereo vision__==. 

We can also define a disparity map. 