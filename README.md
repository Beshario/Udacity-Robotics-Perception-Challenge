# RoboProj3
Udacity Robotic's Perception Challenge


# 3D Perception Project
---
[//]: # (Image References)

[clsutering]:https://user-images.githubusercontent.com/6395647/31702867-c5a1db28-b3a7-11e7-982b-d482b24d44e0.png
[extracte inliners]:https://user-images.githubusercontent.com/6395647/31702868-c5b508ce-b3a7-11e7-80fc-512fc1870ee3.png
[extracted outliers]:https://user-images.githubusercontent.com/6395647/31702869-c5c21a8c-b3a7-11e7-9276-ae18665a876f.png
[figure_1-1]:https://user-images.githubusercontent.com/6395647/31702870-c5d27c7e-b3a7-11e7-8e98-73cf72758f86.png
[figure_1]:https://user-images.githubusercontent.com/6395647/31702871-c5df706e-b3a7-11e7-81ed-abdcc3e0f30f.png
[figure_2]:https://user-images.githubusercontent.com/6395647/31702872-c5ea98fe-b3a7-11e7-870d-d5cb16c34dc8.png
[figure_3econd try s]:https://user-images.githubusercontent.com/6395647/31702873-c5f7236c-b3a7-11e7-8a91-df820ace549c.png
[final normalized]:https://user-images.githubusercontent.com/6395647/31702874-c6096f4a-b3a7-11e7-957a-a7271708e0a7.png
[final]:https://user-images.githubusercontent.com/6395647/31702875-c6165fe8-b3a7-11e7-93fb-4ac1f26b0b74.png
[finalfinalfigure_1]:https://user-images.githubusercontent.com/6395647/31702876-c6239b9a-b3a7-11e7-8c36-afe75337a914.png
[finalnormalized]:https://user-images.githubusercontent.com/6395647/31702877-c62f5516-b3a7-11e7-978c-f73085ab1d24.png
[output2]:https://user-images.githubusercontent.com/6395647/31702878-c63ce334-b3a7-11e7-808d-b7b6ed1ee663.png
[passthrough filter]:https://user-images.githubusercontent.com/6395647/31702879-c649bcbc-b3a7-11e7-8b25-7a30924a42c6.png
[scene1]:https://user-images.githubusercontent.com/6395647/31702880-c657ab38-b3a7-11e7-97cc-4d396134cdf2.png
[screenshot from 2017-09-30 10-53-59]:https://user-images.githubusercontent.com/6395647/31702881-c665283a-b3a7-11e7-9ab3-8f30b1d93a58.png
[screenshot from 2017-10-17 03-16-35]:https://user-images.githubusercontent.com/6395647/31702882-c672628e-b3a7-11e7-89d2-0ac5c9e25600.png
[table]:https://user-images.githubusercontent.com/6395647/31702883-c680610e-b3a7-11e7-8868-18801c699329.png
[voxel downsampling]:https://user-images.githubusercontent.com/6395647/31702884-c68c86b4-b3a7-11e7-9ae1-9cc44022b1ca.png
[voxel0 01]:https://user-images.githubusercontent.com/6395647/31702885-c6a220dc-b3a7-11e7-802d-827f6bb46477.png




![pr2 robot][pr2_robot]

This project is about programming the robot to make it (understand) what the robot is seeing via an RGB-D camera. an RGB-D camera is like a normal camera with a depth sensor. therefore it registers information about the colours (RGB sensors) and depth (distances from the camera to object surfaces).

This report will go through the pipeline of Robot Perception

The three main parts of perception are: filtering the data, clustering relevant objects, and recognizing objects the clustered objects.

## Contents

- [Filtering](#filtering)
- [Clustering for Segmentation](#clustering-for-segmentation)
- [Object Recognition](#object-recognition)
- [PR2 Robot Simulation](#pr2-robot-simulation)
- [Code Sources](#code-sources)
- [Run the Simulation](#run-the-simulation)

## Filtering

First, the data is stored as a [point cloud data](https://en.wikipedia.org/wiki/Point_cloud) The raw point cloud object from the PR2 simulation looks like this:

![raw cloud object][pipeline_0_raw_cloud]

### Outlier Removal Filter

Sensors come with noise data, in the case of RGB-D cameras, they are manifested as random colour and spacial points over the image. In order to filter the noise, a method as the [PCL's StatisticalOutlierRemoval filter](http://pointclouds.org/documentation/tutorials/statistical_outlier.php) computes an average distance to a group of points, if a point is too far whose mean distances are outside a defined interval (determined by a given standard deviation) are removed.

For the PR2 simulation, I found that a mean k value of *20* and a standard deviation threshold of *0.1* provided the optimal outlier filtering. Here is the cloud after performing the outlier removal filter.

![outlier removal filtering][pipeline_1_outlier_removal_filter]

### Voxel Grid Filter

![voxel grid filter][pipeline_2_voxel_grid_filter]
a voxel to a voxel  is the volume equivalent of a pixel to an image,

A voxel downsamples the data by taking the average of the data points inside of it(RGB and Depth). therefore the set of points are statistically represented by that voxel

a Voxel of *0.01*mm was used, small enough to not leave important details out and easy to compute
### Passthrough Filter

![passthrough filter][pipeline_3_passthrough_filter]

Passthrough filter is a 3D-cropping mechanism that works by cropping from two ends along a specified axis.

The PR2 robot simulation needed two passthrough filters for both the Y and Z axis (global). This prevented processing values outside the area immediately in front of the robot. For the Y-axis, a range of *-0.4* to *0.4* was used, and for the Z axis, a range of *0.61* to *0.9* was used.

### RANSAC Plane Segmentation

Random sample consensus is an iterative model that tests an example of points in a dataset to see if it fits a certain model (here, a group of points are tested if they fit a relationship equation that describes a plane surface in 3D) 
after certain iterations, a model can be confirmed existent (a plane is found at certain points), the dataset is divided into groups, inliers are the points that fit the model, outliers are the ones that do not fit --this method could be also used to as outlier removal method--.

A max distance value of *0.01* was used

The extracted inliers included the table.
![RANSAC plane segmentation - extracted inliers][pipeline_4_extracted_inliers]

The extracted outliers was everything else( the objects on the table):

![RANSAC plane segmentation - extracted outliers][pipeline_5_extracted_outliers]

## Clustering for Segmentation

With a filtered point cloud, we can perform segmentation. 

The two main clustering algorithms here are:

- K-means is used for (pre-defined) number of clusters 

- DBSCAN


### DBSCAN

The density-based spatial cluster of applications with noise (also called *Euclidean clustering*) is a clustering algorithm that creates clusters by grouping data points. DBscan starts with one point, then hops to other points within predefined distance till it reaches edge points that cannot hop further through. these points now form a cluster.

DBscan finds other clusters the same way, nevertheless, if there's a certain cluster that has less than a minimum defined number of points, it is considered an outlier cluster or noise



DBSCAN receive a [K-d tree](https://en.wikipedia.org/wiki/K-d_tree) data type, look at this quick lesson [here](https://www.youtube.com/watch?v=TLxWtXEbtFE)

DBSCAN has an advantage over k-means because you need to know about the density of the data not the number of clusters, which you may not have


because KDtree is a spacial data type (based on spatial info) the data was converted to only x,y,z point cloud. DBSCAN is performed then indices of clusters are returned, we extracted

 the clusters from the cloud and randomly coloured each cluster as shown:

![DBSCAN object cluster][dbscan_object_cluster]

## Object Recognition

The object recognition code allows each cluster to be examined and identified. In order to do this, the system first needs to get trained to make a model to learn what each object looks like. Once it makes the model, the system can make predict which object it sees.

### Capture Object Features

The code for building the histograms can be found in [features.py](PR2-Project/sensor_stick/src/sensor_stick/features.py).

The [capture_features_pr2.py](PR2-Project/sensor_stick/scripts/capture_features_pr2.py) script saved the object features to a file named `training_set_pr2.sav`. It captured each object in *50* random orientations, using the *HSV* color space and *128* bins when creating the image histograms.

### Train SVM Model

A support vector machine (SVM) is used to train the model (specifically a SVC). The SVM loads the training set generated from the `capture_features_pr2.py` script, and prepares the raw data for classification. I found that a *linear kernel* using a C value of *0.1* builds a model with good accuracy.

I experimented with cross-validation of the model and found that a *50* fold cross validation worked best. A *leave one out* cross-validation strategy provided marginally better accuracy results, however, it required much longer to process.

In the end, I was able to generate an accuracy score of 92.17%. The [train_svm.py](PR2-Project/RoboND-Perception-Project/pr2_robot/scripts/train_svm.py) script trained the SVM, saving the final model as `model.sav`.

The confusion matrices below shows the non-normalized and normalized results for a test case using the trained model generated above.

![Confusion matrices for the non-normalized and normalized test case][confusion_matrices]

## PR2 Robot Simulation

The PR2 robot simulation has three test scenarios to evaluate the object recognition performance. The following sections demonstrate each scenario.

### Test 1

![Test 1 object recognition][test_1_object_recognition]

View the goal [pick list](PR2-Project/RoboND-Perception-Project/pr2_robot/config/pick_list_1.yaml), and the calculated [output values](PR2-Project/RoboND-Perception-Project/pr2_robot/scripts/output_1.yaml).

### Test 2

![Test 2 object recognition][test_2_object_recognition]

View the goal [pick list](PR2-Project/RoboND-Perception-Project/pr2_robot/config/pick_list_2.yaml), and the calculated [output values](PR2-Project/RoboND-Perception-Project/pr2_robot/scripts/output_2.yaml).

### Test 3

![Test 3 object recognition][test_3_object_recognition]

View the goal [pick list](PR2-Project/RoboND-Perception-Project/pr2_robot/config/pick_list_3.yaml), and the calculated [output values](PR2-Project/RoboND-Perception-Project/pr2_robot/scripts/output_3.yaml).

## Code Sources

The original Udacity perception exercise code can be found [here](https://github.com/udacity/RoboND-Perception-Exercises). To install the exercise data on your own computer, view the installation guide [here](Install_Guide.md).

The PR2 3D perception project code source can be found [here](https://github.com/udacity/RoboND-Perception-Project). Follow the instructions in the README.md to get the demo code working.

## Run the Simulation

View instructions for running the code mentioned above [here](PR2-Project/RoboND-Perception-Project/pr2_robot/scripts/README.md).
