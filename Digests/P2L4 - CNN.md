[..]

## CNNs
[..]

#### Equivariance 
So, convolutions (aka correlations) are _equivariant_ wrt _translation_, but not wrt to rotation or scale. 

#### Multiple input channel
While we have 3 dimensions in convolution, this is still a 2D operation: the kernel is slid on only 2 dimension, and computes all 3 layers at the same time. 

#### Activation
As we slide the kernel onto the image, we obtain a _single value output_, called __activation__. It's also called __feature map__.

#### Convolutional layer composed of many filters
We can add multiple kernels/filters to a single layer, in order to obtain a _set of filters_.
The final equation that we'll obtain for the conv layer is this:
![[overall.png]]
Remember this like the ave maria:
- _The number of __output channels__ is equal to the __number of kernels___. 

## Introducing non-linearity
Since CNNs as of now are basically contrained linear operator, to meaningfully connect them we need some kind of non-linear operators. 

#### Padding (valid padding)
If we have no padding in the input (aka __valid padding__) we may shrink our feature maps, according to this formulas:
$𝐻_{𝑜𝑢𝑡} = 𝐻_{𝑖𝑛} − 𝐻_{𝑘} + 1$
$𝑊_{𝑜𝑢𝑡} = 𝑊_{𝑖𝑛} − 𝑊_{𝑘} + 1$

#### Zero padding
A common solution is to add 0s to the initial image, in this way we preserve the dimension following this equation:
$𝐻_{𝑜𝑢𝑡} = 𝐻_{𝑖𝑛} − 𝐻_{𝑘} + 1 + 2P$
$𝑊_{𝑜𝑢𝑡} = 𝑊_{𝑖𝑛} − 𝑊_{𝑘} + 1 + 2P$

### Receptive fields
The _input pixels_ affecting a _hidden unit_ are called its __receptive field__.
If we apply a $H_k \times W_k$ kernel at each layer, the receptive field size of an element in the $L$-th activation would be:
$$r_L = [L(H_k-1)+1]\times[L(W_k-1)+1]$$
Where $L$ is the number of layers. 
Since the receptive field size grows linearly with the number of layers, to capture large features we may need a huge number of layers. 
- Instead, we __downsample__ the activations inside the network. 

##### Strided convolutions
If we apply a $𝐻_𝐾$ × $𝑊_𝐾$ kernel size at each layer, but with stride $𝑆_𝑙$ at layer 𝑙, the receptive field of an element in the 𝐿-th activation has size:
![[strided_form.png]]
i.e., the _size of the receptive field_ grows __exponentially__ with respect to the _number of layers_ with stride > 1.

The output size with these two formulas is defined as:
$$
\begin{matrix}
𝐻_{𝑜𝑢𝑡} = \lfloor\dfrac{𝐻_{𝑖𝑛}−𝐻_{𝐾}+2𝑃}{S}\rfloor + 1 \\
𝑊_{𝑜𝑢𝑡} = \lfloor\dfrac{𝑊_{𝑖𝑛}−𝑊_{𝐾}+2𝑃} {S}\rfloor+ 1
\end{matrix}
$$
## Layers in CNNs
Some other layers in CNNs are:
- _non-linear activation functions_ 
- _fully connected layers_ 
- __pooling__ layers 
- __(batch-)normalization layers__

### Pooling layers
Pooling layers are layers that aggregate several values into a single output value through a function, using a _prespecified (not learned) kernel_.

Key differences with normal convolutions:
- each input channel is _computed independently_.
- consequently, $C_{out}$ = $C_{in}$.

Using pooling layers is an apriori decision, since I can't learn this type of kernels (especially max-pooling).

#### Max-pooling
![[max_pooling.png]]
__Downsampling comes from stride__, _not from pooling_!!!
Compared to strided convolutions, max pooling 
- has _no learnable parameters_ (pro and con). 
- provides _invariance_ to small _spatial shifts_.

## Flattening tensors
To be inputted inside a _Dense layer_, the image is flattened. 
In CNNs
- the convolutionals part is called __feature extractor__.
- the dense part is called __classifier__.

The Dense part of the CNN has a global vision of the image.

----
## Bitch-norm
#### Internal Covariate Shift
Deep architecture, even when using ReLUs, are too fucking _slow to train_.
- This could be due to the __internal covariate shift__: the change in the _distribution_ of _network activations_ $h$, due to changes in _parameters_ during training.  

__Batch-Norm__'s idea consists in _normalizing_ the output of a layer, so that each dimension of the _output_ has __zero-mean__ and __unit variance__.

### Batch-Norm during training
Given the formulas:
![[train_time_form.png]]
During training, the batch norm:
- Input: a mini-batch of _activations_ $\{𝑎^{(𝑖)}| 𝑖 = 1, … , 𝐵\}$ where each sample has _dimension_ $D$.
- It has 2D _learnable parameters_ $𝜸_𝒋$ and $𝜷_𝒋$, for each dimension $D$ ($j = 1,...,D$).
- BN is comprised of 4 steps, all of them are _differentiable_ and can be _back-propagated through_: it guaranties that ==optimization does not “_undo_” _normalization_==.
- At training time, we also keep a _running average_ of _mean_ and _variance_ ($𝛽 = 0.1$ is usually called __BN momentum__).
- This is the running average:
$$
\begin{matrix}
𝜇_𝑗^{(𝑡)} = (1 − 𝛽) 𝜇_𝑗^{(𝑡−1)} + 𝛽𝜇_𝑗 \\
𝑣_𝑗^{(𝑡)} = (1 − 𝛽) 𝑣_𝑗^{(𝑡−1)} + 𝛽𝑣_j
\end{matrix}
$$
### Batch-Norm at test time
At test time, we do not want to depend on other items in the batch: we want the __output__ to _only depend_ on the __input__, ==deterministically==. 
- So, in this case, $u_j$ and $v_j$ are __constants__, namely the final values of running averages at training time. 
-  _BN becomes a deterministic linear transformation_. 
![[test_time_from.png]]

#### Pros & Cons of BN
- Pros:
	- high learning rates
	- training is not determininstic: act as regularization.
	- No overhead during test time: it can be fused with the previous layer
- Cons:
	- 0 explainability
	- does not scale well with micro batches
	- complex implementation (need to distinguish between training and testing)

#### BN for CNNs
In Batch norm for convolutional layers, we normalize along mini-batch and _spatial dimensions_ -> out activation has 4 dimensions:
![[batch_norm_cnn.png]]
It's this way so that elements of the same feature map are _normalized_ in the _same way_. 
