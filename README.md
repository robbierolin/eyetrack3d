# Eyetrack

The purpose of this project is to explore the feasibility of Convolutional Neural Networks for 3D gaze estimation. The goal of 3D gaze estimation is to determine the 3D point in space that a person is looking at. 

An image-based solution has the benefits of being lightweight and flexible, which is of particular importance considering the growing number of AR/VR devices. In the project we use an image of each of the left and right eyes to predict gaze location.

## Data Collection
Eye images were captured at 5Hz using a SMI eyetracker installed in an Oculus DK2 which captures left and right eye images asynchronously. Each eye image is 320 x 240.

Using OpenGL, a red cube with side length 0.1 moved around a black background while eye images were captured.  The cube moved between two randomly chosen points with a collection volume over the course of 2 seconds.  The collection volume had coordinate 0.2 x 0.2 x 1.8 and was head relative

The data collection session lasted 10 minutes and produced 3016 left and right eye images (~2.3 GB).

<img src="/res/eyevideo.gif" width="400" alt="Data">

## Preprocessing
* Since left and right eye images and the cube positions were captured asynchronously, timestamps were used to create data points of the form (<em>I<sub>L</sub></em>, <em>I<sub>R</sub></em>, <em>p</em>) where <em>I<sub>L</sub></em>, and <em>I<sub>R</sub></em> are left and right eye images, respectively, and <em>p</em> is the position of the cube at close points in time.
* To account for saccades, we removed the first two data points of images that occurred after the cube started on its path between two points in the collection volume. 
* Images were manually scanned to remove those with blinks.
* The data was normalized by subtracting the mean and dividing by the standard deviation for each pixel.
* Output data was normalized to [0,1] in each dimension

There were 2695 images after preprocessing (11% removed).  These were randomly split into 1887 training data, 404 validation data, and 404 test data (70:15:15)

We can plot the cube positions of our data to see how well distributed it is.

<img src="/res/train.png" width="400" alt="Training Data Distribution"><img src="/res/val.png" width="400" alt="Validation Data Distribution"><img src="/res/test.png" width="400" alt="Test Data Distribution">

Looks good.

## Model
We learn a two-stream convolutional neural network. Each stream takes an image of either the left eye or right eye as input. The latent features of the images are merged before they are input to a fully connected layer that outputs the x,y, and z coordinates of the cube position. All activation functions are RELU except the output layer has a sigmoid.

The two streams have identical architectures which are as follows:

    Layer 1: CONV2_1 5 (20x20) filters
    Layer 2: MAXPOOL_1 (2x2) pool
    Layer 3: CONV2_2 5 (3x3) filters
    Layer 4: CONV2_3 5 (3x3) filters
    Layer 5: MAXPOOL_2 (2x2) pool
    Layer 6: CONV2_4 10 (3x3) filters
    Layer 7: CONV2_5 10 (3x3) filters
    Layer 8: CONV2_6 10 (3x3) filters
    Layer 9: MAXPOOL_3 (2x2) pool

This gives a total of 81493 parameters, with 72003 from the fully connected layer and 4745 from each eye stream.

## Training 
After some tedious manual searching, the network was trained with the following properties:
* optimizer: Adadelta (standard parameters)
* batch_size: 1
* epochs: 15
* regularization: only on the last layer (L2, lambda = 10<sup>-4</sup>)
* loss: Mean Squared Error (MSE)

Training took less than 10 minutes on my GTX 980.

Here are some plots of mean squared error and mean absolute error (MAE), measures of how far away our predictions are.

<img src="/res/train_loss.png" width="300" alt="Training Loss"><img src="/res/train_mae.png" width="300" alt="Training Mean Absolute Error"><img src="/res/val_loss.png" width="300" alt="Validation Loss"><img src="/res/val_mae.png" width="300" alt="Validation Mean Absolute Error">

The network reached the following metrics during training
* MSE_train: 0.0036
* MAE_train: 0.0361
* MSE_val: 0.0092
* MAE_val: 0.0580

## Results
* MSE_test: 0.0066
* MAE_test: 0.0539

Here is how the network performs on the unseen test data. The red cube is what was looked at during training and the blue is the NN output.

<img src="/res/TestData.gif" width="500" alt="Performance on test data">

And here is how it performs over all the data in order.

<img src="/res/AllData-resized.gif" width="500" alt="Performance on all data">

## Visualizing Network

To see what the network learned we run gradient ascent on noise images to maximize the activations of each neuron in our network. These are some that were the most activated.

CONV2_1

<img src="/res/conv_1_left_filter_1.png" width="300"> <img src="/res/conv_1_left_filter_3.png" width="300">

CONV2_2

<img src="/res/conv_3_left_filter_0.png" width="300"> <img src="/res/conv_3_left_filter_3.png" width="300"> <img src="/res/conv_3_right_filter_2.png" width="300">

CONV2_4

<img src="/res/conv_5_left_filter_8.png" width="300"> <img src="/res/conv_5_right_filter_8.png" width="300">

CONV2_5

<img src="/res/conv_6_left_filter_4.png" width="300">

CONV2_6

<img src="/res/conv_7_right_filter_2.png" width="300">


## Conclusions

These are really nice results.  We have shown that it's possible to learn meaningful eye tracking features using Convolutional Neural Networks. A usable model for a wide variety of settings would likely need to be trained on more varied training data. 

Knowledge of eye behaviours (i.e. saccades and pursuits) could easily be incorporated into time-series models built on top of these predictions to create robust, accurate trackers. Deep sequence models (e.g. LSTM) could also be used to learn this behaviour. Higher frequency capture devices would likely be needed. 


