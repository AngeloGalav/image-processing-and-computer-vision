---
TODO:
- risistemare diverse facts
- ristemare parte su ZCC
- rivedere proprio la parte su shape based matching
- risistemare hugh transform
---


The __Instance-level Object Detection__ problem occurs in countless applications and can be formulated as follows:
- Given a _reference image_ (aka _model image_) of a specific object, determine whether the object _is present or not_ in the _image under analysis_ (aka target image) and, in case of detection, estimate the pose of the object.
![[instance_detection_example.png]]

Depending on the application, the pose may often be given by a translation, a roto-translation or a similarity (roto-translation plus scale).

Usually, we train a model with a lot of images in order to detect patterns/instances, but we would like to train this net using a single image. 


The problem has a number of diverse facets: 
- the sought object may appear only once or multiple times in the target image, we may be interested in detecting a number of diverse objects, each of which, in turn, may appear once of multiple times. • For the sake of simplicity, and without much loss of generality, we will refer mainly to the basic setting: detecting a single object that may appear once in the target image. Generalization to the other variants turns out more often than not straightforward. • Typical nuisances to be dealt with are intensity changes, occlusions and clutter. • Computational efficiency is a major requirement in most practical applications. • This problem is characterized by a limited variability as the assumption deals with the appearance of the object being captured by a single model image (i.e. roughly planar objects or no view-point changes) and the pose is typically either a 2D translation or a 2D roto-translation or a similarity. • Given the limited variability, the problem can been addressed successfully by classical computer vision techniques, the major applications dealing mainly with the space of industrial vision. • Instead, Category-level Object Detection aims at detecting certain kind of object(s) (e.g. cars, pedestrians..) regardless of their appearance and pose. Due to the high-variability, this problem is addressed by machine/deep learning techniques

## Template Matching
We have a _template_ (or model image) and a _target image_.
The _model image_ is slid across the target image to be compared at each position to an equally sized window by means of a suitable _similarity or dissimilarity function_:
- We slide our template over the image at each possible candidate position and in each candidtate position I compute a _similarity or dissimilarity function_. 
![[template_matching.png]]

#### Dissimilarity and Similarity functions
![[dis_sim_fun.png]]

\[N.B.: we dont do padding in this case obv.\]

Everytime you compute a dissimilarity function between a _template_ and _sub-image_ of the whole image,

#### SAD and SSD
We can use __SSD (Sum of Squared Differences)__ as a simple _dissimilarity function_, in which we are basically computing and then summing the _pixel-wide difference_ w.r.t. each pixel:
![[ssd_formula.png]]

Another function is the of __Sum of Absolute Differences__:
![[sad_difference.png]]

#### NCC 
An issue we may encounter is light intensity changes, so we may use _affine intensity changes_.
Are SSD and SAD invariant to intensity changes? NO, when should we use them[???????]
For this reason, we should use the __Normalised Cross-Correlation__:
![[NCC.png]]
But, it actually represents the cosine of the angle between vectors $I(i,j)$ e $T$, so it goes $[-1,+1]$. 
![[NCC_2.png]]

The NCC is not completely invariant though, since in the formula $I = \alpha I + \beta$, summing $\beta$ changes the direction of the vector. 
- So, only the linear component $\tilde I(i,j) = \alpha \cdot T$ is invariant.

NCC is also slower since we need to compute a multiplication in the numerator and two operation on the denominator. 

#### Zero-Mean Normalised Cross-Correlation, Correlation Coefficient
$$
\micro(\tilde I) =  \dfrac{1}{MN} \sum^{M-1}_{m=0} \sum^{M-1}_{n=0} I(i+m, j+n)
$$
$$
\micro(T) =  \dfrac{1}{MN} \sum^{M-1}_{m=0} \sum^{M-1}_{n=0} T(m,n)
$$

This is invariant to affine intensity variation


### Comparison of ZCC under significant intensity changes
![[comparison.png]]
ZNCC turns out to be a similarity function very robust to intensity changes!


## Fast Template Matching
Template matching may be exceedingly slow whenever the model and/or target images have a large size (i.e. computational complexity is $O(M×N×W×H)$). 

