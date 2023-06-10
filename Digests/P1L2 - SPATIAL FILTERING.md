## Denoising
##### Temporal mean
We could try to remove noise using multiple images in a certain time frame:
![[denoising_formula1.png]]

##### Spatial mean
Or, we could try to compute a mean _across neighbouring pixels_:
![[single_image_denoising.png]]
K is the __supporting window__, which defines the neighbouring region around my pixel. 
Obviously there's a tradeoff in choosing the window filter. 

## Image filters
_Image Filters__ are image processing operators that compute the _new intensity_ (colour) of a pixel $p$, based on the _intensities_ (colours) of the neighbourhood of pixels. 
- An important _sub-class of filters_ is given by __Linear and Translation-Equivariant__ (__LTE__) operators. 

### Linearity and translation-equivariance
- Given an input 2D signal $i \{x, y\}$
A 2D operator, $T\{\cdot\} :o(x,y) = T\{ i \{x, y\}\}$, is said to be __linear__ iff:
![[formulas_2.png]]
and $a,b$ are two constants.

The operator is said to be __translation-equivariant__ iff:
![[formulas_3.png]]

## Convolution
![[convolution_example.png]]

To sum up, we apply a kernel.
This kernel is flipped and the translated over the image. 

#### Properties of convolutions
The convolution operation is often denoted using the symbol “∗”, e.g.
$$
o(x,y) = i(x,y) ∗ h(x, y)
$$
 - __Associativity__: gives us the possibility of performing operations more efficiently: doing a chain of operation is better (in this case) than doing a single, big operation. 
- __Commutativity__.
- __Distributive__ property w.r.t. to the Sum: $f * (g + h) = f * g + f *h$.
- __Convolution Commutes with Differentiation__ -> $(f * g)' = f' * g = f *g'$.

## Correlation
 The correlation of signal $i(x,y)$ w.r.t. signal $h(x,y)$ is defined as:
![[correlation_1.png]]
- Unlike convolution, _correlation is not commutative_!!!!
![[correlation_view_example.png]]

## Discrete Convolution
![[discrete_convolution.png]]

Conceptually, to obtain the output image we need to _slide the kernel_ across the whole input image and _compute the convolution at each pixel_. 

## Mean filtering
It's the simplest way of denoising an image. 
- It consists in replacing each pixel intensity with the average intensity over a chosen neighbourhood. 
- Mean filtering is inherently fast because _multiplications are not needed_ (you can multiply only once).

Now this is great, however it is too blurry and it also throws the details of our image away.

---
## Gaussian filtering
As $\sigma$ gets larger, _small details disappear_ and the image content deals with larger size structures. Thus, filtering with a chosen $\sigma$ can be thought of as setting the “scale” of _interest_ to analyse image content.

The discrete Gaussian kernel can be obtained _by sampling the corresponding continuous function_. As for the size of the kernel:
- We should use _larger sizes_ for filters with _high_ $\sigma$, _smaller sizes_ whenever $\sigma$ is _smaller_. 
-  a typical rule-of-thumb dictates taking a $(2k + 1)×(2k + 1)$ _kernel_ with $k = \lceil 3\sigma \rceil$.

### Separability
To further speed-up the filtering operation, one can deploy the separability property: due to the _2D Gaussian_ being the product of _two 1D Gaussians_, the original __2D convolution can be split into the chain of two 1D convolutions__, i.e. 

## Median filter 
It's _non-linear filters_ whereby each pixel intensity is replaced by the _median_ over a given _neighbourhood_.
Good with salt&pepper noise, but _it cannot deal with Gaussian noise.

## Bilateral filter
The bilateral filter allows to denoise Gaussian-like noise without blurring the image (aka preserving the edges). 
![[output_bilateral.png]]
Where:
![[formulas_expanded.png]]

In this type of filter, the values of the kernel $H$ will be the highest when the distance (both spatial and intensity) between $p,q$ is small. 
- The pixel on the other side of the edge contribute less.

## Non-local means filter
The key idea is that the _similarity among patches_ spread over the image can be _distributed_ to _achieve denoising_.

---
