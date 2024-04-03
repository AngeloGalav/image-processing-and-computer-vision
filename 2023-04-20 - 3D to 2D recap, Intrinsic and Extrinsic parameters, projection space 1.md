## Image formation (recap)
Images are _2D perspective projections_ of the 3D world.
Perspective projection model: 
- given a point in the 3D space, $𝑀 = [𝑥, 𝑦, 𝑧]^𝑇$ , whose coordinates are given in the __Camera Reference Frame (CRF)__, 
- its _perspective projection_ onto the _image plane_ $I$, denoted as $𝑚 = [𝑢, 𝑣]^𝑇$ is _given by the non-linear equations_ (1)
![[formula.png]] 
- The __Camera Reference Frame (CRF)__ is a 3D reference frame for which: 
	- the origin is the __optical center__ $𝐶$. 
	- $𝑥, 𝑦$ axis are parallel to $𝑢$ and $𝑣$ (the image _horizontal_ and _vertical_ axis) 
	- $𝑧$ axis is the __optical axis__.
![[crf.png]]

To create a more realistic camera model we need to consider three additional issues: 
1. Image coordinates are usually expressed in an _image reference frame_ with the _origin in the __top left corner___. 
2. Images are _grid of pixels_, _not a continuous plane_ (aka pixelization) 
3. We do not normally know or want to project back the coordinate of a 3D point in the Camera Reference Frame, but in some generic __World Reference Frame__.
![[realistic_image.png]]

## Complete Forward Imaging model: 3D to 2D
![[complete_image_model.png]]
- We define a WRF. Then, we convert WRF coordinates into CRF coordinates using a simple convolution. 
- After that, we obtain our 2D point coordinates by using a perspective projection.  
- __Extrinsic parameters__: position of the camera, rotation etc... Stuff that is in respect to the WRF.
	- Independent from the camera that I have!
- __Intrisic parameters__: optical center, focal length... 

## From CRF to 2D point coords: Image Pixelization
Digitization can be accounted for by including into the _projection equations_ the __pixel size__ $𝚫𝒖$ and $𝚫𝒗$ along the two axis (typically they have the same value, i.e. pixels are squared, but in the general model we let them be different) 
- We have a _discrete grid of pixels_ (not a continuous plane)
![[image_pixelization.png]]

#### Origin of the image reference frame
We also need to model the _translation_ of the __piercing point__ (intersection between the _optical axis and the image plane_) w.r.t. the origin of the image coordinate system (top-left corner of the image) 
- We do not use _negative pixel_ indices when accessing images
![[piercing_point.png]]

#### Final, intrinsic parameters
The _final relationship_ between 3D coordinates in the __Camera Reference Frame__ and its __corresponding pixel__ in an image is:
![[final_formula.png]]
we can also write $\dfrac{f}{\Delta u}$ as $f_u$, and substituting it in the formula. $f_u$ is the _focal length measured in horizontal pixels_.  
The total number of intrinsic parameters is then __4__: 
- $𝑓_𝑢, f_v$ and the coordinates of the _piercing point_ $𝑐 (𝑢_0, 𝑣_0)$.
They represent the _camera geometry_, which is _independent of its position in the world_: regardless of where I put my camera, these values will be the same.

## From WRF to CRF
3D coordinates are measured into a __World Reference Frame (WRF)__ external to the camera, related to the CRF by a Roto-Translation: 
- A _rotation_ around the _optical centre_ (e.g. expressed by a 3x3 rotation matrix $𝑹$). 
- A _translation_ (expressed by a 3x1 translation vector $𝒕$). 
(what we do is essentially a convolution). 
Therefore, the relation between the coordinates of a point in the two RFs is:
![[convo_wrf.png]]
Where:
- $M_C$ is the point in CRF coordinates.
- $R$ is the rotation matrix.
- $M_W$ is the point in WRF coordinates. 
- $t$ is the translation vector.

