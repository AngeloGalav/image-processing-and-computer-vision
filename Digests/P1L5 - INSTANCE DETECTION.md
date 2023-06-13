# Instance detection
Given an image of a specific object, determine whether the object is present or not in the image. 
We would like to train a model using a single image in this case. 
Also, we just want to detect a _single object_ that _may appear once_ in the _target image_.

Typical nuisances are _intensity changes_, _occlusions_ and _clutter_.

## Template matching
We have a _template_ (or model image) and a _target image_.
The _model image_ is slid across the target image to be compared at each position to an equally sized window by means of a suitable _similarity or dissimilarity function_.

## Dissimilarity functions
![[ssd_formula.png]]
SSD is not light invariant. 


A similarity function that goes to [-1, +1] is:
![[NCC.png]]
- So, _only the linear component_ $\tilde I(i,j) = \alpha \cdot T$ is invariant.

NCC is also slower. 

#### ZNCC
This one is invariant to _affine intensity changes_ (not just linear)
![[zncc.png]]

## Fast template matching
Since template matching can be slow, a popular approach is to deploy a pyramid. 
- It consists of full searches on the top level, with local refinements traversing back the pyramid. 
Very fast approach, though the _number of levels_ needs to be _chosen carefully_ (and empirically) to avoid loss of information, which negatively affect the detection.

## Shape based/Edge based matching
A set of _control points_ $P_k$ is extracted from the model image by an Edge detector, and the _gradient_ at each $P_k$ is stored. 
- Then, at each position $(i,j)$ of the target image, the _recorded gradient directions_ associated with _control points_ are _compared_ to those at their _corresponding image points_, $\tilde P_k(i,j)$ 

- The more the red arrows are aligned to the black arrows the _more similar_ the sub-image is to the template.

## Similarity function
![[unit_vector.png]]

The resulting similarity function is:
![[similarity_function.png]]

If the angle between them is 0, then they are similar!

##### More robust
To achieve the _invariance_ to __global inversion of contrast polarity__ along object’s _contours_, we simply add a modulo to the $S(i,j)$ function. 
To achieve local invariance to inversion of polarity, we simply add modulo to the inside of the sum only. 

## Hugh Transform
- It's object detection for shape with a known equation. Input data is projected onto a space called __Hugh space__. 
- Turns global detection problem into a local one (hugh space instead of global space)
- HT is applied _after edge detection_ (input data comes from edge detection).
- HT is _robust to noise_ and recognizes even partially occluded space.

### HT for lines
n the usual image space interpretation of the line equation, the _parameters_ $(\hat m, \hat c)$ are _fixed_ so that the equation $y − \hat mx − \hat c = 0$ represents the mapping from _point_ $(\hat m, \hat c)$ of the _parameter space_ to the _image points_ belonging to the _line_. 

However we may _instead fix_ $(\hat x, \hat y)$, so that we get $\hat y − m\hat x −  c = 0$, so that the equation represents the mapping from image point $(\hat x, \hat y)$ to the _parameter space_ providing all the __lines__ through the _image point_. 

To put it simply, if we have $n$ points aligned in the same line in the _image space_, in the _parameter space_ we have a single point (representing the line) intersecated by $n$ lines (representing the points). 

Rather than looking at an extended shape into the image, we look for a _specific feature_ (__where lines intersect__) in the _parameter space_ of lines.

### Accumulator Array
the HT consists in ___mapping__ image points_ so as to _create curves_ into the _parameter space_ of the _shape_.
-  Intersections in the parameter space indicate the presence of image points of instance of the target shape. 
	- Detection by HT is based on this: searching points in the params space through which many curves intersect. 

To make this work, the parameter space needs to be _quantized_, and allocated as a _memory array_, called the __Accumulator Array__.

- Curves are drawn into this space with the __voting process__:
1. The transform equation is _repeatedly computed_ to increasingly satisfying the equation. 
2. A _high number of intersecting_ curves means a _high number of votes_ into the bin.
3. we find the local maxima, which shows us a high number of votes. 

#### Final remarks 
The usual line parametrization considered so far ( i.e. $y-mx-c=0$ ) is _impractical_ due to $m$ spanning an _infinite range_, so a "_normal parametrization_" is used:
$p = xcos\theta + ysin\theta$