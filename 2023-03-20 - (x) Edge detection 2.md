## Canny's Edge Detector - 2
In a situation in which we have a single, strong light source, we have the problem of shadows, which can be confused as edges. So what we do is apply Canny edge detection, and in this way the threshold is recognized correctly.
![[canny_edge_detection.png]]
### Zero crossing
![[zero-crossing.png]]
Instead of using the first derivative to find edges, we can use the _second derivative_, through a process called __zero-crossing__. 
- We find the edge in the zero of the function, in particular a zero which is between positive and negative values. 

Look for zero-crossing of the second derivative of the signal to locate edges (instead of the peaks of the first derivative) 
- it crosses zero but before it is immediately positive (or negative)… 
- It requires significant computational effort

The second derivative along the gradient’s direction can be obtained as $n^THn$. 
![[second_derivative.png]]
Computing the second derivative along the gradient turns out very expensive. In their seminal work on edge detection, instead, Marr & Hildreth proposed to rely on the __Laplacian__ as _second order differential operator_ (as an approximation of the second derivative):
![[laplacian.png]]

To approximate the calculation, instead, we can use _differences on differences_ (since the first derivative is already approximated through differences).  
- One can use _forward_ and _backward_ differences to approximate first and second order derivatives, respectively:

![[difference_on_difference.png]]

It can be shown that the zero-crossing of the Laplacian typically lay close to those of the second derivative along the gradient. Yet, the former differential operator is much faster to compute, i.e. just a convolution by a 3x3 kernel, than the latter.
![[laplacian_kernel.png]]
(The kernel is obtain through )

### Laplacian of Gaussian (LOG) 
We still have to deal with noise. A robust edge detector should include _a smoothing step_ to _filter out noise_ (especially in case second rather than first order derivatives are deployed). 
- In their edge detector, Marr & Hildreth proposed to use a Gaussian filter as smoothing operator

Edge detection by the LOG can the be summarized conceptually as follows: 
- Gaussian smoothing: ![[gaussian_smooth_af.png]]
- Second order differentiation by the Laplacian 
- Extraction of the zero-crossing of ![[zero_crossing_extraction.png]]

Practical implementations of the LOG may deploy the properties of convolutions to speed-up the computation
- Zero crossing (e.g., the resulting Laplacian is like \[-10, 0, 20\], …sometimes impossible…, we'll never get a precise 0 in the middle).  
- You can find sign changes from minus to plus (and vice versa), like \[-10,20\], between two consecutive pixels 

- Once a sign change is found, the actual edge may be localized: 
	- At the pixel where the LOG is positive (darker side of the edge) 
	- At the pixel where the LOG is negative (brighter side of the edge) 
	- ==At the pixel where the absolute value of the _LOG is smaller_ (the best choice, as the edge turns out closer to the “true” zero-crossing) ==. 

- To help discarding spurious edges, _a final thresholding step may be enforced_ (usually based on the _slope of the LOG_ at the found zero-crossing)

![[log_examples.png]]

Unlike those based on smooth derivatives, ==the LOG edge detector allows the degree of smoothing to be controlled== (i.e. by changing the $σ$ parameter of the Gaussian filter). 
- This, in turn, allows the edge detector to be tuned according to the degree of noise in the image (i.e. higher noise -> larger $σ$) 

Likewise, $σ$ may be used to control the scale at which the image is analyzed, with larger $σ$ typically chosen to extract the edges related to main scene structures and smaller $σ$ to capture small size details.

Zero-crossing are usually sought for by scanning the image by both rows and columns to identify changes of the sign of the LOG. 


# Local Invariant Feature
Several Computer Vision tasks deal with finding “__Corresponding Points__” between two (or more) images of a scene:
- Correspondences = image points which are the projection of the same 3D point in different views of the scene 
- Establishing correspondences may be difficult as these points may look different in the different views (e.g. due to viewpoint variations and/or lighting changes)

## Panorama Stitching
- Create a larger image by aligning two images of the same scene 
	- The two images may be aligned by estimating a homography, which requires at least 4 correspondences (more is better) 
	- Find “_salient points_” independently in the two images 
		- points that are _distinctive_. 
	- Compute a local “_description_” for each salient point.  
	- _Compare_ descriptions to find matching pairs. 
![[panorama_stitching.png]]

## The Local Invariant Features Paradigm
The task of establishing correspondences is split into 3 successive steps:
- __Detection__ of _salient points_ (aka keypoints, interest points, feature points...) 
- __Description__ - computation of a _suitable descriptor_ based on pixels in the keypoint neighbourhood 
- __Matching__ descriptors between images

_Descriptors_ should be _invariant_ (robust) to as many transformations as possible. 

##### Detector 
- __Repeatability__: it should find the _same keypoints_ in _different views_ of the scene despite the transformations undergone by the images 
- __Saliency__: it should find keypoints surrounded by _informative patterns_ (good for making them discriminative for the matching) (i.e. not the sky).

##### Descriptor
- __Distinctiveness vs. Robustness Trade-off__: the description algorithm should _capture the salient information_ around a keypoint, so to _keep important tokens_ and disregard changes due to nuisances (e.g. light changes) and noise. 
- __Compactness__: the description should be as _concise as possible_, to minimize memory occupancy and allow for efficient matching.

_Speed_ is desirable for both, and _in particular for detectors_, which need to be run on the whole image (while _decriptors_ are computed _at keypoints only_). 

## Moravec Interest Point Detector
Edge pixels can be hardly told apart as they look very similar along the direction perpendicular to the gradient 
- _Edges are locally ambiguous_, many other points that look just the same.
![[Images/edges.png]]
- Pixels exhibiting a large variation along all directions => better for establishing reliable correspondences. 

The _cornerness_ at $p$ is given by the minimum squared difference between the patch (e.g. 7x7) centered at $p$ and those centered at its 8 neighbours.
![[moravec_intereset_point_detector.png]]
![[corner_detection.png]]

(obviously it is not the difference between the 2 square, but it is the different between points inside each single square). 


$M$ encodes the local image structure around the considered pixel, and is also called _structure matrix_.  