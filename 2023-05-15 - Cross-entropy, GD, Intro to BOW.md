### Can we defeat the curse of dimensionality?
kNN computes a different decision boundary for every region of the space. Very powerful but it needs a lot of samples.
- What happens if we use just one __global parametric__ model?

### Linear classifier example
![[shitty_classifier_example2.png]]
![[shitty_classifier.png]]
But...
__Bad idea!__ Classes are categorical variables, their assignment to integer ids is random and _it is not designed to capture the semantic of classes_ (i.e. if cat is class 3 and we get class=2,5 out of 𝑓, it does not mean that 𝑥 depicts half a bird and half a cat; or, similarly, the fact that two ids are nearby or far away does not mean that the corresponding classes share or not visual similarities)

A better representation would be:
![[improved_model_lineart.png]]

## Template matching
![[template_matching 1.png]]
It's a technique that we've seen during the Image Processing module. 
- It's a correlation since I don't want to "flip" the image.  

## In ML, linear often means affine…
![[ml_linear_affine.png]]

## Learning as optimization
The space of the functions that a machine learning model can produce is its hypothesis space $ℍ$.
So, we went from a viewpoint of this kind (in kNN):
- Learning = solve an optimization problem to find the “best” function $ℎ ∈ ℍ$
![[best_function.png]]

To one of this kind, by introducing _parametric models_.
- Learning = solve an optimization problem to find the _“best” parameters_ $𝜃 ∈ Θ$
![[learning_optimization_problem.png]]
The parameters are the weights and biases of our model basically. 

## 0-1 loss
Can the loss function be the number of errors? ($𝑳 = \#𝒆𝒓𝒓𝒐𝒓𝒔$?)
![[01loss.png]]
This choice, known as the 0-1 loss, results in a hard optimization problem. Given the classifier represented by the solid line, changing it to become the dashed or the dotted line produces the same number of errors: $\#errors$ is insensitive to small (and even large, sometimes) changes of the parameters. However, they are not equivalent directions of change: if we keep moving the classifier in the direction of the dotted line, we will improve it; whereas, if we keep moving in the direction of the dashed one, we will make more errors: ==the error rate does not tell us if we are “moving” in the “right” direction while we change the parameters==.

## Loss function
Instead of directly optimizing accuracy, we then usually optimize a _proxy measure_, the _loss function_, that is easier to optimize but still correlated with how good our classifier is.
Loss function is also called objective function, cost function, error function,…  
- If the loss is _high_, our classifier is performing _poorly_, and we also expect _low accuracy_.
- If the loss is _low_, our classifier is _good_, and we also expect _high accuracy_.

Hence, we prefer values of the parameters that minimize it on the training set.

We will always work with losses whose value on a dataset is the _average_ (or the sum) of the values for the _single samples_:
![[loss.png]]
#### What can we use as loss function?
Historically, the __RMSE__ (Mean Squared Error) between the prediction and the training label was used:
![[RMSE.png]]

## Softmax function
It turns out there are theoretical and practical reasons to prefer a loss that transforms the _scores_ computed by the classifier into _probabilities_ and then perform __maximum likelihood estimation__ (MLE) of $𝜃$.
- How do we go from scores $𝑠_𝑗$ to probabilities? We use the __softmax function__ (softmax: $ℝ^𝑛$ → $ℝ^𝑛$ )
	- In this way, we ensure that the scores sum to 1 and are all below 1 (but also that they are non-negative).
![[softmax.png]]
![[softmax_example.png]]

Should be called “softargmax”, as it is a smooth and differentiable approximation of the one-hot encoding of the 𝒂𝒓𝒈𝒎𝒂𝒙 To implement it reducing numerical issues , it is useful to note that. 

Note:
- $softmax_𝑗(𝑠 + 𝑐) = softmax_𝑗(s)$ (he)
This can help us with numerical issues related to the explosion of the exponential. 
In particular, we can compute: $softmax(𝑠 − max_𝑘 𝑠_k$)
- which is the the softmax of $s$ subtracted of the maximum score. In this way the largest score is 0, while the other all negatives.   

## Cross-entropy loss
Now that our linear classifier outputs «probabilities» over the classes, we can think of it as a family of _probability mass functions_ over the _classes_ given an image, indexed by the vector of parameters $θ$.
![[softmax342.png]]
Then, the _maximum likelihood estimation_ of $𝜃$ is:
![[mle_of_theta.png]]
- The __higher__ the value of the _cross-entropy loss_, the __worse__ is the prediction.
Normally, we would like cross-entropy to be around 0.5.

## Putting everything together (aka proving that cross-entropy is a good proxy measure)
Given
![[p_model.png]]
and the per-sample loss to minimize to perform maximum likelihood estimation
![[mle.png]]
the overall expression for the per-sample loss becomes
![[logsumexp.png]]
The second term of the sum on the right is usually referred to as the `logsumexp` and numerical libraries (e.g. PyTorch) usually have numerically stable functions to compute it.
It is an _approximation_ of the _max function_, hence we can think of the __cross-entropy loss__ as approximately
![[final_form.png]]
and of minimizing it as ==penalizing the most active incorrect prediction==, i.e. a _proxy to increase accuracy_.
- In fact, it is basically subtracting the score of the predicted class to the score of the correct class.  

### Cross-entropy vs. Negative Log Likelihood
In short, they are basically the same thing, but:
- Cross-entropy expects raw, unnormalized values
- Negative Log Likelihood wants log-probabilities (i.e. values after using a LogSoftmax).

## Gradient descent
Now, what is left to do is to find a way to select the best parameters for our model. 
How can we use this knowledge to train our model, i.e. select the “best” $𝜃$?
With __gradient descent__!

![[GD.png]]
The algorithm:
![[gd_algo.png]]
We also have some new possible design choices, such as:
- Initialization method? 
- Number of epochs? Or alternative way to decide to stop? 
- Learning rate 𝒍𝒓?

### How to compute gradients?
- __Numerically__, as we just did 
	- _Slow_ and _approximate_, but _easy_ to implement.

- __Analytically__, by exploiting the rules of calculus and, in particular, the __chain rule__:
$$
\dfrac{d}{dx}f(g(x)) = f'(g(x))g'(x)
$$
		- _Exact_, but _slow, tedious_, and error prone.

- __Automatically__, by automatic differentiation, e.g. with the _backpropagation algorithm_.

[end]

## What does a linear model learn?
The accuracy of the model is again pretty low, about 38%.
Let’s use the _template matching interpretation_ of a linear classifier to understand what the model is learning.
![[what_is_learned_baby_dont_hurt_me.png]]
It looks like the background color is still the predominant feature used by the model.
Moreover, one template cannot capture multiple appearances within one class, e.g. rotated cars, trucks, etc.. 
Distance between templates and images is still a distance in input space, same problem we had with k-NN classifier, and performance is similar.

So, we need our model to learn __more effective representations__.

---
# Image representations
As we know, 𝑘-NN and linear classifiers are limited by the low effectiveness of input pixels as data features. 

## Bag of words
Many ideas from NLP are used in CV. The BoW is one of them. 
Within the computer vision community, this representation is referred to as the Bag of Words (BoW) or the bag of visual words (BoVW) or the bag of features (BoF) representation
![[Images/bow.png]]
It's called "bag" because it's sort of an unstructured collection of features. 

Given an image, we would like to represent it as a _histogram_, counting the frequency of appearance of every _(code)word_ from a _dictionary_. But images are not made of discrete elements: 
- how can we extract “words” from images? 
- how can we create the dictionary (aka codebook, vocabulary,…)?