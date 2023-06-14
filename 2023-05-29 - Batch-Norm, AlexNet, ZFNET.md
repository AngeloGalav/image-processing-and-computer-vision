## Batch-normalization

#### Internal Covariate Shift
Problem: even when using ReLUs instead of sigmoids, and when properly initializing all the layers, _deep architectures_ following the pattern presented in the previous slides were very _hard and/or slow to train_. 

One (maybe not correct…) intuition: if we consider the $𝑊_2$, $𝑏_2$ linear classifier in the neural network above, the (distribution of the) __representation vector__ $𝒉$ tries to _classify changes_ at each _training iteration_, because $𝑊_1$ and $𝑏_1$ are updated.

Moreover, the gradient that $𝑊_2$ and $𝑏_2$ receive was computed to _improve performance_ for the “old” distribution of $𝒉$. This effect becomes worse when multiple layers influence the changes of $𝒉$. 

This phenomenon is referred to as __internal covariate shift__: the change in the _distribution of network activations_ due to the change in _network parameters_ during _training_.

### Batch Norm Layer at training time
![[bath_norm at training time.png]]
The idea behind Batch Normalization is to normalize the output of a layer _during training_, so that _each dimension_ has __zero mean__ and __unit variance__ in a _batch_ (or rather, wrt a batch).
- _Learnable_ per-dimension __scale__ ($𝜸_𝒋$) and __shift__ ($𝜷_𝒋$) _parameters_ to reintroduce _flexibility_ into learned representations.
- $s$ is the normalized, _final_ output activation.
	- It is the input to the classifier.

##### Batch-norm details at training time
Given this formulas:
![[train_time_form.png]]

The batch-norm has:
- Input: a mini-batch of _activations_ $\{𝑎^{(𝑖)}| 𝑖 = 1, … , 𝐵\}$ where each sample has _dimension_ $D$.
- It has 2D _learnable parameters_ $𝜸_𝒋$ and $𝜷_𝒋$, for each dimension $D$ ($j = 1,...,D$).
- BN is comprised of 4 steps, all of them are _differentiable_ and can be _back-propagated through_: it guaranties that ==optimization does not “_undo_” _normalization_==.
- Learning $𝛾_𝑗 = \sqrt{𝑣_𝑗}$ and $𝛽_𝑗 = 𝜇_𝑗$ may recover the identity function, if it were the optimal thing to do.
- At training time, we also keep a _running average_ of _mean_ and _variance_ ($𝛽 = 0.1$ is usually called __BN momentum__).
- This is the _running average_:
$$
\begin{matrix}
𝜇_𝑗^{(𝑡)} = (1 − 𝛽) 𝜇_𝑗^{(𝑡−1)} + 𝛽𝜇_𝑗 \\
𝑣_𝑗^{(𝑡)} = (1 − 𝛽) 𝑣_𝑗^{(𝑡−1)} + 𝛽𝑣_j
\end{matrix}
$$
##### Batch normalization at test time
At test time:
- Input: a mini-batch of _activations_ $\{𝑎^{(𝑖)}| 𝑖 = 1, … , 𝐵\}$ where each sample has _dimension_ $D$.
- It has 2D _learnable parameters_ $𝜸_𝒋$ and $𝜷_𝒋$, for each dimension $D$ ($j = 1,...,D$).

But also...
At test time, we do not want to stochastically _depend_ on the _other items_ in the mini-batch: we want the _output to depend only on the input_, __deterministically__.  D

Mean and variance become _two constant_ values, the _final values_ of the __running averages__ computed at _training time_.

Hence, _BN becomes a deterministic linear transformation_, which can also be fused with the previous fully connected or convolutional operation

$𝜇_𝑗$ = ==final value of mean during the running average computed at training time==. It’s _constant_ at test time.
$𝑣_𝑗$ = ==final value of variance during the running average computed at training time==. It’s _constant_ at test time.

The final result of BN at test time is:
![[test_time_from.png]]

#### Pros & Cons of BN
_Pros_ 
- allows the use of _higher learning rates_. 
- careful initialization is less important 
- Training is not deterministic, acts as _regularization_ (prevents overfitting)
- No _overhead at test-time_: so much so that it can be _fused with previous layer_.
__Cons__ 
- Not clear why it is so beneficial 
- Need to distinguish between _training_ and _testing_ time makes implementations more complex, source of many bugs 
- Does not scale down to “_micro-batches_”
![[bn.png]]

## Batch Norm for convolutional layers
In Batch norm for _convolutional layers_, we normalize along mini-batch and _spatial dimensions_ -> out activation has 4 dimensions:
![[batch_norm_cnn.png]]
Why? The idea is to respect how _convolutional layers work_: elements of the same feature map are normalized in the same way.
This type of normalization is also known as Spatial Batch Norm, BatchNorm2D.

