## Standard stereo geometry - Part 2
Since we are given just two 2D images there is _no info about the correspondence_ between two points in the two images
- i.e., given a point $p_L$, I want to find $p_R$. 

Since we assume that the two cameras are perfectly aligned, we can ensure that the two points are effectively correspondent if they are aligned on the _vertical axis_. We can verify that using __stereo matching__: we draw _an horizontal line_ which _connects_ the _two points_.  
![[stereo_matching.png]]
To determine if two points are correspondent, we could examine the _color_ of each point and the _pattern_ of the neighboring pixels (i.e. matching a block of pixels).  

It's very difficult to find patterns on uniform regions, but it is possible to it using a projector. 

### Epipolar Geometry (_epipolar line_)
What if we need to find the correspondence between $p_R$ and $p_L$ and the 2 planes are _not aligned_ (meaning, that the _camera/projection plane are not aligned_)? Do we need to search through the whole image for the corresponding point? NO!
- We can _project __the line__ related to __point___ $p_L$ in the right plane and _search across that line_ (on the plane). This line is also called __epipolar line__. 
	- The _search space_ of the stereo correspondence problem _is always 1D_! (since we are searching on the line)
![[epipolar_line.png]]

Issue: ==this projection can be computed only if the _transformation_ between the two cameras is known== (relative mapping between the two cameras). But:
- It is _almost impossible_ to build a stereo rig which is _perfectly aligned horizontally_
- Searching through oblique epipolar lines is awkward! 
- It is also computationally _less efficient_ 
What can we do?

### Rectification
What people do in practice is to convert _epipolar geometry_ to _standard geometry_ (Rectification / Warping / __Homography__)
- Warp the images as if they were acquired through a standard geometry (horizontal and collinear conjugate epipolar lines) 
- Compute and _apply to both images_ a _transformation_ (i.e. homography) known as rectification. 
In this way, the transformation transforms the images as if the image planes were completely aligned. 

### Stereo Correspondence
Given a point in one image (e.g. L) find that in the other image (R) which is the projection of the same 3D point. Such image points are called __corresponding points__.

When we are looking for matches at bigger distances, we have smaller changes.  
![[farther_away_disparity.png]]
As we can see from the image, the blue point (which is nearer to the camera) as a larger disparity in comparison to the yellow and red points (which are further away from the camera). 

## Properties of Perspective Projection
As we've seen: 
- the farther objects are from the camera, the smaller they appear in the image. 
- The image of a 3D line segment of length $L$ lying in a _plane __parallel___ to the image plane at distance $z$ from the _optical centre_ will exhibit a length given by:
$$
l = L \dfrac{f}{z}
$$
This relationship is _more complicated_ for an _arbitrarily oriented 3D segment_, as its _position_ and _orientation_ need to be accounted for as well. 
Nonetheless, for given position and orientation, length always shrinks alongside distance.

Perspective projection maps 3D lines into image lines. 
- If we have a line L and three points (A, B, C). 
When the line is no-longer parallel, these three points maybe at a different distance, and so the relation no longer holds)

![[perspective_length_example_2.png]]
In these streets, since the road is _not parallel to the image plane_, the ratios are differents. 
These is due to perspective projection.  
Ratios of lengths are not preserved (unless the scene is planar and parallel to the image plane). 
- _Parallelism_ between 3D lines _is not preserved_ (except for lines parallel to the image plane)

In perspective projections, line on a plane that is not parallel to the image plane meet on a point called __vanishing point__.  
![[vanishing_point.png]]
- If the lines are parallel to the image plane they meet _at infinity_. 

Every line has its own point at infiniry. 
- The vanishing point of a 3D line is the __image__ _of the point at infinity_ of the line 
- It can be determined by the ==intersection between _the image plane_ and _the line parallel_ to the given one== (and passing through the optical centre)

>[!WARNING]
>Do not forget that it's the _image_ of the point at infinity.

![[camera_model.png]]
- ==All parallel 3D lines will share the same vanishing point==, i.e. they “meet” at their vanishing point in the image (you only need _one line_ to find the vanishing point).  
- If the 3D lines are parallel _in_ the image plane they will not meet at infinity => they stay parallel.

### Weak perspective
- It may happen that all the lines are parallel (also) in the image…no (much) perspective distortion 
- In this case the image formation process can be approximated by a simpler model:
![[perspective_example_3.png]]

![[scaled_orthographic_projection.png]]
Let’s denote as $[z_0 - \Delta z, z_0 + \Delta z]$ the _range of distances_ of the framed subject. If $\Delta z$ is small compared to $z_0$ we obtain:
![[formulas_1.png]]
- Which means that the perspective scaling factor $(f / z)$ is _approximately constant_ $(f / z_0)$ for _all points of the framed subject_.

- The above equations define a scaled orthographic projection : $u = sx, \ v =sy$. 
(ortographic projection + scaling). 

# Depth of Field (DOF)
As we've seen, from the pinhole camera model we missed lenses. 
- A scene point is __on focus__ when all light rays coming from the same point _hit the image plane at the same point_. 
- In a pinhole device this happens to all scene points because of the very small size of the hole, so that the camera features an _infinite_ __Depth of Field (DOF)__.
	- We have an infinite focus point -> Depth of Field is (plane of focus of the points)
![[pinhole_camera.png]]
In the pinhole model, we have a small quantity of light rays passing through, so the images are __very dark__.
- if I _enlarge the hole_, the light is not coming from a ray but from a cone of light. So, the image will be _very blurry_ (out of focus).  
	- Instead of enlarging the hole, we can take _more rays overtime_ (long exposure times), but this will cause _motion blur_ (if something moves). 

## Lenses
- Use lenses to gather more light from a scene point and focus it on a single image point.
	- The infine DOF is lost: _the focus is preserved up to a __limited range__ of distances_. 
- This enables much smaller exposure times => avoid motion blur in dynamic scenes

We will consider the approximate model known as thin lens equation: 
$$
	\dfrac{1}{u} + \dfrac{1}{v} = \dfrac{1}{f}
$$
>[!warning]
>The variables $u, v, f$ are not the same that we've seen so far, they represent different measures. 

To graphically determine the position of a focused image point we can leverage on two properties of thin lenses:
1. Rays parallel to the optical axis are deflected to pass through $F$
2. Rays through $C$ are undeflected
![[lenses.png]]

Here's what each variable represents:
- $P$ : scene point.
- $p$ : corresponding focused _image point_.
- $u$ : distance from $P$ to the lens. 
- $v$ : distance from $p$ to the lens.
- $f$ : focal length (parameter of the lens) 
- $C$ : centre of the lens 
- $F$ : focal point (or focus) of the lens

The image plane is supposed to be on the left of the lens (using the image above as reference). 

#### Circles of confusion
- On one hand: choosing the distance of the image plane determines the distance at which scene points appear on focus in the image 
- On the other hand: to acquire scene points at a certain distance we must set the position of the image plane accordingly: 
- Given the chosen position of the image plane, scene points both _in front and behind the focusing plane will result __out-of-focus___, thereby appearing in the image as circles, known as __Circles of Confusion__ or Blur Circles, rather than points 
- The advantage of lenses is to have a _small exposure time_ for capturing moving objects but we pay in terms of depth of field.
![[circles_of_confusion.png]]