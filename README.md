# Readme - Eyetrack

The purpose of this project is to explore the feasibility of Convolutional Neural Networks for 3D gaze estimation. The goal of 3D gaze estimation is to determine the 3D point in space that a person is looking at. 

An image-based solution has the benefits of being lightweight and flexible, which is of particular importance considering the growing number of AR/VR devices.

## Data Collection
Eye images were captured at 5Hz using a SMI eyetracker installed in an Oculus DK2.  

Using OpenGL, a red cube with side length 0.1 moved around a black background while eye images were captured.  The cube moved between two randomly chosen points with a collection volume.  The collection volume had coordinate 0.2 x 0.2 x 1.8 and was head relative

The data collection session lasted 10 minutes and produced 3016 left and right eye images.

[eyevideo]: https://github.com/robbierolin/eyetrack3d/res/eyevideo.avi "Training Images"
![alt text][eyevideo]

## Preprocessing
* Since left and right eye images and the cube positions were captured asynchronously, timestamps were used to create data points of the form (I_l, I_r, p) where I_l, and I_r are left and right eye images, respectively, and p is the position of the cube at close points in time.
* To account for saccades, we removed the first two data points of images that occurred after the cube started on its path between two points in the collection volume. 
* Images were manually scanned to remove those with blinks.
* The data was normalized by subtracting the mean and dividing by the standard deviation for each pixel.
* Output data was normalized to [0,1] in each dimension

There were X images after preprocessing.  These were randomly split int X1 training data X2 validation data and X3 test data (70:15:15)

We can plot the cube positions of our data to see how well distributed it is.

[train]: https://github.com/robbierolin/eyetrack3d/res/train.png "Training Data"
[val]: https://github.com/robbierolin/eyetrack3d/res/val.png "Validation Data"
[test]: https://github.com/robbierolin/eyetrack3d/res/test.png "Test Data"
![alt text] [train]
![alt text] [val]
![alt text] [test]
Looks good.

## Model
We learn a two-stream convolutional neural network. Each stream takes an image of either the left eye or right eye as input. The latent features of the images are merged before they are input to a fully connected layer that outputs the x,y, and z coordinates of the cube position. All activation functions are RELU except the output layer has a sigmoid.

The two streams have identical architectures which are as follows:

    Layer 1: CONV2 5 (10x10) filters
    Layer 2: CONV2 5 (10x10) filters
    Layer 3: MAXPOOL (2x2) pool
    Layer 4: CONV2 5 (3x3) filters
    Layer 5: CONV2 5 (3x3) filters
    Layer 6: MAXPOOL (2x2) pool
    Layer 7: CONV2 10 (3x3) filters
    Layer 8: CONV2 10 (3x3) filters
    Layer 9: MAXPOOL (2x2) pool


## Training 
After some tedious manual searching, the network was the following properties
* Optimizer: Adadelta (standard parameters)
* batch_size: 1
* epochs: 
* regularization: only on the last layer (L2, \lambda = 10^{-4})
* loss: Mean-squared error

Training took X.Y hours on my GTX 980.

Here are some plots of mean squared error and mean absolute error, measures of how far away our predictions are.

[IMAGE][IMAGE][IMAGE]

The network reached the following metrics during training
* MSE_train: 
* MAE_train:
* MSE_val:
* MAE_val:

## Results
* MSE_test:
* MAE_test:

Here is how the network performed on the unseen test data. 
[VIDEO]

And here is how it performs over all the data.
[VIDEO]

## Visualizing Network

To see what the network learned we run gradient ascent on noise images to maximize the activations of each neuron in our network. These are some that were the most activated.

[IMAGE][IMAGE][IMAGE]

## Conclusions

These are really good results.  Knowledge of eye behaviours (i.e. saccades and pursuits) could easily be incorporated into time-series models built on top of these predictions to create robust, accurate trackers. Deep sequence models (e.g. LSTM) could also be used to learn this behaviour. Higher frequency capture devices would likely be needed. 