#### Rotation matrix
Any valid rotation matrix is an _orthonormal matrix_, i.e. its rows and columns are _orthonormal_.
Any two vectors $𝒂$ and $𝒃$ are __orthonormal__ if and only if
![[orthonormal.png]]
For an orthonormal matrix $𝑹$ it is then true that: $RR^T = R^TR=I$. (i.e. its inverse is its transpose matrix). 
What are the coordinates $𝑪_𝑾$ of the _optical centre_ $𝑪$ in the _WRF_?
$𝟎 = 𝑹𝑪_𝑾 + 𝒕 ⇒ 𝑹𝑪_𝑾 = −𝒕 ⇒ 𝑪_𝑾 = −𝑹^Tt$
So, the final coordinate formula is:
$$
C_W = -R^Tt
$$
#### Number of extrinsic parameters
- The rotation matrix has 9 entries but it has only _3 independent parameters_ (Degrees of Freedom), which correspond _to the rotation angles around the axes of the Reference Frame_.
- $t$ has 3 independent entries.
- => The total number of __extrinsic parameters__ is then __6__ (3 translation parameters, 3 rotation parameters). They encode the _position of the camera w.r.t. the WRF_.

## Can we just combine the equations we have?
![[combiniting_the_equations.png]]
Can we make it linear? 🫠

## Projective Space
The standard 2D Euclidean plane can be represented as $ℝ^𝟐$ , i.e. by 2D vectors in a given reference frame. In this space:
- _parallel lines_ do _not intersect_ 
	- ...We used to say that parallel lines intersect at infinity.
- _points at infinity cannot be represented_.
	- ...The vanishing point meet at infinity in our case. 

Let’s now _append one more coordinate_ to our Euclidean pairs, so that:
![[projectvie_space1.png]]
and assume that both vectors are _equivalent representations_ of the same _2D point_.
Moreover, we _do not constrain the 3rd coordinate to be 1_ but instead let:
![[projective_space2.png]]

In this representation a point in the plane is represented by an _equivalence class of triplets_, wherein _equivalent triplets differ just by a multiplicative factor_. 

These are referred to as the __homogeneous coordinates__ (a.k.a. __projective coordinates__) of the 2D point having Euclidean coordinates $[𝑢, 𝑣]^𝑇$. 
The _space_ associated with the homogeneous representation is called __Projective Space__, denoted as $ℙ^𝟐$ .

Extensions to Euclidean spaces of any other dimension is straightforward $(ℝ^𝒏 → ℙ^𝒏)$, e.g. for the 3D Euclidean space $ℝ^𝟑$ we can convert a point $𝑀_𝐶 ∈ ℝ^𝟑$ into projective coordinates $\tilde 𝑀_𝐶 ∈ ℙ^𝟑$ by _adding a 4th coordinate_:
![[center_projective_space.png]]

Here's a graphical representation of this space:
![[projective_space.png]]

## Point at infinity of a 2D line
![[point_infty_2d.png]]
![[line_infty.png]]
The point $[0,0,0]$ is forbidden in this space: it does not belong in it!
- The 3rd coordinate is k $\not =$ 0. 

## Point at infinity of a 3D line
Same thing as 2D line, but with an added coordinate:
![[3dline_infty.png]]

## Points at infinity ≠ vanishing point
__The point at infinity is a point in space, the vanishing point is a point onto the image plane__. 
However, there is very clear relationship between the two: the vanishing point for a set of 3D parallel lines is the projection of their point at infinity.

Point at infinity cannot be represented in the 3D Euclidean Space: to map such points into the Euclidean Space we would divide by the fourth coordinate $(x/0, y/0, z/0)$, which is not a valid representation in the Euclidean Space.

With homogenous coordinates it is therefore possible to represent and process both ordinary points as well as points at infinity.
Point $(0, 0, 0, 0)$ is _not_ part of $ℙ^𝟑$. _It is NOT the origin of the Euclidean Space_ $(0, 0, 0)$.
- such point is represented in homogeneous coordinates as $[0, 0, 0, k]$ with $k≠0$.

## Perspective Projection in homogeneous coordinates
Let’s go back to the non-linear transformations between 3D coordinates in the CRF and image coordinates. 
Given a 3D point $𝑀_𝐶 = [𝑥_𝐶, 𝑦_𝐶, 𝑧_𝐶]^T$ in the CRF and its projection onto the image plane $𝑚 = [𝑢, 𝑣]^𝑇$ , what happens if we express this operation between projective spaces instead of Euclidean ones? 
If they are expressed in homogenous coordinates (denoted by symbol ~) then the equations can be written as _matrix multiplication_, i.e. in linear form:
![[projection_space.png]]
In homogeneous coordinates the perspective projection becomes a _linear transformation_:
![[linear_perspective.png]]

>[!WARNING]
>Remember!! $k \not = 0$!!!!

----
