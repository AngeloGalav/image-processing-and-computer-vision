# Edge detection
Edge or contour points are local features of the image that capture important information related to its semantic content. 
Edges are exploited in countless image analysis tasks (e.g. foreground-background segmentation, template matching, stereo matching, visual tracking).
In machine vision, edge detection is key to measurement tools.
- _Edges_ are pixels that can be thought of as lying exactly in _between_ _image regions_ of _different intensity_, or, in other words, to separate different image regions
![[edges 1.png]]

- _Sharp_ change of a 1D signal (1D step-edge)
	- In the transition region between the two constant levels the absolute value of the first derivative gets high (and reaches an _extremum_ at the _inflection point_)
	- The simplest edge-detection operator relies on thresholding the absolute value of the derivative of the signal
		- _Derivatives_ are the best tools so far to _compute edges_. 

![[inflection_point.png]]![[inflection_point_2.png]]

We use the absolute value since if we go from high intensity to low intensity, the derivative is negative, but the edge is still there lool. 

## Edge detection
A 2D Step-Edge is characterized not only by its strength but also its _direction_:
![[edge_detection.png]]

- An operator which allow us to sense the edge whatever its direction is the _gradient_:
![[gradient_edge.png]]
- To determine how steep a difference is, we can use the _norm_ of the gradient. 
	- Gradient’s direction gives the direction along which the function exhibits its maximum variation 
- A generic directional derivative can be computed by the dot product between the gradient and the unit vector along the direction. 

## Edge Detection by Gradient Thresholding
![[edge_detection_by_gradient_thresh.png]]
The magnitude can give us the orientation of the edge. 

## Discrete approximation of the Gradient
To _discretize derivate_ image __partial derivatives__, we can exploit the use of _differences between intensities_. 
We can use both _backward_ an _forward_ differences:
![[discrete_approximation.png]]

Or _central_ differences:
![[central_differences.png]]
![[approximation_of_the_gradient_points.png]]

We can estimate the _magnitude_ using different approximations:
![[different_approxiamtion.png]]
The __max-norm__ (which can be seen as an approximation of the infinite norm)
![[edge_discrete_approximation.png]]
- _The max-norm is both the fastest and the best method_, wrt to edge direction. 
	- more isotropic -> more chance to find edges. 
	- (?? more false positives?)

## Noise in Edges
In real images an edge _will not look as smooth as we have seen_ (due to noise).
Taking derivatives of noisy signals is an ill posed problem…the solution is not robust wrt to input variations => __Derivatives amplify noise__ 
- And we cannot apply the max norm anymore, otherwise we'll be filled with _false edges_. 

To work with real images, an edge detector should therefore be _robust to noise_, so as to highlight the meaningful edges only and filter out effectively the spurious transitions caused by noise
- To be robust to noise, we can _smooth_ the _signal_ before computing the derivatives required to highlight edges 
- __smoothing side-effect__: blurring _true edges_, making it more difficult to detect and localize them.

Pixel of _different colors_ will look _similar_ after filtering because of the _blending_.

- _Smoothing_ and _differentiation_ can be carried out jointly within a single step 
	- This is achieved by _computing differences_ of _averages_ (rather than averaging the image and then computing differences) 
	- To try avoiding smoothing across edges the two operations are carried out along _orthogonal directions_.

We can compute the differences on averages. 

![[differences_on_the_averages.png]]
![[averages_edge.png]]
(obviously we can do this operation on the $x$ direction also).
Partial derivative can be used to compute the _magnitude_
- we can take account of the division during the threshold. 

## Prewitt and Sobel operators - another way to approximate partial derivatives

![[prewitt_sobel.png]]
In short:
- __Sobel__ -> the central pixel can be weighted more
- __Prewitt__ -> approximate partial derivatives by _central differences_ of the mean (Prewitt operator)
	- - i.e. $\micro(i, j+1) - \micro(i, j-1)$

## Edges Detection through threshold
-  Detecting edges by __gradient thresholding__ is inherently inaccurate: 
	- It is difficult to choose the right threshold 
	- the image contains meaningful edges characterized by different contrast (i.e. “stronger” as well as “weaker”) 
	- Trying to detect weak edges implies poor localization of stronger ones
