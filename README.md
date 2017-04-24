# holiday-similarity

Various attempts to build a neural network to distinguish between similar and different images from the [INRIA Holiday Photos Dataset](http://lear.inrialpes.fr/~jegou/data.php).

Project started out as an attempt to figure out how to use the Keras ImageDataGenerator to apply the same transformation to a pair of input images. This can be found in [01-holidays-data-augment.ipynb](src/01-holidays-data-augment.ipynb).

I then attempted to build a Siamese network based on the Keras example [mnist_siamese_graph.py](https://github.com/fchollet/keras/blob/master/examples/mnist_siamese_graph.py). This uses a shared convolutional network with Contrastive Divergence as the loss function. This network predicts a number between 1 and 0, 1 being very similar and 0 being very dissimilar. For the Convolutional Network, I used LeNet configuration, which is probably not powerful enough to do any learning for the Holidays dataset. You can see the code in [02-holidays-siamese-network.ipynb](src/02-holidays-siamese-network.ipynb). Even with a slightly more complex network configuration that I started out with, I was unable to get more than 60% accuracy on the network, so I abandoned this approach.

Next I tried to set up a baseline by generating image vectors from a bunch of pretrained networks available as part of Keras. Image Vectors were generated using VGG-16, VGG-19, ResNet50, InceptionV3, and xCeption networks. The notebook for vector generation is [03-pretrained-nets-vectorizers.ipynb](src/03-pretrained-nets-vectorizers.ipynb).

In order to train a network if an image pair is similar or not, I would take the training triple consisting of the (image\_left, image\_right, label) triple and lookup vectors for the two images, then use a merge strategy to merge the two vectors. The strategies used are element-wise cosine (dot), element-wise absolute difference (l1) and element-wise euclidean distance (l2). For each of the strategies above, we feed the resulting vector into a Naive Bayes, SVM, XGBoost and Random Forest classifier respectively. All but the 3rd is from Scikit-Learn, and the 3rd is from the XGBoost package. The notebooks for these are [04-pretrained-vec-dot-classifier.ipynb](src/04-pretrained-vec-dot-classifier.ipynb), [05-pretrained-vec-l1-classifier.ipynb](src/05-pretrained-vec-l1-classifier.ipynb) and [06-pretrained-vec-l2-classifier.ipynb](src/06-pretrained-vec-l2-classifier.ipynb). The best results were with XGBoost classifier, and the two top vectorizer networks were ResNet50 and InceptionV3.

We then replace the XGBoost classifier with a 3 layer fully connected network and repeat the experiment using the ResNet50 and InceptionV3 vectors, and each of the 3 merge strategies in addition to concatenation, in [07-pretrained-vec-nn-classifier.ipynb](src/07-pretrained-vec-nn-classifier.ipynb). Here we find that the best results come with dot product and Inception V3 vectors.

Finally, we replace the vectorizer backend network with a pre-trained InceptionV3 network with the prediction layer removed, similar to how it is used for generating image vectors, and reuse the FCN we trained in the previous notebook as the head of this network, in [08-holidays-siamese-finetune.ipynb](src/08-holidays-siamese-finetune.ipynb). A Siamese network requires that we share the CNN, but since we are treating the weights for the Inception network as frozen, it makes no difference whether we share the network or use copies of the network. We also use the ImageDataGenerator to augment our images, something we cannot do when using vectors. Resulting model is unfortunately not as good as the one using image vectors from a pre-trained network with a FCN front-end.

