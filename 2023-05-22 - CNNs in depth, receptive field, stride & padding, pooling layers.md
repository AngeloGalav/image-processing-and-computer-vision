#### Revision of the last lesson: some interesting stuff 
We have an idea of [[2023-05-18 - Intro to CNN#Equivariance (wrt to translation)|equivariance]], meaning that a picture of a cat remains a picture of a cat, no matter how it is oriented etc...
Equivariance allows us to see features in every possible orientation (for equivariance wrt to rotation).
However, as we know, correlation is only equivariant to _translation_, and not with respect to rotation or scale.

#### Activation
By slinding the filter over the image, we get a _single-valued output image_, which is the __output activation__. It's also called a __feature map__.
![[feature map.png]]
We can then apply a second filter, with different weights. 
![[second_filter.png]]

## Convolutional layer
![[convolutional_layer.png]]
If we have 4 filters, each of size 3 × 5 × 5, we can describe the overall operation realized by the layer as
![[overall.png]]
Notice how we have a _bias_ for _each filter_.
In the general case, we compute $𝑪_{𝒐𝒖𝒕}$ convolutions between vector-valued kernels and input activations.
- _The number of __output channels__ is equal to the __number of kernels___. 

##### Non-linearity for CNNs
We have seen that _convolutional layers_ can be interpreted as a constrained form of _linear layers_, [[2023-05-18 - Intro to CNN#Convolution as matrix multiplication|here]]. 
Hence, they follow the same rule: to meaningfully compose them, we need to _insert_ __non-linear activation__ __functions__ between them.

## Relationship between spatial dimensions
In general:
$𝐻_{𝑜𝑢𝑡} = 𝐻_{𝑖𝑛} − 𝐻_{𝑘} + 1$
$𝑊_{𝑜𝑢𝑡} = 𝑊_{𝑖𝑛} − 𝑊_{𝑘} + 1$
If we stack several convolution layers, _feature maps will shrink_ after each layer. The absence of padding is also referred to as `padding=“valid”` (meaning there's no padding).
![[zero_padding.png]]

## Zero Padding
Common “solution” is to add _zero padding_ around the _original image_.
In this way, we preserve the dimension of the input into the final image. 

$𝐻_{𝑜𝑢𝑡} = 𝐻_{𝑖𝑛} − 𝐻_{𝑘} + 1 + 2P$
$𝑊_{𝑜𝑢𝑡} = 𝑊_{𝑖𝑛} − 𝑊_{𝑘} + 1 + 2P$
Usually, we have that $P = \dfrac{H_k - 1}{2}$ to have _output of the same size of input_. 
![[0padding.png]]

##### Why using 0?
Usually, using 0 is as good as using anything else. 
For dense task, however, we can use better padding (i.e. repeating columns) which gives may give us better results. Usualy 0 is still good in general. 

### Receptive fields
The _input pixels_ affecting a _hidden unit_ are called its __receptive field__.
For instance, if we apply a $𝐻_𝐾 × 𝑊_𝐾$ kernel size at each layer, the _receptive field_ of an element in the $𝐿$-th activation has _size_:
$$𝑟_𝐿 = [1 + 𝐿 (𝐻_𝐾 − 1)] × [1 + 𝐿 (𝑊_𝐾 − 1)]$$
As shown in the formula, each size _grows __linearly___ with the number of layers $𝐿$ → to compute _attributes_ corresponding to a large portion of the input, we would need _too many layers_.

To obtain _larger receptive fields_ with a _limited number of layers_, we _down-sample_ the activations inside the network.
This involved __striding__, meaning we skip some rows and columns in each layer.

## Receptive field with stride (strided convolutions)
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
##### Receptive field with stride S=2
For the _very last layer_, _nothing changes_ (thats why we have $l-1$), meaning we dont have the stride in it. 
![[cnn1.png]]
We need strides so that the receptive field can be big enough, without having to add too many layers. 

##### An example
![[conv_example.png]]
The number of learnable parameters for the convolution layer on the left is (+1 is _for biases_):
$$𝟏𝟔 × (𝟖 × 𝟓 × 𝟓 + 𝟏) = 𝟏𝟔 × 𝟐𝟎𝟏 = 𝟑, 𝟐𝟏𝟔$$
In general, a generic convolution layer whose shape is $𝑪_{𝒐𝒖𝒕} × 𝑪_{𝒊𝒏} × 𝑯_𝑲 × 𝑾_𝑲$ has a number of _learnable parameters_ of $𝑪_{𝒐𝒖𝒕} (𝑪_{𝒊𝒏} \cdot 𝑯_𝑲 \cdot 𝑾_𝑲 + 𝟏)$.
- $C_{out}$, as we know, is also the number of _kernels_ applied to the layer. 

The size of the output for this example is $𝑯_{𝒐𝒖𝒕} = 𝑾_{𝒐𝒖𝒕} = 𝟑𝟐 − 𝟓 + 𝟐 ∗ 𝟐 + 𝟏 = 𝟑𝟐$.
Hence, there are 𝟏𝟔 × 𝟑𝟐 × 𝟑𝟐 = 𝟏𝟔, 𝟑𝟖𝟒 values in the output activation, i.e. about 64 KB. In general, there are $𝑪_{𝒐𝒖𝒕} 𝑯_{𝒐𝒖𝒕} 𝑾_{𝒐𝒖𝒕}$ values in the output activation.

To compute each of the output values we take the dot product between $200 = 𝟖 × 𝟓 × 𝟓$ weights and the corresponding input values, which requires $𝟐𝟎𝟎$ multiplications and $𝟐𝟎𝟎$ additions for inputs of size $𝑛$, i.e., 𝟐 ∗ 200 floating-point operations (flops). 
If the hardware supports _Multiply-accumulate operations_ (MACs) that can be _performed in one cycle_, it is common to express computational complexity in terms of them, by dropping the factor 𝟐.

Hence, the total number of flops for this convolution is $𝟏𝟔, 𝟑𝟖𝟒 × (𝟖 × 𝟓 × 𝟓) × 𝟐 ≅ 𝟔. 𝟓𝐌$, while the total number of MACs is ≅ 𝟑. 𝟐𝟓𝐌. 
In general, the number of flops is $𝟐 (𝑪_{𝒐𝒖𝒕} 𝑯_{𝒐𝒖𝒕} 𝑾_{𝒐𝒖𝒕}) (𝑪_{𝒊𝒏}𝑯_𝑲𝑾_K)$.

###### Other common convolutions
- 1D convolutions
- 3D convolutions are usually used in videos.

## Other layers in CNNs
Neural networks based on convolutional layers are called Convolutional Neural Networks. Besides convolutional layers, they are composition of:
- _non-linear activation functions_ 
- _fully connected layers_ 
- __pooling__ layers 
- __(batch-)normalization layers__

### Pooling layers
__Pooling layers__ aggregates several values into one output value with a _pre-specified_ (i.e. not learned) _kernel_. 
==Key difference with convolution==: each _input channel is aggregated independently_, i.e. the kernel works only along the spatial dimensions, and $𝐶_{𝑜𝑢𝑡} = 𝐶_{in}$.

Possible Hyper-parameters of a pooling are:
- Kernel width and height $𝑾_𝑲 × 𝑯_𝑲$ 
- Kernel _function_ (pre-specified): max, avg, … 
- _Stride_ $𝑺$ (usually >1) 
A common choice of hp is: 2 × 2, 𝑆 = 2, kernel func= max

Some famous pooling layers are:
- _Avg-pooling_
	- Avg-pooling makes no sense to use since we can just use a kernel that does the same thing.
- __Max pooling__ 
	- Max-pooling is the only pooling that makes sense to use, since it would require mre than a kernel.  

Since pooling is an hyperparameter, it can downsample, but it doesn't have to. 

##### Max-pooling
Max-pooling is an apriori decision: since it is not learned (and thus it is more effiencient to use) i dont really know if our model _should_ really use this operation. 

This is how Max-pooling would be implemented if we had to use kernels:
![[max_pooling.png]]

__Downsampling comes from stride__, _not from pooling_: we can achieve it with convolutions as well. Yet, compared to strided convolutions, max pooling 
- has no learnable parameters (pro and con) 
- provides _invariance_ to small spatial shifts.

### A tensor is flattened at the end
![[cnn_flatten.png]]
I flatten because at a certain point a need a global vision of the image (a global receptive field), and the only way to do that is by using a _linear, fully connected layer_.

>[!WARNING]
>__Remeber!!__: we can 
16 kernels, 5x5x6 

### Example: LeNET5
![[example_LENet.png]]