# Instance Detection
The __Instance-level Object Detection__ problem occurs in countless applications and can be formulated as follows:
- Given a _reference image_ (aka _model image_) of a specific object, determine whether the object _is present or not_ in the _image under analysis_ (aka target image) and, in case of detection, estimate the pose of the object.
![[instance_detection_example.png]]

Depending on the application, the pose may often be given by a translation, a roto-translation or a similarity (roto-translation plus scale).

Usually, we train a model with a lot of images in order to detect patterns/instances, but we would like to train this net using a _single image_. 

The problem has a number of diverse facets:  the sought object may appear only once or multiple times in the target image, we may be interested in detecting a number of diverse objects, each of which, in turn, may appear once of multiple times.
For the sake of simplicity, and without much loss of generality, we will refer mainly to the basic setting: detecting a _single object_ that _may appear once_ in the _target image_. 
Generalization to the other variants turns out more often than not straightforward. 

Typical nuisances to be dealt with are _intensity changes_, _occlusions_ and _clutter_. 
Computational efficiency is a major requirement in most practical applications. 

This problem is characterized by a _limited variability_ as the assumption deals with the appearance of the object being captured by a _single model image_ (i.e. roughly planar objects or no view-point changes) and the pose is typically either a 2D translation or a 2D roto-translation or a similarity. 
Given the limited variability, the problem can been addressed successfully by _classical computer vision_ techniques, the major applications dealing mainly with the space of industrial vision. 
Instead, Category-level Object Detection aims at detecting certain kind of object(s) (e.g. cars, pedestrians..) regardless of their appearance and pose. Due to the high-variability, this problem is addressed by machine/deep learning techniques

## Template Matching
We have a _template_ (or model image) and a _target image_.
The _model image_ is slid across the target image to be compared at each position to an equally sized window by means of a suitable _similarity or dissimilarity function_:
- We slide our template over the image at each possible candidate position and in each candidtate position I compute a _similarity or dissimilarity function_. 
![[Images/template_matching.png]]

## Dissimilarity and Similarity functions
![[dis_sim_fun.png]]

\[N.B.: we dont do padding in this case obv.\]

Everytime you compute a dissimilarity function between a _template_ and _sub-image_ of the whole image.
Both $\tilde I(i,j)$, the window at position $(i, j)$ of the target image having the same size as $T$, as well as $T$ can be thought of as $M·N$-dimensional vectors (flatten).
- Compute pixel-wise intensities differences using _SSD_.

#### SAD and SSD
We can use __SSD (Sum of Squared Differences)__ as a simple _dissimilarity function_, in which we are basically computing and then summing the _pixel-wide difference_ w.r.t. each pixel:
![[ssd_formula.png]]

Another function is the of __Sum of Absolute Differences__:
![[sad_difference.png]]
Are SSD and SAD invariant to intensity changes? No. 
So instead, we may use NCC.

#### NCC 
[An issue we may encounter is light intensity changes, so we may use _affine intensity changes_. (not sure if this is true though)]

For this reason, we should use the __Normalised Cross-Correlation__:
![[NCC.png]]
But, it actually represents the _cosine of the angle between vectors_ $I(i,j)$ e $T$, so it goes $[-1,+1]$. 
![[NCC_2.png]]
NCC is considered a _similarity function_, since $NCC=0$ when both vectors are the same. 
The NCC is not completely invariant though, since in the formula $I = \alpha I + \beta$, summing $\beta$ changes the direction of the vector. 
- So, _only the linear component_ $\tilde I(i,j) = \alpha \cdot T$ is invariant. (invariacne to linear intensity chagnes)

NCC is also _slower_ since we need to compute a multiplication in the numerator and two operation on the denominator. 

#### Zero-Mean Normalised Cross-Correlation, Correlation Coefficient
$$
\micro(\tilde I) =  \dfrac{1}{MN} \sum^{M-1}_{m=0} \sum^{M-1}_{n=0} I(i+m, j+n)
$$
$$
\micro(T) =  \dfrac{1}{MN} \sum^{M-1}_{m=0} \sum^{M-1}_{n=0} T(m,n)
$$
We subtract the mean and compute the ZNCC:
![[zncc.png]]
This is invariant to _affine intensity variation_: $\tilde I(i,j) = \alpha \cdot T + \beta$

### Comparison of ZCC under significant intensity changes
![[comparison.png]]
ZNCC turns out to be a similarity function very robust to intensity changes!


## Fast Template Matching
Template matching may be exceedingly slow whenever the model and/or target images have a large size (i.e. computational complexity is $O(M×N×W×H)$). 

A popular approach is to deploy an _image pyramid_: 
- Smoothing and sub-sampling 
	- Typically 1/2 on both sides at each level 
	- Shrink the image, shrink the template 
- _Full search at top level_ and then _local refinements_ traversing back the pyramid down to the original image
![[pyramid.png]]
Very fast approach, though the _number of levels_ needs to be _chosen carefully_ (and empirically) to avoid loss of information, which negatively affect the detection.

## Shape-based Matching
Another istance level object detection that we've seen is __SIFT__, but you cannot detect edges or corners in SIFT.

__Edge-based template matching approach__ 
We essentially use edges to define a shape, and the edges are then use to detect the shape in the target image. 
From edges, we can exploit the _orientation_, and add the _orientation information_ to the control points.  

