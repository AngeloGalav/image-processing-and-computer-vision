
## VGG: Deep but regular

### Stages
VGG introduces the idea of designing a network as repetitions of __stages__, i.e. a fixed combination of layers that process activations at the same spatial resolution. In VGG, stages are either: 
- conv-conv-pool 
- conv-conv-conv-pool 
- conv-conv-conv-conv-pool

One stage has same receptive field of larger convolutions but requires less params and computation and introduces more non-linearities. 
- No free-lunch, though: _memory for activations __doubles___.
![[stages_vgg.png]]

In stage, you are processing activation with the same spatial resolution. 
- repetition of the same arrengement of layers.

VGG-16 is way,wayy bigger than AlexNet. It has more than double the number of the params. 
- The cost within our stages stays constants (when we decrease the dimensionality, we double the number of channel). This is true up to the last stage. 
- Activation cost way more than parameters to store them in memory (as always, we pay a price in terms of memory). 
	- As usual, the layers that are most responsible for the memory consuption are the first. 

- Very large activation, and absence of stem layers cause a high processing power requirement. 

## Inception v1 (GoogLeNet)
Google needed a model that was not only precise, but also quick and efficient. 
- "We dont want to understand, but to tune every hyperparamenter we have to be efficient and effective". 
![[googlenet.png]]

### Stem layers
Stem layers aggressively downsamples inputs:

### Inception modules
![[inception_modules.png]]
Two main problems: 
- Due to max-pool, $\#channels$ _grows very fast_ when inception modules are stacked on top of each other 
	- In order to be stacked, they need the same spatial dimension. So layers are padded. 
- 5x5 and 3x3 convs on many channels become prohibitively expensive if we stack a lot of them, 
	- e.g. here conv 5x5 = 2x28x28x32x192x5x5=240 Mflops, conv 3x3 = 2x28x28x128x192x3x3=350 Mflops
	- with max-pooling, We do not have control in the number of channels in output 

### 1x1 convolutions
A 1 x 1 convolution is actuallually a 1x1x128 (aka the input channels) convolution  
- If I want 3 output channel, as we know, we need 3 kernels. 

1x1 convolution are very fast and effective. 
- They are still mixing content (channels) just _not spatially_. 

1 x 1 convolutions behaves like all other convolutions. They allows us to change (usually shrink) the depth of the activations while preserving the spatial size. We can also interpret them as applying a linear FC layer at each spatial location.
![[1x1-conv.png]]

If we stack several of them we are actually applying an __MLP__ (multi-layer perceptron) at each location.
- A MLP can approximate every function!!!

### Real Inception module
The problem of the naive inception module was that there were too many channels (so too many operation to do).
- After max-pooling they insert 1x1 convs, in order to _control the number of channels_ of max-pooling. 
- They placed 1x1 convs before 5x5 convs in order to shrink the number of channels again. 
![[real_inception.png]]
By adding 1x1 convolutions before larger convs and after max pool we can 
- Control the number of output channels by reducing the depth of the max pool output 
- Control the time complexity of the larger convolutions by reducing the channel dimension, e.g. now flops are
A lot of gain in terms of FLOPS! 

### Getting rid of the fully-connected layer at the end (Fully-connected classifier vs global average pooling)

![[w_agv.png]]

To reduce the number of parameters needed at the interface between convolutional features and fully connected layers, NiN proposed to get rid of spatial dimensions by averaging them out. 
- coarse 

[..]

### 
With Inception, we are giving GD all the options. 
- It's like an Ensemble of very many VGG layers. 

#### Some nice question
Why dont we use 1x1 convs only?
- 1x1 convolution dont allow to mix information spatially alone. We need other layer to 
	- They work extremely well when combined with other 


## Inception v3
It leverages convolution factorizations to increase computational efficiency and to reduce the number of parameters. Less parameters should be more disentangled and therefore easier to train. Achieved 4.48% top-5 error on ILSVRC12 with 12 crops.

## Residual Networks
VGG lesson: growing depth improves performance. Yet, stacking more layers doesn’t automatically improve performance. Too many parameters increase overfitting and hurts generalization? We also observe higher training errors, so overfitting it’s not the only reason, there is also a training problem, even when using Batch Norm. Yet, a solution exists by construction: if a network with 20 layers achieves performance X, then we can stack 36 more identity layers and we should keep performance at X. SGD is not able to find this solution with the parameterization we use for layers: optimizing very deep networks is hard.
![[residual_net_graph.png]]

### Residual block
The proposed solution is to change the network so that learning identity functions is easy by introducing residual blocks. Implemented by adding _skip connections_ skipping two convolutional layers.
![[res_block.png]]

Instead of $H(x)$, we know have $F(x) + x$. 
Weights usually initialized to be very small (or 0 for biases). Network starts with the identity function and learns an “optimal” perturbation of it. It makes heavy use of batch-norm.

### Residual Networks arch
Inspired by _VGG regular design_.
Network is a stack of _stages_ with fixed design rules: 
- Stages are a stack of residual blocks.
- Each residual block is a stack of two 3x3 convolutions with batch-norm. 
- the first block of each stage halves the spatial resolution (with stride-2 convs) and doubles the number of channels.
 ![[res_net_arch.png]]
It uses _stem layer_ and _global average pooling_ as GoogleLeNet.
Naming conventions follow VGG, as well: ResNet-X, where X is the _number of layers_ with learnable parameters.

### Skip conn dimensions
The residual blocks described so far cannot be used as the first block of a new stage, because the number of channels and the spatial dimensions do not match along the residual connection (dashed arrow).
The authors tried two solutions: 
- apply stride 2 to 𝑦 and zero pad the missing channels (no extra parameters) 
- 1x1 conv with stride 2 and 2C output channels (solid arrow)
	- Halve the spatial dimension, double the number of channel
and verified the second one to perform slightly better.
![[skip-conn.png]]

#### And the result is... OMG
![[alexnet_moment.png]]
Residual blocks allow us to train deep networks. When properly trained, deep networks outperform shallower network as expected Won all 2015 competitions by a large margin, still the standard baseline/backbone for most tasks today.