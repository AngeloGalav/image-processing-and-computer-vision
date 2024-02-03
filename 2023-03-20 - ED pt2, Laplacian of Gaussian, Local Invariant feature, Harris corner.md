## Canny's Edge Detector - 2
In a situation in which we have a single, strong light source, we have the problem of shadows, which can be confused as edges. So what we do is apply Canny edge detection, and in this way the threshold is recognized correctly.
![[canny_edge_detection.png]]
### Zero crossing
![[zero-crossing.png]]
Instead of using the first derivative to find edges, we can use the _second derivative_, through a process called __zero-crossing__. 
Look for zero-crossing of the second derivative of the signal to locate edges (instead of the peaks of the first derivative) 
- it crosses _zero_ but _before it is immediately positive_ (or negative)… 
- It requires significant __computational effort__!!

### How to compute the second derivative
The second derivative along the gradient’s direction can be obtained as $$n^THn$$
Where $n$ and $H$ are:
![[second_derivative.png]]
Computing the second derivative along the gradient turns out very expensive. In their seminal work on edge detection, instead, Marr & Hildreth proposed to rely on the __Laplacian__ as _second order differential operator_ (as an approximation of the second derivative):
![[laplacian.png]]

##### Discrete Laplacian
To approximate the calculation, instead, we can use _differences on differences_ (since the first derivative is already approximated through differences).  
- One can use _forward_ and _backward_ differences to approximate first and second order derivatives, respectively:

![[difference_on_difference.png]]

It can be shown that the __zero-crossing of the Laplacian__ typically lay close to those of the second derivative along the gradient. Yet, the former differential operator _is much faster to compute_, i.e. just a convolution by a 3x3 kernel, than the latter.
![[laplacian_kernel.png]]
(The kernel is obtain through )

### Laplacian of Gaussian (LOG) 
We still have to deal with noise. A robust edge detector should include _a smoothing step_ to _filter out noise_ (especially in case second rather than first order derivatives are deployed). 
- In their edge detector, Marr & Hildreth proposed to use a Gaussian filter as smoothing operator

__Edge detection__ by the __LOG__ can the be summarized conceptually as follows: 
1. _Gaussian smoothing_: ![[gaussian_smooth_af.png]]
2. Second order _differentiation by_ the _Laplacian_ 
3. Extraction of the _zero-crossing_ of ![[zero_crossing_extraction.png]]
Practical implementations of the LOG may deploy the properties of convolutions to speed-up the computation

- Zero crossing (e.g., the resulting Laplacian is like \[-10, 0, 20\], …sometimes impossible…, we'll never get a precise 0 in the middle).  
- You can find sign changes from minus to plus (and vice versa), like \[-10,20\], between two consecutive pixels 

- Once _a sign change is found_, the actual _edge_ may be _localized_: 
	- At the pixel where the LOG is positive (darker side of the edge) 
	- At the pixel where the LOG is negative (brighter side of the edge) 
	- ==At the pixel where the absolute value of the _LOG is smaller_ (the best choice, as the edge turns out closer to the “true” zero-crossing) ==. 

- To help discarding spurious edges, _a final thresholding step may be enforced_ (usually _based_ on the _slope of the LOG_ at the found zero-crossing)

![[log_examples.png]]

Unlike those based on smooth derivatigative LoG Value (Brighter Side of the ves, ==the LOG edge detector allows the degree of smoothing to be controlled== (i.e. by changing the $σ$ parameter of the Gaussian filter). 
- This, in turn, allows _the edge detector_ to be tuned according to the _degree of noise_ in the image (i.e. higher noise -> larger $σ$) 

Likewise, $σ$ may be used to control the scale at which the _image is analyzed_, with larger $σ$ typically chosen to extract the edges related to main scene structures and smaller $σ$ to capture small size details.

Zero-crossings are usually sought for by scanning the image by both rows and columns to _identify changes_ of the _sign of the LOG_. 

----
# Local Invariant Feature
Several Computer Vision tasks deal with finding “_Corresponding Points_” between two (or more) images of a scene:
- Correspondences = image points which are the projection of the same 3D point in different views of the scene 
- Establishing correspondences may be difficult as these points may look different in the different views (e.g. due to viewpoint variations and/or lighting changes)

## Panorama Stitching
- Create a larger image by aligning two images of the same scene 
	- The two images may be aligned by estimating a homography, which requires at least _4 correspondences_ (more is better) 
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

##### Properties of good detectors 
- __Repeatability__: it should find the _same keypoints_ in _different views_ of the scene despite the transformations undergone by the images 
- __Saliency__: it should find keypoints surrounded by _informative patterns_ (good for making them discriminative for the matching) (i.e. not the sky).

##### Properties of good descriptors
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

(obviously it is not the difference between the 2 squares, but it is the different between points inside each single square). 

## Harris Corner Detector
Harris&Stephens proposed to rely on a continuous formulation of the Moravec’s “error” function.
Assume to shift the image with a generic _infinitesimal shift_ $(Δx, Δy)$: 
- $w(x,y)$ is a __window__ _set to 1_ around the _pixel_ under evaluation and 0 _in all the image_ 
- in the end consider only the pixel around the $x,y$ position 

Due to the shift being infinitesimal, we can deploy _Taylor’s expansion_ of the intensity function at (x,y):
![[taylor_expasion.png]]
Remember, the Taylor expansion consists in $f(x + \Delta x) = f(x) + f'(x)\Delta x$

The final __Harris equation__ that we obtain is this:
![[harris1.png]]
from which we can derive this:
![[harris2.png]]
$M$ encodes the _local image structure_ around the considered _pixel_, and is also called __structure matrix__.  

Now, let us hypothesize that $M$ is a _diagonal matrix_:
![[diagonal.png]]

Since $M$ is _real_ and _symmetric_, it can always be _diagonalized_ by a __rotation__ of the _image coordinate system_ (and obtain the diagonal matrix of lambdas from before). 
- The columns of $R$ are the __orthogonal unit eigenvectors__ of $M$ 
- $λ_i$ the _corresponding eigenvalues_. 
- $R^T$ is the _rotation matrix_ that aligns the image axes to the _eigenvectors_ of $M$
![[M_is_real.png]]
We just compute the eigenvalues of $M$. 
- Rotation invariance: the same eigenvalues are always computed. 

Since we would need to compute _eigenvalues_ at each pixel would be costly, 
we compute a more efficient “_cornerness_” function: 
![[corenerness_function.png]]

The variable $k$ is a _sensitivity factor_ that is used to balance the importance of the trace and the determinant of the matrix $M$ when calculating the corner response. It is a constant that must be chosen carefully; 
- if it's too small, too many features are detected as corners
- if it's too large, too few are detected.

Analysis of the above function would show that:
![[C_values.png]]