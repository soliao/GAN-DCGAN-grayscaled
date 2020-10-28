# DCGAN-grayscale-MNIST-Fashion-MNIST-Food-101
Use deep convolutional generative adversarial networks (DCGAN) to generate images in grayscale


### References

1. Original paper\
Radford et al. (2015) **Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks** [[arXiv](https://arxiv.org/abs/1511.06434)]

2. Tutorial on immplementation using MXNet [[link](https://gluon.mxnet.io/chapter14_generative-adversarial-networks/dcgan.html)]

3. Lazy Programmer's GAN (with fully connected layers) Colab Notebook: https://colab.research.google.com/drive/1NGi0HyEuR8cMWyzmGzBiv06PDqmxZddY

4. This post on some useful information
https://machinelearningmastery.com/practical-guide-to-gan-failure-modes/


# Datasets

2 datasets were tested:

**MNIST**

MNIST dataset consists of 70,000 (60,000 training and 10,000 testing) images of hand-written digits (0-9). The dataset can be loaded by:

``` python3
## Load the mnist dataset

print("Using MNIST dataset")
print("60,000 images, each image with size 28 x 28 (grayscale)")

train_data, _ = tf.keras.datasets.mnist.load_data(path = "mnist.npz")

orig_data = np.array(train_data[0])

orig_data.shape
```

In the code above, I only loaded the 60,000 training data to be my dataset.


**Fashion-MNIST**

Fashion-MNIST dataset consists of 70,000 (60,000 training and 10,000 testing) images of Zalando's articles (T-shirts, Trousers, Pullovers, ...). The dataset can be loaded by:

``` python3
## Load the fashion-mnist dataset

print("Using Fashion-MNIST dataset")
print("60,000 images, each image with size 28 x 28 (grayscale)")

train_data, _ = tf.keras.datasets.fashion_mnist.load_data()

orig_data = np.array(train_data[0])

orig_data.shape
```

In the code above, I only loaded the 60,000 training data to be my dataset.



# Hyperparameters

### General principles

These are the general features of DCGAN I used for both (MNIST & Fashion-MNIST) datasets

#### Leaky-Relu activation
The original paper suggested using ReLU for the generator, while using LeakyRelu activation in the discriminator. However, I used LeakyRelu (`alpha = 0.2`) in **both generator and discriminator**.

#### Batch normalization
Contrary to the original paper, I only used batch normalizations in **hidden Conv2DTranspose layers** in the **generator**. I didn't use batch normalization in the discriminator.

#### Weight initialization
As recommended by the original DCGAN paper, I initialized the wieghts by normal distribution (stddev = 0.02)

``` python3
from tensorflow.keras.initializers import RandomNormal

w_init = RandomNormal(mean = 0.0, stddev = 0.02)
```

#### Batch size
In each epoch, the generator created 32 fake images. These 32 fake images and 32 real images sampled from the dataset will be used to train the discriminator. After then, a new batch of 32 fake images will be used to train the generator.

#### Learning rate
I used learning rate = 1e-5 for both the generator and the discriminator. 



# Model architectures

## MNIST and Fashion-MNIST

**Generator**

``` python3
def build_generator_DC(hidden_dim = 100):
    
    ## weight initialization
    w_init = RandomNormal(mean = 0.0, stddev = 0.02)

    ## input vector
    z = Input(shape = (hidden_dim, )) # (None, hidden_dim)
    
    ## Project and reshape
    n = H // 4 # H = height of the image
    x = Dense(n*n*128, kernel_initializer = w_init)(z)
    x = LeakyReLU(alpha = 0.2)(x)
    x = Reshape((n, n, 128))(x)

    ## Conv2D-T
    x = Conv2DTranspose(256, 4, 2, 'same', kernel_initializer = w_init)(x)
    x = BatchNormalization()(x)
    x = LeakyReLU(alpha = 0.2)(x)

    ## Conv2D-T
    x = Conv2DTranspose(256, 4, 2, 'same', kernel_initializer = w_init)(x)
    x = BatchNormalization()(x)
    x = LeakyReLU(alpha = 0.2)(x)
        
    ## last Conv2D (no batch norm!)
    img = Conv2D(1, 4, padding = 'same', activation = 'tanh', kernel_initializer = w_init)(x)
    
    # generator model
    model = Model(inputs = z, outputs = img, name = 'generator')

    return model
```

**Discriminator**

``` python3
def build_discriminator_DC():
    
    ## weight initialization
    w_init = RandomNormal(mean = 0.0, stddev = 0.02)

    ## input image (img)
    img = Input(shape = (H, H, 1))
    
    ## Conv
    x = Conv2D(256, 4, 2, padding = 'same', kernel_initializer = w_init)(img)
    x = LeakyReLU(alpha = 0.2)(x)

    ## Conv
    x = Conv2D(256, 4, 2, padding = 'same', kernel_initializer = w_init)(x)
    x = LeakyReLU(alpha = 0.2)(x)

    ## Conv
    x = Conv2D(256, 4, 2, padding = 'same', kernel_initializer = w_init)(x)
    x = LeakyReLU(alpha = 0.2)(x)

    ## final layer
    x = Flatten()(x)
    y = Dense(1, activation = 'sigmoid')(x)

    # generator model
    model = Model(inputs = img, outputs = y, name = 'discriminator')

    return model
```
