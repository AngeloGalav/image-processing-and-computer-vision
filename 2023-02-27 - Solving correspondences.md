---
TODO:
- fill the last part
---

## Standard stereo geometry - Part 2
Since we are given just two 2D images there is _no info about the correspondence_ between two points in the two images
- i.e., given a point $p_L$, I want to find $p_R$. 

Since we assume that the two cameras are perfectly aligned, we can ensure that the two points are effectively correspondant if they are aligned on the _vertical axis_. We can verify that using __stereo matching__: we draw _an horizontal line_ which _connects_ the _two points_.  
![[stereo_matching.png]]
To determine if two points are correspondent, we can examine the _color_ of each point and the _pattern_ of the neighbouring pixels (i.e. matching a block of pixels).  

It's very difficult to find patterns on uniform regions, but it is possible to it using a projector. 

### Epipolar Geometry
What if we the 2 points are not aligned anymore (meaning, that the _camera/projection plane are not aligned_)? Do we need to search through the whole image?
- We can _project __the line__ related to __point___ $P_L$ in the right plane and _search across that line_.
	- The search space of the stereo correspondence problem is always 1D! (since we are searching on the line)
![[epipolar_line.png]]

Issue: ==this projection can be computed only if the _transformation_ between the two cameras is known== (relative mapping between the two cameras). But:
- It is _almost impossible_ to build a stereo rig which is perfectly aligned horizontally
- Searching through oblique epipolar lines is awkward! 
- It is also computationally less efficient 
What can we do?

### Rectification
What people do in practice is to convert _epipolar geometry_ to _standard geometry_ (Rectification / Warping / __Homography__)
- Warp the images as if they were acquired through a standard geometry (horizontal and collinear conjugate epipolar lines) 
- Compute and _apply to both images_ a _transformation_ (i.e. homography) known as rectification. 
\[?? why both images? maybe to reduce artifacts??
\]

### Stereo Correspondence
Given a point in one image (e.g. L) find that in the other image (R) which is the projection of the same 3D point. Such image points are called corresponding points.

When we are looking for matches at bigger distances, we have smaller changes.  
![[farther_away_disparity.png]]
The blue point as a larger disparity in comparison to the yellow and red points. 

## Properties of Perspective Projection
As we've seen: 
- the farther objects are from the camera, the smaller they appear in the image. 
- The image of a 3D line segment of length $L$ lying in a plane parallel to the image plane at distance $z$ from the optical centre will exhibit a length given by:
$$
l = L \dfrac{f}{z}
$$
This relationship is _more complicated_ for an _arbitrarily oriented 3D segment_, as its _position_ and _orientation_ need to be accounted for as well. 
Nonetheless, for given position and orientation, length always shrinks alongside distance.


Perspective projection maps 3D lines into image lines 
If we have a line L and three points (A, B, C). 
When the line is no-longer parallel, these three points maybe at a different distance, and so the relation no longer holds. )

![[perspective_length_example_2.png]]
In these streets, since the road is _not parallel to the image plane_, the ratios are differents. 
These is due to perspective projection.  
Ratios of lengths are not preserved (unless the scene is planar and parallel to the image plane). • Parallelism between 3D lines is not preserved (except for lines parallel to the image plane)


In perspective projections, line on a plane that is not parallel to the image plane meet on a point called __vanishing point__.  
![[vanishing_point.png]]
- If the lines are parallel to the image plane they meet _at infinity_. 

- The vanishing point of a 3D line is the __image__ _of the point at infinity_ of the line 
- It can be determined by the ==intersection between _the image plane_ and _the line parallel_ to the given one== (and passing through the optical centre)

>[!WARNING]
>Do not forget that it's the _image_ of the point at infinity.


![[camera_model.png]]
- ==All parallel 3D lines will share the same vanishing point==, i.e. they “meet” at their vanishing point in the image (you only need one line to find the vanishing point).  
- If the 3D lines are parallel in the image plane they will not meet at infinity => they stay parallel 
- Every line has its own point at infinity. If the line is parallel to the image point, there's no vanishing point. 
	- 

### Weak perspective
- It may happen that all the lines are parallel (also) in the image…no (much) perspective distortion 
- In this case the image formation process can be approximated by a simpler model:
![[perspective_example_3.png]]

![[scaled_orthographic_projection.png]]

Let’s denote as $[z_0 - \Delta z, z_0 + \Delta z]$ the range of distances of the framed subject. If $\Delta z$ is small compared to $z_0$ we obtain:
![[formulas_1.png]]
- Which means that the perspective scaling factor $(f / z)$ is approximately constant $(f / z_0)$ for all points of the framed subject

- The above equations define a scaled orthographic projection : $u = sx, \ v =sy$. 
(ortographic projection + scaling). 

# Depth of Field (DOF)
As we've seen, from the pinhole camera model we missed lenses. 

- A scene point is on focus when all its light rays gathered by the camera hit the image plane at the same point 
- In a pinhole device this happens to all scene points because of the very small size of the hole, so that the camera features an _infinite_ __Depth of Field (DOF)__.
	- We have an infinite focus point -> Depth of Field is (plane of focus of the points)
![[pinhole_camera.png]]
In the pinhole model, since there are very little light rays passing through 
- if I enlarge the hole, the light is not coming from a ray but from a cone of light. So, the image will be very blurry.  
- Instead of enlarging the hole, we can take more rays overtime, but this will cause motion blur (if something moves). 

## Lenses
- Use lenses to gather more light from a scene point and focus it on a single image point.
	- The infine DOF is lost: the focus is preserved up to a limited range of distances. 
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
- $P$ : scene point 
- $p$ : corresponding focused image point 
- $u$ : distance from $P$ to the lens 
- $v$ : distance from $p$ to the lens 
- $f$ : focal length (parameter of the lens) 
- $C$ : centre of the lens 
- $F$ : focal point (or focus) of the lens

The image plane is supposed to be on the left of the lens (using the image above as reference). 

\[..\]
