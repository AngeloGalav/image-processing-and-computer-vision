## Understanding Noise
Noise appears in the image as random fluctuations of the actual values. 
Noise changes continuosly, is not always constant obviously. I.e. noise at time $k=1$ is not the same as noise at time $k=10$. 

Noise is more apparent in _uniform regions_.
![[noise_example.png]]
$\hat I$ represent the _real_ value of the colour in the scene, and it is not dependent on $k$. 
$n_k(p)$ is independent and identically distributed:
- It does not depend on the noise of other pixels.  
- The noise is _identically distributed_, and is equivalent to a _Gaussian distribution_ with 0 mean ($N(0, \sigma)$). 
That's why we call it a __Gaussian noise__.

If we assume that the camera is not moving and that the scene is not changing, I can denoise the picture by taking an _average of the pixels intensity_ in sequent _time frames_ $k$. 
![[denoising_formula1.png]]
Since the noise has a 0-average Gaussian distribution, then the _average_ of $n_k(p)$ _approaches 0_ when increasing N.

### What if we're given a single image?
We may compute a mean _across neighbouring pixels_, i.e. a spatial rather than temporal mean: 
![[single_image_denoising.png]]
$K$ is called a __supporting window__, which defines a _neighbouring region_ around my pixel.  
If the window is too large, we may consider pixels from different objects and average their value (and we dont want that). 
- Trade-off in window size. 

## Image filters
__Image Filters__ are image processing operators that compute the _new intensity_ (colour) of a pixel $p$, based on the _intensities_ (colours) of those belonging to a _neighbourhood_ (aka support $S$) of p.
![[image_filter_formula.png]]
They accomplish a variety of useful image processing functions, such as e.g. denoising and sharpening (edge enhancement). 
An important _sub-class of filters_ is given by __Linear and Translation-Equivariant__ (__LTE__) operators. 

Signal theory: their _application_ in image processing (of LTE operation) consist in a _2D convolution_ between the input image and the impulse response function (point spread function or kernel) of the LTE operator.
LTE operators are used as feature extractors in CNNs (Convolutional Neural Networks).

### Linearity and translation-equivariance
Given an input 2D signal $i \{x, y\}$ , a 2D operator, $T\{\cdot\} :o(x,y) = T\{ i \{x, y\}\}$, is said to be __linear__ iff:
![[formulas_2.png]]
and $a,b$ are two constants.

The operator is said to be __translation-equivariant__ iff:
![[formulas_3.png]]

If the operator is LTE, the output signal is given by the _convolution_ between the input signal and the impulse response (point spread function), $h(x, y) = T\{\delta (x, y)\}$, of the operator:
![[formulas_4.png]]
where: 
- $\delta$ is the _unit impulse_. 
- $h$ is _impulse response_.

### Graphical view of convolution
![[convolution_example.png]]
- first we do a reflection, then we apply a translation. 

### Properties of Convolutions
The convolution operation is often denoted using the symbol “∗”, e.g.
$$
o(x,y) = i(x,y) ∗ h(x, y)
$$
- __Associativity__: gives us the possibility of performing operations more efficiently: doing a chain of operation is better (in this case) than doing a single, big operation. 
- __Commutativity__.
- __Distributive__ property w.r.t. to the Sum: $f * (g + h) = f * g + f *h$.
- __Convolution Commutes with Differentiation__ -> $(f * g)' = f' * g = f *g'$.

#### Correlation
- The correlation of signal $i(x,y)$ w.r.t. signal $h(x,y)$ is defined as:
![[correlation_1.png]]
- Accordingly, the correlation of $h(x,y)$ w.r.t. $i(x,y)$ is given by:
![[correletation_2.png]]
- Unlike convolution, _correlation is not commutative_:
![[correletation_3.png]]

### Correlation graphical view
![[correlation_view_example.png]]
I'm just moving the kernel $(x,y)$

#### Convolution and Correlation
The correlation of $h$ w.r.t. $i$ is similar to convolution: the product of the two signals is integrated after translating $h$ _without reflection_.
Hence, if h is an even function ($h(x,y) = h(−x, −y)$), the convolution between $i$ and $h$, $i ∗ h = h ∗ i$ , _is the same as the correlation_ of $h$ w.r.t. $i$.
![[formulas_5.png]]

It is worth observing that _correlation is never commutative_, even if $h$ is an even function. 

To recap:
- Convolution is commutative 
- Correlation is not commutative.
- If h is an even function: $i ∗ ℎ = ℎ ∗ i =  h ∘ i$

>[!FUN FACT]
_Convolutional Neural Networks_ are actually _Correlational Neural Networks_.

## Discrete Convolution
![[discrete_convolution.png]]
where $I(i,j)$ and $O(i,j)$ are the discrete 2D _input_ and _output_ signals, respectively, and $H (i,j) = T \{\delta(i,j)\}$ is the _kernel_ of the discrete LTE operator.
- i.e. the response to the 2D discrete unit impulse (Kronecker delta function), $\delta(i,j)$. 

As for continuous signals, discrete convolution consists in _summing the product_ of the _two signals_ where _one_ has been _reflected about the origin_ and _translated_. 

==The previously highlighted four major convolution properties hold for discrete convolution too==.

#### Practical Implementation
In image processing, both the _input image_ and the _kernel_ are stored into _matrixes_ of given _finite sizes_, with the image being much larger than the kernel. One would cycle through the kernel:
![[image_kernel_matrix_example.png]]

![[discrete_conv.png]]

Conceptually, to obtain the output image we need to _slide the kernel_ across the whole input image and _compute the convolution at each pixel_ (do not overwrite the input matrix!, the convolution matrix output has to be saved on a different matrix). 

The number of computation is $2\cdot(2k+1)^2$, since for each element I'm doing a _multiplication and a sum_ (2 ops). 

Border Issue, two main options: 
- CROP (common in image processing) 
- PAD (preferred in CNNs) 
	- How to pad: zero-padding, replicate (aaaIa..…dIddd), reflect (cbaIabc..…dfgIgfd), reflect_101 (dcblabcd......efghlgfe).

## Mean Filter
Mean filtering is the simplest (and fastest) way to denoise an image. It consists in replacing each _pixel intensity_ by the _average intensity_ over a chosen neighbourhood (e.g. 3x3, 5x5, 7x7…).

![[meanie.png]]

- According to signal processing theory, the Mean Filter carries out a __low-pass filtering__ operation, which in image processing is also referred to as __image smoothing__. 
- Smoothing is often aimed at image denoising, though sometimes the purpose is to cancel out small-size unwanted details that might hinder the image analysis task. 
- Mean filtering is inherently fast because _multiplications are not needed_ (you can multiply only once).

![[mean_filtering1.png]]

-----
Old todos in case I forget something important
# TODOS
- find dirac function definition
- convolution definition
- modify the convolution image
- Add point references to practal implementation in discrete convolution
