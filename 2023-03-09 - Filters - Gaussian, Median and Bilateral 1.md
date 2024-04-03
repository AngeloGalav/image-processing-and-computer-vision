##### Some remarks regarding the previous lesson
When using linear noise filtering (i.e. _mean filtering_), we can see that while the noise is reduced, _the whole image becomes blurred_. 
With mean filter, we are smoothing little details away along with the noise. So, in general, we are _losing sharpness_.
The blur increases  with the dimensions of the kernel (small kernel -> small blur). 

#### Salt and pepper noise (aka Impulse noise)
This kind of noise is basically made of pixel have very different intensities w.r.t. to the rest of the image. 
![[salt_pepper.png]]
Can be caused by either:
- _Physical damage_ to sensor
- Signal transmission error. 
- Disparity maps: we compare the distance to every point of this region.  
	- If we are comparing two different images, we'll get the salt and pepper noise on the disparity map.  

This type of noise is an asshole, since a filtering operation (such as mean filtering) causes these intensities to be _spread out_ on the whole image. 

## Gaussian filter
LTE operator whose impulse response is a 2D Gaussian function (with zero mean and constant diagonal covariance matrix). 

This type of filter gives _higher weights to the closer pixels_, while giving lower weights to the further away pixels. 
- It works because pixels that are further away should not influence the pixel. ![[simple_gaussian.png]]

The higher $\sigma$, the stronger the smoothing caused by the filter. This can be understood, e.g., by observing that ==as $\sigma$ increases, the weights of closer points get smaller while those of farther points get larger==.
![[gaussian_artifacts.png]]
As $\sigma$ gets larger, _small details disappear_ and the image content deals with larger size structures. Thus, filtering with a chosen $\sigma$ can be thought of as _setting the “scale”_ of interest to analyse image content.

>[!Information]
>In CV, Gaussian fillters are used to create a _scale space_, which is a space in which we define the scale at which we want to analyze our image, so it like seeing the image at a different scale.
> - Many gaussian filters with a different $\sigma$ are applied to the input image, obtaining the scale space. This creates many images at different degrees of blurriness (different scales, aka levels of details), allowing for the analysis of image structures at various sizes. 
> - To extract features, we try to find the _scale invariant features_. Gaussian filters allow us to create a scaled space to extract these features. 

The discrete Gaussian kernel can be obtained _by sampling the corresponding continuous function_, which is however of infinite extent. A finite size must therefore be properly chosen.

To this purpose, we can observe that: 
- The larger is the size, the more accurate turns out the discrete approximation of the ideal continuous filter. 
- The computational cost _grows_ with _filter size_. 
- The Gaussian gets smaller and smaller as we move away from the origin.

- We should use _larger sizes_ for filters with _high_ $\sigma$, _smaller sizes_ whenever $\sigma$ is _smaller_. 
- Rule-of-thumb to chose the size of the filter given $\sigma$: 
	- as the interval $[−3\sigma, +3\sigma]$ captures 99% of the area (“energy”) of the Gaussian function, a typical rule-of-thumb dictates taking a $(2k + 1)×(2k + 1)$ _kernel_ with $k = \lceil 3\sigma \rceil$.
![[Gaussian_kernel.png]]

### Deploying Separability
To further speed-up the filtering operation, one can deploy the __separability property__ of Gaussian: due to the _2D Gaussian_ being the product of _two 1D Gaussians_, the original _2D convolution can be split into the chain of two 1D convolutions_, i.e. either along $x$ first and then along $y$, or viceversa.
![[deploy.png]]

As we can see, this way we have a speed increase when computing this filter!!
However, Gaussians are still going to spread the noise, since it's still a weighted sum. So we need non-linear filters. 

## Median filters
The __median filter__ is a _non-linear filters_ whereby each pixel intensity is replaced by the _median_ over a given _neighbourhood_, the median being the value falling half-way in the sorted set of intensities.
![[median_filter.png]]

Median is _very good with impulse noise_, but _it cannot deal with Gaussian noise._
So we combine the 2 filters. 

## Bilateral Filter
Advanced non-linear filter to accomplish _denoising of Gaussian-like noise_ without blurring the image (aka _edge preserving_ smoothing)
![[window_bilateral.png]]
![[output_bilateral.png]]
[the image is wrong: it should be inverted -> s depends on spatial distance, r on pixel intensity]
In particular:
![[formulas_expanded.png]]

The _kernel_ $H$ is the multiplication of those two function. $s$ stands for _spatial_, $r$ stands for _range_.

==This Gaussian is gonna be high when the pixels have a similar intensity and they are near each other==. In that case, the weight of the kernel is also gonna be high, thus the contribution of kernel overall is gonna be high on that pixel. 

_Normalization Factor_ (Unity Gain):
$$W(p) = \sum_{q \in s} G_{\sigma_{s}} (d_s(p,q)) G_{\sigma_{r}} (d_r(p,q))$$
==It is useful to not change the _overall intensity_ in the image==.
- If the kernel is not summed to 1, I'm gonna have to normalize since the sum does not sum to 1, otherwise we are shriking or amplifying the intensity of the whole image. 
- Used also in Gaussian noise. 

###### Consequences of bilateral filter
- Given the supporting neighbourhood, neighbouring pixels take a larger weight as they are both _closer_ and _more similar_ to the central pixel. 
- At a pixel nearby an edge, the neighbours falling on the other side of the edge look quite different and thus cannot contribute significantly to the output value _due to their weights being small_.
	- They are small because the intensity is different and they are far. 

>[!WARNING]
>A limitation of this filter is that __it may remove details or even edges that are not too defined__!
### Non-local Means Filter
Another well-known non-linear edge preserving smoothing filter. The key idea is that the _similarity among patches_ spread over the image can be _distributed_ to _achieve denoising_.
![[non_local_means.png]]
We're basically using the pixel-by-pixel difference from 2 patches. 
$h$ is an hyper-parameter called bandwith that must be found through trial and error. 