The effect of stacking more layers is that you reduce the training error.
Resnets are not more powerful, they have the same params, but are easier to train. 
As a byproduct of this easier training, you get better generalization for complex archictures. 

## Bottleneck residual block
Now that they had the ResBlock, they wanted to go deep by stacking multiple residual block. The problem is that everytime you add a new ResBlock, you also add 2 additional layer, which translates into an additionak: $36 * 𝐶^2 * 𝐻* 𝑊$ flops, and $18*C^2$ parameters.
![[bottleneck.png]]
Used with very deep resnets: more layers with approx same parameters and flops. It enables _faster increases in depth_ without altering _computational budget_.
The architecture is composed by:
- The 1st 1x1 conv, which transfroms a 4C input into a 1C output, so that the following 3x3 conv can handle it.
- A 3x3 conv, which has the same computational complexity as the one found in the  normal ResBlock. 
- The 2nd 1x1 conv, which brings the number of channel back to 4C so that the residual connection is possible.

__Bottleneck ResBlocks__ are faster and provide larger depth.

A good, default choice is ResNet-50. 

## Transfer Learning
We normally want to run CNNs on new classification datasets, not on ImageNet.

One of the most important _features_, from a practical point of view, of _learned representations_ is that they can be effectively _transferred to new datasets_.
__Transfer learning__ is the process of using and adapting a _pre-trained NN_ to new _datasets_. Usually, we pre-train on large datasets, and then we use it as frozen feature extractor or fine-tune it on the new dataset.
![[transfer_learning.png]]
![[transfer_detail.png]]

In particular:
- 2a. With Frozen CNNs, we perform network surgery, by _cutting the classification "head"_ of ou network and _putting a new one_ that has _the right number of output classes_. After that, the head is then _trained_. 
- 2b. With fine-tuning, we re-train both the new head and the feature extractor. However, we have an head that is untrained and a model that is trained. So, in order to _not skew_ our results, we have to:
	1. Start with _frozen feature extractor_ for a few epochs
	2. Use _smaller LR_ (learning rate) than the one used to train original architecture 
	3. _Progressive LRs_: use smaller learning rates when _training extractor_, and even freeze lower layers (since lower layers, like stage 4, usually are more speficic to the domains of the previously learned net).

When doing transfer-learning, you might wanna be aware of Batch-Norm layers, namely you want to freeze them even when fine-tuning, since they may change statistics (i.e. mean and variantion, standard deviation) from what you want to expect. 
You can gain high accuracy just by not unfreezing the BN layers. 

## Inception-ResNet-v2 - Mixed couple
Inception-v4 is basically a larger Inception-v3 with a more complicated stem. The authors also tried the residual connections idea around the Inception module.

## ResNeXt
Inception modules are effective _multibranch architectures_, which can be thought of as following a split-transform-merge paradigm. Yet, they are heuristically and carefully designed.
ResNeXt is a simpler way (in the spirit of VGG and ResNet) to realize _multiple pathways_. 
It _decomposes_ each __bottleneck residual block__ of ResNets into $𝐺$ _parallel branches_, i.e. the cardinality of the convolution. 

Once $𝐺$ is chosen, it is possible to obtain a $𝑮 × 𝒅$ ResNeXt block with complexity similar to the original ResNet block ($34*𝐶^2*𝐻*𝑊$) by equating the two FLOPs expressions and solving for $𝑑$.

>[!WARNING]
>Remember! Once I fix $G$, I can find $d$ -> $d$ is a byproduct of $G$.

![[res_next.png]]

This operation basically does the same as a _stack of convolutions_, and I'll show you why. 

#### Grouped convolutions – general case
![[grouped_convolution.png]]
In grouped convolution, we have a kernel that focuses only on determined channels of the input, while the other kernel focuses on the other channels. 
The result is that I obtain the same output (shapes, not really the same output though) with the same input, but:
- ==𝑮 times _less params_, 𝑮 times _less flops_.==

## Converting the multibranch
#### Part 1 - The final layers
![[ipcv_shittiest_formula.png]]
Since I have just a bunch of dot products and sums, I can express the operation of summing $y^{(1)}_k$ and $y^{(2)}_k$ as a 1x1 convolution. 
![[to_1b1_conv.png]]

