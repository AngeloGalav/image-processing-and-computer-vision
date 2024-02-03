## Bag of Words representation
As we've discussed, the BoW representation discards the structure of the text input, and focuses exclusively on the keywords present in the text. 
In CV, a similar technique is used, in which you only focus on the _features_ of your images, while discarding the _spatial structure_ of them. 

#### Images as frequency of visual words
When using BoW in NLP, we want to have frequency associated to the words present in the text. So, the BoW is in a sense a histogram. But how can we find the (equivalent of) codewords in our image?   
![[Images/Bow.png]]
In other words, how can we develop a list of feature and use it to measure the frequency?
Also to note: in NLP words are very much explicative, while features in images are not. 

## 1. Extract visual words
In the BoF (Bag of Features), the features are _non-overlapping __patches___ extracted from the image.  In particular, words detection uses either:
- use a _regular grid_ of _positions at several scales_ (as seen in the upper-right part of the image) or 
- run an _interest points_ (keypoints) _detector_ (Harris, DoG, Harris affine, MSER, …) 

Then either 
- use the _patch_ around the _keypoint_ as “_word_” 
- compute its __SIFT descriptor__ to be used as “_word_”
- compute some _color descriptor_, e.g. mean and variance of subpatches in R, G and B channels (also because SIFT descriptors discard colors)
![[visual_words.png]]
Now that the we have the tokens and the visual words, we need to compute the codebook.  

## 2. Compute the codebook
_Clustering_ of extracted patches or descriptors in a finite set of centroids creates the codebook. Usually simple _k-means_ clustering is used.
- This is happening in the space of SIFT descriptors.  ![[sift_space.png]]
To each cluster, it will assign a _centroid_ (according to the k-means algorithm). 
The centroid is then used as the _word_. 
By varying $k$, the word also varies. 

If we used CNNs, we could have also used the color of the patches as information. 

## 3. Encode training images and new images
Given an image, the _same extraction pipeline_ used to create the _codebook_ (e.g. regular grid + SIFT description of patches) is applied to it. Each extracted descriptor/patch is _matched_ against the codebook and assigned to the _nearest centroid_ in the codebook. 
In vanilla BoVW, the histogram entry corresponding to such codeword is _incremented_.

The BOW representation, for a long time, has been 
![[phase_3.png]]

## 4. Train a classifier
The classifier does not work anymore with the _image pixels_ directly, but with the _histogram of codeword frequencies_. 
Note that we obtain a fixed-size representation out of varying input sizes. We also need to appropriately __normalize__ the _histogram_ to make the representation _invariant_ with respect to the _input size_ (of different inputs). 
![[train_classifier.png]]

## VLAD: vector of locally aggregated descriptors
Successful evolutions of BoW focused on making the assignment to a codeword more informative. If descriptors are $𝐷$ dimensional (e.g. 128 when using SIFT), and there are $𝐾$ codewords, VLAD creates a $𝐷 × 𝐾$ descriptor which _stores the differences_ of input words with respect to the codewords, i.e their relative position with respect to the codeword.
![[vlad.png]]
In particular, the position of the patch is measured wrt to the code word in the patch/SIFT descriptor space. This differences are then taken into account inside the data structure. 

As we can see, we don't have histograms anymore since we can have negative values now (i.e. if a codeword is on the left or the right of a centroid in the SIFT space).

#### BOVW was the dominant paradigm until 2012
ImageNet initially was made to be very easy to use with BoW approach -> they made a vocalubary of 1000 codewords running k-means on 1 million SIFT descriptors. 

That is, until DL came to be. 

In BoW, the representation is fixed, and there's learning on the hyperparameters of the SVM. 
That is opposed to what happens in Deep learning. 

DL methods are now mostly used to modify the _representation of images_ in order to make feature extraction easier.

## Neural Networks
![[linclass.png]]
It is not so different from the standard representation. 

### Activation function
![[linclass2.png]]

#### Gradients of activation functions
Gradients play a central role in optimization. Gradient of the output of activation functions with respect to input is very different between sigmoid and ReLU, easier to train (deep) networks when using ReLU. Yet, ReLu can give rise to “_dead_” neurons, if for all inputs _the output is negative_.
![[linclass3.png]]

In the past, a lot of care was put in the initialization of the tensor, to that the CNN would start on the linear part of the Sigmoid (the center part more or less). 

In fact, let's put it this way: if the gradient is very small (i.e. in the range of -3 in the picture), it takes a lot of time during training to reach the part that was actually associated with the right prediction. 

ReLU is superior to the sigmoid function in this sense -> if you are on the right, _you can quickly change output_ if there's a _wrong_ output, ==but this is not true to the left!!== (in this case, you completely kill the output). However... :
- Some neurons may die, they will never recover from producing 0. 

### Activation functions – Leaky ReLU
To avoid dead neurons, a small response can be produced also for negative inputs. The simplest variant is called Leaky ReLU, which has a small but _non-zero_ constant gradient also for negative inputs.
So, even if you're still dampening a lot, there's still a possibility for recovery. 
![[leaky_relu.png]]
However, sometimes changing activation functions is not enough to improve our results. 

#### Why do we insert activation functions?
Without activation functions, _we end-up again with a linear classifier_.