## Batch Norm alternatives
[he will not ask this stuff]
There are alternatives to Batch Norm:
![[batch_norm_alternatives.png]]

-----
# Successful Architectures
## AlexNet
![[cnn2.png]]
- The convolutions are the smallest cube in this image

#### Input
- Input elements = $𝑊_{𝑖𝑛} × 𝐻_{𝑖𝑛} × 𝐶_{𝑖𝑛}$ =227\*227\*3 =154,587
- mini-batch size 128,
	- activations are usually floating point number, i.e. _4 bytes_ for each element.

#### First layer
- First layer is a 96x11x11 convolutional layer, with __stride 4__. 
- Output channel = $n_{kernels}$ = 96
- Activation H/W (Output size) = (227 - 11 + 0) / 4 + 1 = 216 / 4 + 1 = _55_

We will see again this __stem layer__ at the beginning of convnets, i.e. a conv layer that performs a _fast reduction_ in the _spatial size of the activations_, mainly to _reduce memory and computational cost_, but also to rapidly _increase the receptive field_.
- This is beacuse S=4, which was done in other to bring down the dimension of the inputs as fast as possible. 

This essentially means that we're extracting 96 different views of our small (55x55) image.
So, in turn, the number of activations are a lot more than the one in the input.
- The quantity of information is increased
- total activations = 55 * 55 * 96 = 290400 

##### Memory and time consuption
The total number of parameters of this layer is = (11 * 11 * 3 +1 ) * 96 = 35k

To process a minibatch just in this layer, we need 27 GFlops!!!

Given activation size and $n_{params}$, how _much memory_ will we need at training time?
- For activations, we need to store the _activation_ itself AND the _gradient_ of the loss with respect to everyone of its entries, i.e. another tensor of the same size.
	- We will have _as many activations (and gradients)_ as there are _input images_ in our mini-batch.

- For every parameter, we will need to store its _value_ and the _gradient of the loss_ with respect to it 
- If using momentum, we will have a velocity term for each parameter, if using Adam first and second order moments for each parameter, etc.. 
 
Hard to have a precise estimate of memory requirements, but we can get approximate values by considering _twice the activation size_ and 3-4 times the $n_{params}$. This is usually a _lower bound_ on the actual requirements.

###### Some stats
Size of activations for conv1 is then 2 * (128 * #activations * 4 / 1024/1024)= 283.6 MB Size of parameters for conv1 is then 3*#params * 4 / 1024/1024= 0.4 MB

>[!IN SHORT]
>Convolutional layer are like this:
>- they consume a lot memory for activation
>- require a lot of computational resources
>- but they have a small number of parameters

#### Max-Pooling layer
All pooling layers are “overlapping” pooling layers: they have size 3x3, but _stride 2_ (so we halve). Reduces the top-1 and top-5 errors in their experiments compared to 2x2 with stride 2.

The pooling layer memory consumption/computational cost is extremely low, so we can actually ignore this facts. 

#### Flatten layer
Flatten layer to throw away the spatial structure and prepare for FC layers.
This layer requires no parameters and no computation, nor memory consumption. 
It's a just different view of the data structure. 

#### Finals stats
![[AlexNet_stats.png]]
At the end of our net (before being put inside the flatten layer), we have a 6x6x256 activations as output. That's super coarse!
- It's basically a 6x6 image!

The final output has size 4096, and uses 4MB to be stored. 
On the other hand, 430MB are required to store all 38M parameters of the net.
(not considering the dense layers parameters, otherwise would have 63M params). 

#### Trends to note
Trends to note: 
- ==_Stem layer_ at the beginning of the network== 
- __Nearly all parameters__ are in the _fully connected layers_. 
- Largest __memory__ consumption from _activations_ due to the first conv layers.
- Largest number of __flops___ required by the _conv layers_. 
- Activations and parameters have a _similar memory footprint_ at training time (this wont be true for other nets). 

_More than 60 million parameters learned, more than 290 Gflops to process a mini-batch, 2.2 Gflops/image_.

## ZFNET
![[ZFNET_clear.png]]
![[ZFNET.png]]
__ZFNET__ tries to reduce the “_trial and error_” approach to network design, by introducing _powerful visualizations_ (via Deconvnets) for layers other than the first one. 

##### The Stem layer changes from AlexNet!!
Based on the visualizations and ablation studies, they found out that _aggressive stride_ and _large filter size_ in the _first layer_ results in __dead filters__ and missing frequencies in the first layer filters and aliasing artifacts in the second layer activations.

They propose to counteract these problems by using $𝟕 × 𝟕$ convs with _stride 2_ in the first layer and _stride 2_ also in the second $𝟓 × 𝟓$ conv layer.