#### Part 2 - Initial part
Or, how to convert Multiple 1x1 convolution as a single 1x1 convolution. 
![[p1.png]]
![[p2.png]]

Two 1x1 convolutions processing the same input tensor in parallel… can be realized as one 1x1 conv with twice the channels, after which we split the activations.

#### Part 3 - The middle part
Now, we have also to transform the middle part of the net as a conv layer:
![[p3.png]]
Finally, the central operation of the reformulated block is actually a _grouped convolution_ with 𝐺 groups.

#### In short
The multibranch design can be realized with a stack of convolutions. 

### ResNeXt results
The results show that you dont need many channels -> you can obtain a good representation just by doing a lot of different combinations of different subjects of 4 channels.

## Squeeze-and-Excitation Net (SENet)
Propose the __squeeze-and-excitation module__ to capture global context and use it to reweight channels in each block.
![[senet.png]]
##### What does SE do?
The SE-layer takes in input the ResNext output, and does the following things:
- It uses _global average pooling_ to squeeze the dimension of the input to 1x1 in the spatatial dimesnsion (__Squeeze__). 
- The FC and activations functions then _reduce_ and then _resize back_ the _channel dimesion_ (__Excitation__)
The output of the sigmoid (the excitation part) will then be used as a scaling factor for the ResNext output, in particular along the channel dimension. 
The SE-block output allows for a dialogue between channel in a sense. 

## MobileNets
People now shifted to make these things run on low power platforms.
In this context, grouped convolutions are exploited. 

### Depth-wise separable convolutions
Standard convolutions 
1. filter features based on the convolutional kernels and 
2. combine features in order to produce new representations. 
Very expressive but also very expensive. 

By separating filtering and combination, _depthwise separable convolutions_ aggressively limit computational cost. In particular, they use grouped convolution, with as many groups as input channels, so that each kernel works on a single channel basically. 
![[depth-wise.png]]
Also, this allows us to compute spatial reasoning fast, but no channel mixing. 

### Inverted Residual blocks
The bottleneck residual block of ResNet was introduced to scale up the model depth by increasing the number of layers per block while keeping the computation and number of parameters roughly constant. 

To this end, it uses a pair of 1x1 convs, where the _first compresses_ the number of channels, while the _second expands_ them. 

Hence, the “real” 3x3 convolution processing spatial information operates in the compressed domain. This may result in information loss going through ReLUs.

To avoid this potential information loss, MobileNetv2 proposes to use _inverted residual blocks_. 
![[inverted_res.png]]

In such blocks, the first 1x1 conv _expands the channels_, while the second compresses them back, according to an expansion ratio 𝑡.
- The opposite of normal bottleneck ResBlocks.

To limit the increase in computation , the inner 3x3 convolution is realized as a depthwise convolution.

The last difference is the removal of non-linearities between residual blocks: this is motivated by a theoretical study on the ability of ReLUs to preserve information in low-dimensional manifolds and experimentally verified.

### MobileNets
MobileNet-v2 is a stack of inverted residuals blocks.

The number of channels grows slowly compared to previous architectures: this is a benefit of blocks exchanging compressed activation and helps in keeping the complexity low. Low number of channels does not require heavy size reduction in the stem layer. Yet, the representation must be expanded before feeding it to the classifier. 

When a block downsamples the activation, it applies stride=2 to the inner 3x3 convolution (ResNet-D trick). 

Whenever spatial dimensions or number of channels do not match between input and output, there are no residual connections

### Wide-ResNet
ResNets allowed us to explore the advantages of depth by stacking residual blocks. But model scaling can be performed also by increasing width, i.e. number of channels at each layer.

To study its effect, a widening factor 𝑘 is introduced by Wide ResNets and the resulting model is usually referred to as WRN-𝒏-𝒌, e.g. WRN-50-2 has 50 layers, each of them has twice the channels it has in ResNet-50. 

Trading depth for width can give similar performance. What is the optimal way to scale a model?