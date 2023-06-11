# Edge detection
Edges capture semantic features that are important for their semantic content. 
- _Sharp_ change of a 1D signal. 

A 2D Step-Edge is characterized not only by its strength but also its _direction_:
![[edge_detection.png]]
To sense the direction of an edge, we can use the _gradient_, and determine how steep it is by using the _norm_ of the gradient. 

## Discrete approximation of the Gradient
To _discretize derivate_ image derivatives, we can exploit the use of _differences between intensities_. 
We can use forward, backward, central differences:
![[approximation_of_the_gradient_points.png]]
At the same time, we can estimate _magnitude_ of the gradient. The best to use is the __max_norm__.

##### Noise and edges
However, there's a slight problem:
Derivates amplify noise! So we need to find a find a method that it's robust to noise!!
- An edge detection method with this property must first smooth the image in order to delete noise, but in this way, we also _blur the true edges_, making them harder to be recognized.  

We can both compute the difference and the smoothing operation in a single step. 
- We compute the _difference on averages_, on _orthogonal directions_. 
![[differences_on_the_averages.png]]

![[averages_edge.png]]

## Prewitt & Sobel operators
- Prewitt -> approximate partial derivatives by _central differences_ of the mean (Prewitt operator).
	- i.e. $\micro(i, j+1) - \micro(i, j-1)$
- Sobel -> the central pixel can be _weighted more_.

## Edge detection
We could use gradient thresholding to find edges. However, it's a difficult process and it's very inaccurate. 
- A better approach consists in finding the _local maxima_. 

## Non-maxima suppression (NMS)
Detecting edges by __gradient thresholding__ is inherently inaccurate, so we need NMS. 
- ABS of the derivative. 
- gradient direction. 

Discrete direction:
![[nms_suppresion.png]]
Approximate direction:
![[nms_suppression_2.png]]

### Our pipeline now 
![[edge_pipeline_2.png]]


### Canny's edge detector
The final version of this consists in the canny edge detector, which is the same as the image above. 
Canny created 3 criterias to measure the performance of an edge detector. 
- __Good Detection__: the filter should _correctly extract_ edges in noisy images 
- __Good Localization__: the _distance_ between the _found edge_ and the _“true” edge_ should be minimum.
- __One Response to One Edge__: the filter should detect _one single edge pixel_ at each “true” edge (no multiple pixels for the same edge).

Canny's edge detector can be achieved by:
1. __Gaussian smoothing__ 
2. __Gradient computation__ 
3. __NMS__ along the gradient direction

The first 2 parts can be done in a single step:
![[Gaussian_sep.png]]

#### Hysteresis
We need to find the threshold. 
- A pixel is taken as an edge if either the gradient magnitude is _higher_ than $T_h$ __OR__ _higher_ than $T_l$ AND the pixel is a _neighbor_ of an already detected edge.

This way, you get a _set of chains_. 

---
## Laplacian of Gaussian
#### Zero-crossing
Instead of using the first derivative to find edges, we can use the _second derivative_, through a process called __zero-crossing__. Essentially we use the zero of the second derivative of the function.  
- it crosses _zero_ but _before it is immediately positive_ (or negative)… 
- It requires significant __computational effort__!!

### Discrete laplacian
We can use the discrete laplacian, through differences of difference (1st for the first order gradient, then for the )

#### What about noise? (Laplacian of Gaussian)
__Edge detection__ by the __LOG__ can the be summarized conceptually as follows: 
1. _Gaussian smoothing_: ![[gaussian_smooth_af.png]]
2. _Second order differentiation_ throuhg the use of the _Laplacian_. 
3. Extraction of the _zero-crossing_ of ![[zero_crossing_extraction.png]]

- Once _a sign change is found_, the actual _edge_ may be _localized_: 
	- At the pixel where the LOG is _positive_ (_darker_ side of the edge) 
	- At the pixel where the LOG is _negative_ (_brighter_ side of the edge) 
	- ==At the pixel where the absolute value of the _LOG is smaller_ (the best choice, as the edge turns out closer to the “true” zero-crossing) ==. 

A final thresholding step may be enforced, usually based on the slope of the LOG. 

#### Why Laplacian of Gaussian is cool
- It allows to control the degree of smoothing, so the _tuning_ of the ED _according_ to _noise_. 
- Also allows the degree of the scale for the analysis of the image: small $\sigma$ for capturing little details. 
