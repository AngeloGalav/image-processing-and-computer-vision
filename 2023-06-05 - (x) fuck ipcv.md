The effect of stacking more layers is that you reduce the training error.
Resnets are not more powerful, they have the same params, but are easier to train. 
As a byproduct of this easier training, you get better generalization for complex archictures. 

## Bottleneck residual block
We needed a new residual block with the same flops/computational layers, but with more layers (2 in particular). 
![[bottleneck.png]]
It costs exactly the same.
- The 1st 1x1 conv ... 
- The 2nd 1x1 conv brings the number of channel back to 4C.

There are many ResNet models, and you choose one depending on your needs. 
However, these different models allow us to see that we can use 1x1 convolution to introduce depth.
- That is because, while the res-block number are the same, you have a big number of parameters anyway.  

A good, default choice is ResNet-50. 

## Transfer Learning
We normally want to run CNNs on new classification datasets, not on ImageNet.

One of the most important features, from a practical point of view, of learned representations is that they can be effectively _transferred to new datasets_.
__Transfer learning__ is the process of using and adapting a _pre-trained NN_ to new _datasets_. Usually, we pre-train on large datasets, and then we use it as frozen feature extractor or fine-tune it on the new dataset.
![[transfer_learning.png]]
![[transfer_detail.png]]

In particular:
- 2a.
- 2b. [...] When fine tuning:
	- 1. Start with frozen feature extractor 2. Use smaller LR than the one used to train original architecture 3. Progressive LRs: use smaller learning rates when training extractor, and even freeze lower layers

When doing transfer-learning, you might wanna be aware of Batch-Norm layers. Since they will also update the weights on mean and standard accomulating. 
You may change this statistic from what you want to expect. 
You can gain high accuracy just by not unfreezing the BN layers. 

## Inception-ResNet-v2 - Mixed couple
Inception-v4 is basically a larger Inception-v3 with a more complicated stem. The authors also tried the residual connections idea around the Inception module.

## ResNeXt
Inception modules are effective multibranch architectures, which can be thought of as following a split-transform-merge paradigm. Yet, they are heuristically and carefully designed.
ResNeXt is a simpler way (in the spirit of VGG and ResNet) to realize multiple pathways. It decomposes each bottleneck residual block of ResNets into 𝐺 parallel branches, i.e. the cardinality of the convolution. 

Once 𝐺 is chosen, it is possible to obtain a 𝑮 × 𝒅 ResNeXt block with complexity similar to the original ResNet block (34𝐶 2𝐻𝑊) by equating the two FLOPs expressions and solving for 𝑑.
![[res_next.png]]

## Grouped convolutions – general case
![[grouped_convolution.png]]
![[ipcv_shittiest_formula.png]]

[...]
## ResNeXt block ≅ regular Inception-ResNet block