A popular approach is to deploy an _image pyramid_: 
- Smoothing and sub-sampling 
- Typically 1/2 on both sides at each level 
- Shrink the image, shrink the template 
- Full search at top level and then local refinements traversing back the pyramid down to the original image
![[pyramid.png]]
Very fast approach, though the _number of levels_ needs to be _chosen carefully_ (and empirically) to avoid loss of information, which negatively affect the detection.

## Shape-based Matching
Another istance level object detection that we've seen is __SIFT__, but you cannot detect edges or corners in SIFT.

__Edge-based template matching approach__ 
We essentially use edges to define a shape, and the edges are then use to detect the shape in the target image. 
From edges, we can exploit the _orientation_, and add the _orientation information_ to the control points.  

- First, a set of _control points_, $P_k$, is extracted from the model image by an _Edge Detector_ and the _gradient direction_ at each $P_k$ is stored.
	- Template composed by offsets and gradient directions 
	- We don't use the magnitude, since it is very much influenced by the intensity difference. 

- Then, at each position $(i,j)$ of the target image, the _recorded gradient directions_ associated with _control points_ are _compared_ to those at their _corresponding image points_, $\tilde P_k(i,j)$ 
- ==The more the red arrows are aligned to the black arrows the more similar the sub-image is to the template== 
- ==We do not perform edge detection on the target image, just on the template== \[why?\].
![[shape_based.png]]
Another istance level object detection that we've seen is __SIFT__, you cannot edges or corners in SIFT, 


## Similarity Function
![[unit_vector.png]]
The resulting similarity function is:
![[similarity_function.png]]

The similarity function spans the interval \[-1; 1\].
- The edge is positive when we have high polarity (from bright to dark).
![[angle.png]]

It takes its maximum value when all the gradients at the control points in the current window of the target image are perfectly aligned to those at the control points of the model image • If they are perfeclty aligned the angle is zero, the cosine is 1, the sum is n, which divided by n is 1, and vice versa • Choosing a detection threshold, Smin, can be thought of as specifying the fraction of model points which must be seen in the image to trigger a detection

### More robust modifications
Certain application settings call for invariance to global inversion of contrast polarity along object’s contours, as the object may appear either darker or brighter than the background in the target image 

This kind of invariance can be achieved by a slight modification to the similarity function defined previously (global) 
![[similarity_2.png]]
The following function is even more robust due to the ability to withstand local contrast polarity inversions (local)
![[similarity_3.png]]

## The Hough Transform
The __Hough Transform (HT)__ enables to _detect objects_ having a known shape that can be expressed by _an equation_ (e.g. lines, circles, ellipses..) based on _projection of the input data_ into a _suitable space_ referred to as __parameter or Hough space__ (different from the image space) 
- The HT turns a _global detection_ problem into a _local_ one (i.e., look for feature points into the parameter space, instead of looking for the whole shape in the image space) • The HT is usually applied after an edge detection process (i.e., the actual input data consist of the edge pixels extracted from the original image) • The HT is robust to noise and allows for detecting the sought shape even though it is partially occluded into the image (up to a certain user-selectable degree of occlusion) • The HT was invented to detect lines and later extended to other analytical shapes (circle, ellipses) as well as to arbitrary shapes => Generalized Hough Transform (GHT) • The GHT principle is widely deployed also within object detection pipelines relying on local invariant features such as, e.g., SIFT

HT formulation for lines, what is fixed and what is changing in this equation? • In the usual image space interpretation of the line equation the parameters (�", �̂) are fixed so that the equation $y − \hat mx − \hat c = 0$ represents the mapping from point $(\hat m, \hat c)$ of the _parameter space to the image points belonging to the line_. 

Howwver we may _instead fix_ $(\hat x, \hat y)$, so that we get $\hat y − m\hat x −  c = 0$, so that the equeation represents the mapping from image point $(\hat x, \hat y)$ to the parameter space providing all the lines through the image point. 

Consider two image points $P_1,P_2$ and map both into the parameter space, we get two lines intersecting at the parameter space point representing the image line through $P_1,P_2$:
![[basic_principle.png]]
More generally, if we map $n$ image points we get as many intersections as $n(n-1)/2$ (i.e. the number of lines through the $n$ image points).
