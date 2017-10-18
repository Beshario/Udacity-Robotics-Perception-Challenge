
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



![demo-1](https://user-images.githubusercontent.com/20687560/28748231-46b5b912-7467-11e7-8778-3095172b7b19.png)

This project is about programming the robot to make it (understand) what it sees via an RGB-D camera. an RGB-D camera is like a normal camera with a depth sensor. therefore it registers information about the colours (RGB sensors) and depth (distances from the camera to object surfaces).

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

For the PR2 simulation, a mean k value of *20* and a standard deviation of *0.1* provided a cleaner image.

### Voxel Grid Filter

![voxel grid filter][voxel0 01]
a voxel to a voxel  is the volume equivalent of a pixel to an image,

A voxel downsamples the data by taking the average of the data points inside of it(RGB and Depth). therefore the new set of points are statistically represented by that voxel

a Voxel of *0.01*mm was used, small enough to not leave important details out and efficient to compute

### Passthrough Filter

![passthrough filter][passthrough filter]

Passthrough filter is a 3D-cropping mechanism that works by cropping from two ends along a specified axis.

The PR2 robot simulation needed two passthrough filters for both the Y and Z axis (global). This lets the robot focus only on what it is important. For the Y-axis,  whatever was out of *-0.4* to *0.4* was cropped, and for the Z axis, it was whatever was outside of  *0.6* to *0.9*.

### RANSAC Plane Segmentation

Random sample consensus is an iterative model that tests an example of points in a dataset to see if it fits a certain model (here, a group of points are tested if they fit a relationship equation that describes a plane surface in 3D) 
after certain iterations, a model can be confirmed existent (a plane is found at certain points), the dataset is then divided into groups, inliers are the points that fit the model, outliers are the ones that do not fit --this method could be also used  as outlier removal method--.

A max distance value of *0.01* was used as the same voxel size

The extracted inliers included the table.
![RANSAC plane segmentation - extracted inliers][extracte inliners]

The extracted outliers was everything else( the objects on the table):

![RANSAC plane segmentation - extracted outliers][extracted outliers]

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

![DBSCAN object cluster][clsutering]

## Object Recognition

The object recognition code allows each cluster to be examined and identified. In order to do this, the system first needs to get trained to make a model to learn what each object looks like. Once it makes the model, the system can then predict which object it sees.

The Robot can extract color and normal histograms of objects

### Train SVM Model

A support vector machine (SVM) is used to train the model. The SVM creates a model from a predefined histogram data from a training set. the training set file is generated from `capture_features_pr2.py` script, then the robot classifies to best what it see to the model it made of what it already learned (cool huh?). a *linear kernel* using a C value of *0.1* is found accurate.

For corss validation, a repetition of 20 was enough for the first  scene, however, 40 repetitions were enough for the other two worlds.

For the project, a general accuracy of *97%* of identifying objects was acheived

The confusion matrices below shows the non-normalized and normalized results for a test case using the trained model generated above.

![Confusion matrix][figure_2]

## PR2 Robot Simulation

The PR2 robot simulation has three test scenes to evaluate the object recognition pipeline. The following sections demonstrate each scenario.

### Test 1

![Test 1 object recognition][scene1]


### Test 2

![Test 2 object recognition][output2]


### Test 3

![Test 3 object recognition][screenshot from 2017-10-17 03-16-35][test_3_object_recognition]


Please Do find the Training Set, model and out Yaml files in the output folder