![[edges_detection.png]]
A better approach to detect edges may consist in finding the _local maxima_ of the absolute value of the derivative of the signal.
Why? Because a local maxima of the gradient magnitude implies a significant transition in the intesity of the image.

## Non-maxima Suppresion (NMS) 
Since we don't know the threshold to use, we have to use another solution -> we need __NMS__.
When dealing with images (2D signals) one should look for: 
- maxima of the _absolute value_ of the derivative (i.e. the gradient magnitude).  
- along the _gradient direction_ (i.e. orthogonal to the edge direction). 

Look at this picture to better understand:
We are trying to find the _maximum value_ along a certain _axis_. In this example:
- 40, 42, 40, 44 are the maximum values in the x axis (40 is the max between {9, 40, 8})
- 11, 42

Btw, these values are gradient magnitudes. 

![[nms_suppression.png]]
Now, as we've said, we need to look also along the gradient direction. Problem is, ==we don’t know in advance the correct direction to carry out NMS==. 

So, the direction has to be estimated locally (based on _gradient’s direction_). 
To find the direction of the gradient, we need to find the _maximum variation_ of the gradient. 

### Compute the direction of the gradient
![[nms_suppresion.png]]
In this example, we are discretizing the gradient. 
So, we select the _closest point_ to A and B, $(i-1, j+1)$ and $(i+1,j-1)$ respectively (aka the points circled in blue). 
- The magnitude of the gradient has to be _estimated_ at points which _do not belong_ to the _discrete pixel grid_.

However, we can also opt to not use a discrete approximation, but rather use a precise value by means of interpolation. 
Such values can be estimated by _linear interpolation_ of those computed at the _closest points_ belonging to the _grid_.

![[nms_suppression_2.png]]
The value of $\Delta x$ is easily computed using the direction of the gradient.  

### Our pipeline now
![[edge_pipeline_2.png]]
We now add NMS to the pipeline. 
In this way, we can exclude all those values which are not the local maxima. 

## Canny’s Edge Detector
Canny proposed to set forth _quantitative criteria_ to measure the _performance_ of an _edge detector_ and then to find the optimal filter with respect to such criteria.

He proposed the following three criteria: 
- __Good Detection__: the filter should _correctly extract_ edges in noisy images 
- __Good Localization__: the _distance_ between the _found edge_ and the _“true” edge_ should be minimum.
- __One Response to One Edge__: the filter should detect _one single edge pixel_ at each “true” edge (no multiple pixels for the same edge).

Also, we want sharp edges! No thick edges!!

Addressing the 1D case and modeling an edge as a noisy step, Canny shows that the _optimal edge detection operation_ consists in finding ==_local extrema_ of the _convolution_ of the _signal_ by a first order _Gaussian derivative_== (i.e. $G’(x)$).
- aka, __local extrema__ convoluted by a __Gaussian derivative__. 

A straightforward Canny edge detector can be achieved by: 
1. __Gaussian smoothing__ 
2. __Gradient computation__ 
3. __NMS__ along the gradient direction

#### Why Gaussians & Convolutions?
We use Gaussians & Convolutions to exploit both [[2023-03-09 - Filters - Gaussian, Median and Bilateral#Deploying Separability|separability]] and the [[2023-03-06 - Convolutions and Correlations#Properties of Convolutions|commutatibility of differentiaiton]] property of convolutions. 
![[Gaussian_sep.png]]
We can use the two partial derivatives to compute the gradient. 

### Canny’s Edge Detector Threshold
NMS is often followed by _thresholding_ of gradient magnitude to help distinguish between true “_semantic_” edges and _unwanted_ ones.
![[camnny_fuck_uyou.png]]

Edge streaking (narrowing) may occur when magnitude varies along object contours, which also may be caused by a too high threshold. 

To solve this, Canny proposed a “__hysteresis__” thresholding approach relying on a _higher_ ($T_h$) and a _lower_ ($T_l$) threshold.
- A pixel is taken as an edge if either the gradient magnitude is _higher_ than $T_h$ __OR__ _higher_ than $T_l$ AND the pixel is a _neighbor_ of an already detected edge.

Hysteresis thresholding is usually carried out by tracking edge pixels along contours
![[hysteresis.png]]
The result is a set of chains, each chain is a list of pixels belonging to that contour.