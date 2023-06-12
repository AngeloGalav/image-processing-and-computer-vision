
>[!A thing to remember]
In CV, there are 3 invariances that we're interested in:
>- Rotation invariance
>- Scale invariance
>- Illumaition invariance

## To summarize: Harris Corner Detection
The Harris corner detection algorithm can thus be summarized as follows: 
1. Compute $C$ at _each pixel_ 
2. Select all pixels where $C$ is _higher_ than a _chosen positive threshold_ ($T$) 
3. Within the previous set, detect as corners only those pixels that are local maxima of $C$ (NMS)

It is worth highlighting that the _weighting function_ $w(x,y)$ used by the Harris corner detector is _Gaussian_ rather than Box-shaped, so to assign _more weight_ to _closer pixels_ and less weight to those farther away.
![[gaussian_harris.png]]


## Invariance Properties
As we know, objects in images undergo changes (due to rotation, illumination, scale).
- __Rotation invariance__: _eigenvalues_ of $M$ are _invariant to a rotation_ of the _image_ axes and thus so is _Harris cornerness function_.

- However, it is __not subject to illumination invariance__ (for the most part). 
	- Actually, it is light invariant for an _addictive factor_  ($I’ = I + b$) due to the use of derivatives (it gets removed during the computation of the gradient, as shown in this picture).
	![[additive_bias.png]]
	- But, it is _not invariant_ to _multiplication_ by a gain factor ($I’ = a \cdot I$) ==> _derivatives get multiplied_ by the same factor.
![[light_invariance.png]]

##### No scale invariance...
Scale invariance is the most important thing, since we want to work with application in which we dont have control of the enviroment. 

- Also, we are __not__ _scale invariant_! But why...?
- Given the chosen size of the detection window, all the points along the border are likely to be classified _as edges_ 
- Should the object appear smaller in the image, use of the _same window size_ would lead to _detect a corner_ 
![[edge2corner.png]]
The use of a fixed detection window size makes it impossible to _repeatably detect homologous_ features when they appear at _different scales_ in images.

An image contains _features at different scales_, i.e. points that stand-out as interesting as long as a proper neighbourhood size is chosen to evaluate the chosen interestingness criterion.
Detecting all features requires to _analyze_ the image _across the range of scales_ “deemed as relevant”.
The _more features we gather_, the _higher_ the chance to _match_ across pictures.
![[large_small_scale_features.png]]
Depending on the acquisition settings (distance and focal length) an object may look differently in the image, in particular it may exhibit more/less details (i.e. features).

Features do exist within a certain range of scales 
- Finding similar features despite the different scales at which they may appear in different images. 
- The same feature would simply be detected at different steps within a multi-scale image analysis process.

We detect features in order to _match_ their _descriptors_.
Yet, the neighbourhoods surrounding _large scale features_ are far _richer of details_ than those around small scale ones. The higher the scale, the finer the details present in the neighbourhood of a feature. 
To make it possible to compute similar descriptors – and so match them- the details that _do not appear_ across the _range of scales_ should be _cancelled-out_ by means of __image smoothing__.

## Scale-Space
Scale invariance is the main issue addressed by second generation local invariant features.
- key finding: apply a __fixed-size detection__ tool on increasingly _down-sampled_ and _smoothed_ versions of the input image:
![[scale_space.png]]
_This representation_ is known as __Scale Space__: as you move along scales, _small details_ should continuously _disappear_ and no structure should be introduced.

A _Scale-Space_ is a one-parameter _family of images_ created _from the original_ one so that the structures at smaller scales are successively _suppressed_ by _smoothing operations_.
- one would not wish to create new structures while smoothing the images.

### Gaussian Scale-Space
Several researchers have studied the problem and shown that a _Scale-Space_ must be realized by __Gaussian Smoothing__
$$
L(x, y, \sigma) = G(x,y,\sigma) * I(x,y)
$$
A Scale-Space is created by repeatedly _smoothing_ the _original image_ with _larger and larger __Gaussian__ kernels_.

### Feature Detection & Scale Selection
The Gaussian Scale-Space is only a tool to represent the input image at different scales, it _neither_ includes any _criterion_ to _detect features_ nor to select their characteristic scale.

As features do exist across a range of scales…how _do we establish at which scale_ a feature turns out maximally _interesting_ and should therefore be described? 

This problem consists on __multi-scale feature detection__ and __automatic scale selection__.
In order to solve it, we must compute suitable _combinations_ of _scale-normalized derivatives_ of the _Gaussian Scale-Space_ (normalized Gaussian derivatives) and find their _extrema_.

- As we filter more (_higher sigma_), derivatives tends to become _weaker_. To compensate Lindberg proposes to _multiply/normalize_ derivatives by _sigma_ (scale-normalized) 

- We have stacked images => we look for extrema in a 3D space (pixel positions as well as scale)

### Scale-normalized Laplacian of Gaussian (LOG)
Scale-normalized Laplacian of Gaussian (LOG):
![[laplacian_of_gaussian.png]]
Let's examine this picture:
![[laplac.png]]
Considering the centers (red points) of the two dark blobs 
- => the extremum of the scale-normalized LOG is found at a larger scale for the larger blob. 

The ratio between the two characteristic scales is roughly the same as the ratio between the sizes (diameters) of the two blobs.
The idea is to look for _extrema_ across $x, y, σ$ 
- meaning, that we have found a feature that has a _position_ given by $x$ and $y$ and a _scale_ given by the _sigma_ at which it is _maximum_.

#### Multi-Scale Feature Detection
Features (_Blob-like_) and scales detected as _extrema_ of the scale-normalized LOG.
LoG filter extrema locates “blobs” 
- maxima = dark blobs on light background 
- minima = light blobs on dark background

![[multiscale_feature.png]]

## Difference of Gaussian (DoG)
Lowe proposes to detect keypoints by seeking for the extrema of the _DoG_ (_Difference of Gaussian_) function across the $(x,y,σ)$ domain:
![[doggy.png]]
This approach provides a computationally efficient approximation of Lindeberg’s scale-normalized LOG:
![[LOG.png]]
Lowe proves that this is a scaled version of Lindberg. $(k-1)$ is a _constant factor_, it does _not influence extrema location_ => the choice of $k$ is _not critical_.

Both detectors are _rotation invariant_ and find _blob-like features_ (__circularly symmetric filters__).

>[!WARNING]
>DoG allows us to find features as _blobs_ in images, which are _extremas_ in the $(x,y,\sigma)$ domain. 
>Also, Blobs are rotation and scale invariant. 