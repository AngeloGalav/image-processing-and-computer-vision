## Homographies
Homographies are uselful for a lot of stuff in the field of computer vision. In particular: 
1. we can _relate_ _any pair of images_ of the planar scene through an _homography_. 
![[homography1.png]]
2. Any two images taken by a camera _rotating about its optical center_ are related by a _homography_.
![[homography2.png]]
3. Any two images taken by _different cameras_ (i.e. different $𝐴$) in a _fixed pose_ (i.e. same CRF) are related by a _homography_.
	- Note: since they are in the same CRF, they are related by focal length. 
![[homography3.png]]
#### Example: Rectification Homographies for stereo
![[rectified.png]]

---
# Image Classification
In image calssification, we have a set of predifined categories, and we have to choose among this categories when viewing a picture. 

There are some challenges when classifying images:
- Intraclass variations
- Background clutter
- Occlusions
- Viewpoint variations
- Illumination changes
- General weirdness of the world…

#### Some notable stuff
- RGB images are _tensors_ in a computer. The 3rd dimension is given by the colour channels. 
![[image_tensor.png]]
- Categories (a.k.a. our output) of samples are expressed as numbers.
i.e.:
0 -> Dog
1 -> Cat
2 -> Bird
3 -> Frog
4 -> Person

#### Traditional CV vs. Current CV
Traditional Computer Vision techniques, e.g. handcrafted rules based on edges, need a _controlled environment_, usually feasible in industrial vision applications, otherwise they are very brittle.
- ... so, (Supervised) Machine learning to the rescue comes to the rescue!!

##### Data driven approach
This new paradigm of machine learning (data-driven approaches) changes the relative importance of data and algorithms. 
- Data and datasets are crucial in this new paradigm.
To know the most used ones for each vision task and to understand the impact of metrics on the results is therefore at least as important as it is to know the latest and greatest algorithms.

## Our first classifier: $𝑘$ nearest neighbor (kNN)
![[training_testing.png]]

### kNN w/ majority vote
- "Training” (No “learning“, only “memorization")
	1. _Store_ all the training examples and labels 
- Testing 
	1. Compute _distance in feature space_ between _test sample_ and _all the training examples_. 
	2. Retrieve _labels_ for the _closest $𝑘$ training examples_. 
	3. Aggregate them (e.g. _majority vote_) to define the predicted label for the test sample.

###### Characteristics
- No “learning“, only “memorization".
- Main drawback: kNN is a “lazy” algorithm, _all computations postponed to test time_.
- kNN is only as good as the features (image) and the distance in feature space are.
- kNN is a non-parametric instance-based algorithm, _it can model complex data distributions_.
	- It can model any function!

##### Why does it work so well?
kNN does not make any assumption on the shape of the discrimination function, so it can learn any strangely shaped function.  
- It computes the Voronoi diagram (or tessellation) first, and from this it computes a decision boundary. 
![[voronoi.png]]

### Choosing parameters 
Parameter 𝑘 or the distance function are examples of _hyper-parameters_ of a machine learning model: choices about our model that we do not learn from data, and we manually set before training starts.

Why can’t we learn them from data? They are either: 
- Too hard to learn from data. 
- Not appropriate to learn, because the “best” value you can compute on the training set is likely to perform poorly on the test set. 

What is the accuracy on the training data of the classifier on the right, with $𝑘 = 1$?

### Model capacity
By changing $𝑘$, we change the _model capacity_ of the classifier.
Informally, we can think of the model capacity as the measure of _how complex/flexible the functions_ that can be learned are. $𝑘 = 1$ gives the model with the largest capacity, which can perfectly classify any (training) dataset. 
By _growing_ $𝑘$, we _reduce the capacity_: decision boundaries between classes become smoother and we _misclassify more_ and more points of the training set. Why shall we want to do it?
- Because... 
![[model_capacity.png]]
- As we can see from the picture, the area of $k=1$ are very much flexible, since we have blue "spots" in green's area. 

### Overfitting and underfitting
When varying model complexity, the training error (blue) and the validation error (red) usually follow a trend like this. To obtain the best generalization, we want the model to work in the “sweet spot” where:
- the training error is small 
- the gap between the training and validation error is also small.
When the training error is large, the algorithm is underfitting the training data. When the gap between the training error and the validation error is large, the algorithm is overfitting the training data.
![[image_classification_overfitting.png]]
### Model and algorithm selection
Q: How can we know which model will work best (will have the lowest generalization error) on unseen data? 

A: “We held out a small part of the __training set__, which we call _validation set_. We choose the _hyperparameter_ combination performing best on the _validation set_. We will finally run it (once) on the test set to evaluate generalization”.

Remember that in general the validation error will underestimate the generalization (or test) error.
### Image flattening
To apply distances defined for vectors to RGB images (i.e. third-order tensors), we will flatten them.

### What is kNN learning on CIFAR10?
Accuracy about 38%.

Using raw pixel values as features often focuses on context 
- a deer on a blue background is classified as plane 
- a horse on a dark background is classified as frog 

To improve from here, we can try to define: 
- better representations 
- better distances

It's shit. 

### The curse of dimensionality
𝑘-NN classifiers are, in principle, universal approximators. But, to provide reliable predictions, they _would need examples of all the possible_ 32x32 RGB images of the ten CIFAR classes, e.g. a lot of deer on a blue background, horses on a dark background, etc… How many 32x32 RGB images are out there? “only” $10^{8.200}$.
- So, it can be trained on any function, provided there's enough data. And the data is never enough...