#### Is ReLU enough?
I'm computing new coordinates for my points according to where they stand with the hyperplanes of the W. 
However, if did not have a line that separated the points in the first case, I won't have such things also in the second space. 
- -> A linear or affine mapping _does not change linear separability_.

With ReLU, points are __collapsed__ on the two axis. In this way, we can easily _separate points_. 
- It is _highly non-linear_.
![[relu.png]]
### Graphical representation & terminology
Given the formula: 
$$
𝑓 (𝒙; 𝜃) = 𝑊_2𝒉 + 𝒃_𝟐 = 𝑊_2𝜙(𝑊_1𝒙 + 𝒃_𝟏) + 𝒃_𝟐 = s$$
- $𝑥$ is the _input tensor_ 
- $ℎ, 𝑠$ are ___activations___ 
- $𝑊_𝑖 , 𝑏_𝑖$ (and other numbers we may use to go from one activation to another one) are _parameters_ 
- Every layer is often called a fully-connected layer, as every element of the input influences every element of the output. A neural network with 2 or more layers is also called a __Multi-Layer Perceptron (MLP)__.

h,b,W are all associated to a specific layer.

### Deep Neural Network
L layers in the network, then the network is consider to be deep.
_Number of layers_ is the _depth of the network_. 
- If 𝐿 > 2, we consider the network to be “deep”. 

#### Width of neural networks
The number of _dimensions computed by each layer_, i.e. the __dimensionality__ of $ℎ_𝑖$ , is the _width_ of the network. 
- Usually it is the maximum of the width. 

### Why “neural” networks?
Better to think it as matrices than neurons:
![[real_dnn.png]]
$b$ is a threshold in a sense.
What we've just seen corresponds to this:
![[DNN_formula.png]]
Which is basically this in a figurative sense:
![[Linear_percep.png]]

#### MLP as universal approximator
Like 𝑘-NN classifiers, neural networks enjoy a form of _universal approximation property_: neural networks with a single hidden layer can be used to approximate any continuous function to any desired precision. For instance, they can solve XOR.

### Depth as an exponential gain
If one hidden layer is enough, why using more than one? For instance, XOR can be seen as the special case with only two inputs of a function called parity.

In the worst case, which happens for the parity function, the number of hidden units grows exponentially with the input dimensions (it requires one hidden unit corresponding to each input configuration that needs to be distinguished).

If more hidden layers are used, instead, a _linear_ number of neurons wrt the input size can learn parity. Well, maybe…

## Convolutions - Vertical edge detection with FC layer
In traditional image processing and computer vision, we usually rely on _convolution/correlation_ with hand-crafted filters (kernels) to _process images_ (e.g. denoise or detect local features). 
- Unlike linear layers, in a convolution, the input and output are _not flattened_, i.e. <u>convolutions preserves the spatial structure of images</u>. 
- Unlike linear layers, a convolution _processes_ only a – small – _set of neighboring pixels_ at each location. In other words, each output unit is connected only to _local input units_. This realizes a so called __local receptive field__. 
- Unlike linear layers, _the parameters_ associated with the connections between an output unit and its input neighbors _are the same for all output units_. Thus, _parameters are said to be shared_ and the convolution seamlessly learns the same detector, regardless of the input position. Convolutions embody __inductive biases__ dealing with the structure of images: images exhibit _informative local patterns_ that may appear everywhere across an image.

###### mini rant on edge detection using CNNs
Since our convolution only needs to learns only 2 parameters in the best case in the case of edge detection (which are [-1, 1]), and allows us to skip many computation. 
Does this mean that CNNs are more powerful than strandard NNs? _NO_! Rather, _they are less powerful_ since they only work well with a specific type of data.  

### Correlation or convolution?
In standard convolutions, we would _flip the kernel_. Nobody does this, so what we're actually applying a cross-correlation, however we still call it convolution for some reason.

## Convolution as matrix multiplication
Convolution can be interpreted as _matrix multiplication_, if we reshape inputs and outputs.
The resulting matrix is __still a linear operator__, which: 
1. shares parameters across its rows 
2. is sparse, i.e. each output unit is connected only to a small set of neighboring input entries 
3. seamlessly adapts to _varying input sizes_ 
4. is equivariant to translations of the input, i.e. translation of the output of the convolution is equivalent to computation of the convolution on the translated input
![[convs_as_sparse.png]]

## Equivariance (wrt to translation)
__Equivariance__ with respect to translation means that we can swap _translation_ and _correlation_ and get the same result:
$$𝑇(𝑥) ⋆ 𝐾 = 𝑇(𝑥 ⋆ 𝐾)$$
It is another form of __inductive bias__ that improves data efficiency with respect to linear layers thanks to weight sharing: we do not need to see _features_ (e.g. edges) _at all locations_ in the _training dataset_ to be able to _learn to detect_ them effectively. 
Note that correlation is ==__not equivariant__ with respect to rotation or scale==.

## Multiple input channels
Images have 3 channels, so convolution kernels will be a _3-dimensional tensors_ of size $3 × 𝐻_𝐾 × 𝑊_𝐾$ and
![[multiple_channels.png]]
This is still a _2D convolution_ however, since we slide the kernel in these 2 dimensions only: the thing is, we are working with 3d data structures. 
- The bias, because of equivariance, is the same in all transformations.  
- Filters and input depth always match, and the 3rd dimension of a filter is usually not implicit. 

In the picture, we have a 5x5 conv., but it has 75 params (5x5x3).