- First, a set of _control points_, $P_k$, is extracted from the _model image_ by an _Edge Detector_ and the _gradient direction_ at each $P_k$ is stored.
	- The Template composed by _offsets_ and _gradient directions_ 
	- We don't use the magnitude, since it is very much influenced by the intensity difference. 

- Then, at each position $(i,j)$ of the target image, the _recorded gradient directions_ associated with _control points_ are _compared_ to those at their _corresponding image points_, $\tilde P_k(i,j)$ 
- ==The more the red arrows are aligned to the black arrows the _more similar_ the sub-image is to the template== 
- ==We do not perform edge detection on the target image, just on the template== \[why?\].
![[shape_based.png]]

### Similarity Function
![[unit_vector.png]]
The resulting similarity function is:
![[similarity_function.png]]

The similarity function spans the interval \[-1; 1\].
It takes its _maximum value_ when all the gradients at the _control points_ in the current window of the _target image_ are _perfectly aligned_ to those at the control points of the model image 
- If they are perfeclty aligned the angle is zero, the cosine is 1, the sum is n, which divided by n is 1, and vice versa. 
- Choosing a __detection threshold__, $S_{min}$, can be thought of as specifying the fraction of model points which must be seen in the image to _trigger a detection_.

- The edge is positive when we have high polarity (from _bright_ to _dark_).
![[angle.png]]

### More robust modifications
Certain application settings call for _invariance_ to __global inversion of contrast polarity__ along object’s _contours_, as the object may appear either darker or brighter than the background in the target image.

This kind of invariance can be achieved by a slight modification to the similarity function defined previously (global) 
![[similarity_2.png]]
The following function is even more robust due to the ability to withstand __local contrast polarity inversions__ (local)
![[similarity_3.png]]

## The Hough Transform
The __Hough Transform (HT)__ enables to _detect objects_ having a known _shape_ that can be _expressed_ by _an equation_ (e.g. lines, circles, ellipses..) based on _projection_ of the input data into a suitable space referred to as __parameter or Hough space__ (different from the image space) 
- The HT turns a _global detection_ problem into a _local_ one (i.e., look for feature points into the parameter space, instead of looking for the whole shape in the image space) 
- The HT is usually applied _after an edge detection_ process (i.e., the actual input data consist of the edge pixels extracted from the original image) 
- The HT is _robust to noise_ and allows for detecting the sought shape even though it is _partially occluded_ into the image (up to a certain user-selectable degree of occlusion) 

The HT was invented to detect lines and later extended to other analytical shapes (circle, ellipses) as well as to arbitrary shapes => Generalized Hough Transform (GHT)

#### HT for lines
In the usual image space interpretation of the line equation, the _parameters_ $(\hat m, \hat c)$ are _fixed_ so that the equation $y − \hat mx − \hat c = 0$ represents the mapping from _point_ $(\hat m, \hat c)$ of the _parameter space_ to the _image points_ belonging to the _line_. 

However we may _instead fix_ $(\hat x, \hat y)$, so that we get $\hat y − m\hat x −  c = 0$, so that the equation represents the mapping from image point $(\hat x, \hat y)$ to the _parameter space_ providing all the __lines__ through the _image point_. 

Consider two image points $P_1,P_2$ and map both into the parameter space, we get two lines intersecting at the parameter space point representing the image line through $P_1,P_2$:
![[basic_principle.png]]
More generally, if we map $n$ image points we get as many intersections as $n(n-1)/2$ (i.e. the number of lines through the $n$ image points).

Considering $n$ _collinear image points_, we can notice that their corresponding transforms (i.e. parameter space lines) will _intersect_ at a ___single__ parameter space point_ representing the image _line_ along which such $n$ points lay.
![[HT.png]]
Rather than looking at an extended shape into the image, we look for a _specific feature_ (__where lines intersect__) in the _parameter space_ of lines.

### Accumulator Array
Therefore, given a sought analytic shape represented by a set of parameters, the HT consists in ___mapping__ image points_ (i.e. usually edge points) so as to _create curves_ into the _parameter space_ of the _shape_.

_Intersections_ of parameter space curves indicate the presence of _image points_ related to an _instance_ of the desired shape:
- the more the _intersecting curves_ the more are such image points and thus the higher is the evidence of the _presence of that instance_ in the image. 

==_Detecting objects_ through the HT consists in _finding parameter space_ points through which many curves do intersect== (a local rather than global detection problem).

To make it work in practice, the parameter space needs to be _quantized_ and allocated as a memory array, which is often refereed to as __Accumulator Array (AA)__ 
- Curves are “drawn” into the AA by a so called ___voting process___: 
	1. the _transform equation_ is repeatedly computed to increment the bins _satisfying the equation_. 
	2. a _high number of intersecting curves_ at a point of the parameter space will provide a _high number of votes_ into a bin of the AA.
	3. Finding parameter space points through which many curves do intersect is thus implemented in practice by finding _peaks_ of the AA, i.e. local maxima showing a high number of votes.

![[AA.png]]
To detect the line _more accurately_, the AA should be _quantized more finely_.

The HT is __robust to noise__ because spurious votes due to noise _unlikely accumulate into a bin_ so as to _trigger_ a _false detection_.
A partially occluded object can be detected provided that the _threshold on the minimum number_ of votes required to declare a detection _is lowered_ according to the degree of occlusion to be handled.

##### Final remarks on HT for Line Detection
The usual line parametrization considered so far ( i.e. $y-mx-c=0$ ) is _impractical_ due to $m$ spanning an _infinite range_, so...:
![[final_HT.png]]