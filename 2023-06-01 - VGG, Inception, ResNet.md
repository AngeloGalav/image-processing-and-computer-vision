## VGG: Deep but regular
![[architecture.png]]
(in this picure, each column with a net represents a variant, i.e. VGG-16 is column D)

Commit to explore the effectiveness of _simple design choices_, by allowing only the combination of : 
- 3x3 convolutions, S=1, P=1 
- 2x2 max-pooling, S=2, P=0 
	- $n_{channels}$ _doubles_ after each pool (aside from the last layer, otherwise there would be too many channels, like 1024!)

- Dropped local response normalization (LRN) 
- Batch norm not invented yet! __Pre-initialization__ of deeper networks with weights from shallower architectures crucial to let training progress (unless smart initialization strategies are used).

>[!WARNING]
>Receptive field can give us an idea of why 3x3 convolutions are used. If we stack 2 layers with 3x3 kernels each, the receptive field in the input layer will have size 5x5. 
>![[receptive_field_recap.png]]


### Stages
VGG introduces the idea of designing a network as repetitions of __stages__, i.e. a _fixed combination of layers_ that process activations _at the same spatial resolution_. 
In VGG, stages are either: 
- conv-conv-pool 
- conv-conv-conv-pool 
- conv-conv-conv-conv-pool
![[vgg-16.png]]

==One stage has _same receptive field_ of _larger convolutions_ but requires _less params_ and computation and introduces more non-linearities==. 
- No free-lunch, though: _memory for activations __doubles___, since you have to also store the activations in the middle of the convolutions. 
![[stages_vgg.png]]

#### VGG-16 summary
__Bigger__ network than _AlexNet_
- _138 M params_ (2.3x AlexNet) again, _mostly in fc layers_.
- _~𝟒 Tflops_ (~ 14x), 31 Gflops/img, mainly due to convolutions 
- ~𝟏𝟔. 𝟓 GB of memory (~ 12x) 
	- _Memory mostly stores activations_ (of first conv layers, which are much larger than in AlexNet for the __absence of stem layers__) rather than parameters. 
	 - This is also due to the fact that there are __no stem layers__, inputs are processed in the same resolution as they come to. 

## Inception v1 (GoogLeNet)
Google needed a model that was not only precise, but also _quick and efficient_. 
- "We dont want to understand, but to tune every hyperparamenter we have to be efficient and effective". 
![[googlenet.png]]
Some things to note:
- Does not uses multiple layers of dense layers, but only one.
- it is used today. 

### Stem layers
__Stem layers__ aggressively _downsamples inputs_, yet, it brings it down a bit _more gently_ than AlexNet, learning from per ZFNet.
- only uses _strides of 2_ 
- _largest conv_ is 7x7
	- 3 convs to make it 28x28

>[!Fun fact]
>Stem layers were "retrofitted" into AlexNet: they were initially invented by Inception, but were also used in AlexNet, even though at the time they did not have a name. 

### Inception modules
The body is a _stack_ of _inception modules_.
#### Naive inception modules
![[inception_modules.png]]
It's made of 4 layers in _parallel_.
The result of each of this layer are _stacked_ along the _channel dimension_. 
In order for them to be stackable, they have to have the _same spatial dimesion_, so the _stride_ and _padding_ of each layer are fixed. 

Two main problems: 
- Due to _max-pool_, $\#channels$ _grows very fast_ when inception modules are stacked on top of each other 
	- max-pooling layer has the same number of channels for input and output (we do not have control in the number of channels in output )
- 5x5 and 3x3 convs on many channels become prohibitively _expensive_ if we stack _a lot of them_, 
	- e.g. here conv 5x5 = 2x28x28x32x192x5x5=240 Mflops, conv 3x3 = 2x28x28x128x192x3x3=350 Mflops

### 1x1 convolutions
A 1x1 convolution is actuallually a 1x1x128 (aka the input channels) convolution  
1x1 convolution are very _fast_ and _effective_. 
- No spatial communication between pixels, but they're are still mixing content (channels) just _not spatially_. 

1x1 convolutions behaves like all other convolutions. They allows us to change (usually shrink) the depth of the activations while preserving the spatial size.
![[1x1-conv.png]]

We could see 1x1 convolutions as _linear layers_ applied to one specific location, and which slides on the image. 
If we _stack_ several of them we are actually applying an __MLP__ (multi-layer perceptron) at each location.
- A MLP can approximate every function!!!

### Real Inception module
The problem of the naive inception module was that there were _too many channels_ (so too many operations to do).
- After max-pooling they insert 1x1 convs, in order to _control the number of channels_ of max-pooling. 
- Also, they placed 1x1 convs before 5x5 (and 3x3) convs to shrink the number of channels and limit the computational cost. 
![[real_inception.png]]
By adding 1x1 convolutions before larger convs and after max pool we can 
- Control the _number of output channels_ by reducing the depth of the max pool output 
- Control _the time complexity_ of the larger convolutions by reducing the channel dimension, e.g. now flops are
A lot of gain in terms of FLOPS! 
(i.e. from 240M flops to 25M flops in 5x5 convs)
==In short, 1x1 convolutions are the cheapest way to _decrease the number of channels_.== 

### Getting rid of some fully-connected layers at the end (Fully-connected classifier vs global average pooling)

![[w_agv.png]]

To reduce the number of parameters needed at the interface between convolutional features and fully connected layers, they proposed to get _rid of determined spatial dimensions_ by ___averaging__ them out_ (the specific spatial dimenions). 

#### Global average pooling
- GoogLeNet uses global average pooling to remove spatial dimensions and one FC layer to produce class scores.
- This results in only 1 million parameteres and a negligible number of flop. 

If the _kernel size of pooling_ covering the _full input activation_ is computed by the layer instead of being specified by the user, i.e. AdaptiveAvgPool2d layer instead of AvgPool2d in PyTorch , this makes the network able (at least as far as tensor dimensions are concerned) to work on _any input image size_.

##### Final stats
Only _7M parameters_, _390 Gflops_ (3 Gflops/img), and 3GB of memory.

>[!WARNING]
>We cant just use 1x1 convolutions only, we need also to mix the spatial features of a specific layer. 

## Residual Networks
VGG lesson: growing depth improves performance. Yet, stacking more layers _doesn’t automatically improve performance_. 

Too many parameters increase overfitting and hurts generalization. We also observe _higher training errors_, so overfitting it’s not the only reason, there is also a training problem, even when using Batch Norm. 

Yet, a solution exists by construction: if a network with 20 layers achieves performance X, then we can stack 36 more _identity layers_ and we should keep performance at X. 

_SGD is not able to find this solution_ with the parameterization we use for layers: optimizing very deep networks is hard.
![[residual_net_graph.png]]

### Residual block
The proposed solution is to change the network so that _learning __identity__ functions_ is _easy_ by introducing __residual blocks__. Implemented by adding _skip connections_ skipping two convolutional layers.
![[res_block.png]]

Instead of $H(x)$, we know have $F(x) + x$. 
Weights usually initialized to be _very small_ (or 0 for biases). Network starts with the identity function and learns an “optimal” perturbation of it. It makes heavy use of batch-norm.

### Residual Networks arch
Inspired by _VGG regular design_.
Network is a stack of _stages_ with fixed design rules: 
- Stages are a stack of residual blocks.
- Each residual block is a stack of _two 3x3 convolutions_ with __batch-norm__. 
- the first block of each stage halves the spatial resolution (with stride-2 convs) and doubles the number of channels.
 ![[res_net_arch.png]]
It uses _stem layer_ and _global average pooling_ as GoogleLeNet.
Naming conventions follow VGG, as well: ResNet-X, where X is the _number of layers_ with learnable parameters.

First layers are stem layers, as in GoogLeNet. However, it only uses one conv+pool layer, and _reduces only to 56 × 56_, probably because residual blocks are lightweight compared to Inception modules.
- The stem layers are more gentle. 

### Skip conn dimensions
The residual blocks described so far cannot be used as the first block of a new stage, because the _number of channels_ and _the spatial dimensions_ do not match _along the residual connection_ (dashed arrow).
The authors tried two solutions: 
- apply stride 2 to 𝑦 and zero pad the missing channels (no extra parameters) 
- 1x1 conv with stride 2 and 2C output channels (solid arrow, this is the one which was chosen) 
	- _Halve the spatial dimension, double the number of channel_.
and verified the second one to perform slightly better.
![[skip-conn.png]]

#### And the result is... OMG
![[alexnet_moment.png]]
Residual blocks allow us to train deep networks. When properly trained, deep networks outperform shallower network as expected. Won all 2015 competitions by a large margin, it's still the standard baseline/backbone for most tasks today.

>[!FUN FACT]
>After their discovery, _residual connections_ are now everywhere, even in transformers. 

