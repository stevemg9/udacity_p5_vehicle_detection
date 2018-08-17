
# Vehicle Deteciton Project
Stephen Giardinelli

A quick note on the code in my notebooks.  Some of the code in these notebooks was taken from or has been adapted from the code provided in the Udacity lessons.

## Data Exploration
The data consisted of both car and non-car images that were 64x64x3 in dimension.  In order to augment the data, the images were flipped horizontally in order to provide a more robust data set.  Some of random selections of the data can be seen in the figure below.
![data_vis.png](output_images/data_vis.png)

## Feature Extraction
### Spatial Features
In order to find the most relevant spatial features, I trained a classifier and tested it's accuracy for different combinations of color space and spatial size.  As you can see in the figure below, the best performing combination was a 16x16 spatial feature set taken from an image in the LAB color space.
![spatial_historgram.png](output_images/spatial_historgram.png)
### Color Histogram
When exploring the color histograms I applied the same method of combining different color spaces with a different amount of bins for the histogram.  To visualize what the color histograms look like for a few randomly selected images and color spaces, view the figure below.
![histogram_fig.png](output_images/histogram_fig.png)
In order to choose the best combination of color space and amount of histogram bins, I trained a classifier on each combination, evaluated its accuracy and plotted the values on the figure below.  The best combination was the HSV color space with 44 bins.
![colorspace_historgram.png](output_images/colorspace_historgram.png)
### Histogram of Gradients (HOG)
I performed the same task of visualizing the HOG features for random car and non car images.  The results can be seen in the figure below.

I also ran multiple tests for combinations of color space and amount of channels. When considering channels, using all three channels combined yielded better accuracy than any single channel.  In terms of color space, they all yielded comparable accuracy with the exception of the RGB color space - LAB, YUV, and LUV were the top three, in that order, but only by a small margin.
![hog_historgram.png](output_images/hog_historgram.png)
I also performed a test on different combinations of pixels per cell, cells per block, and orientation bins.  I tested the following values:
    - Cells Per Block = [1,2] (I attempted 4, as well but I ran out of memory crashed the process)
    - Pixels Per Cell = [4,8,16]
    - Orientation Bins = [7, 9, 11, 13, 15, 17]
I tested all of these on the LAB and YUV color space's but for the sake of brevity, I will only show the plots for YUV.  I ended up choosing YUV in the end because it proved to show less false positives when run on the video.  Also, even though orient = 17 provided better accuracy on this test, when applied to the video orient=9 yielded better results with fewer false positives.
![hog_yuv_cpb_1.png](output_images/hog_yuv_cpb_1.png)
![hog_yuv_cpb_2.png](output_images/hog_yuv_cpb_2.png)
Based on all of the aforementioned analysis, I arrived at a final feature extraction set of:
    - Spatial Features
        - Color space: LAB
        - Spatial Size: (16x16)
    - Color Histogram:
        - Color space: HSV
        - Number of Bins: 44
        - Histogram Range: [0,255]
    - HOG
        - Color space: YUV
        - Orient: 9
        - Pixels Per Cell: 8
        - Cells Per Block: 2

## Training A Classifier
In order to train a classifier, I used the above feature extraction parameters.  I trained a LinearSVM and the StandardScaler to normalize the data.  The classifier was trained on all of the car and non-car image from the project repo and I performed a horizontal flip on all images to augment the dataset and double the amount of training data.  My final accuracy score on the validation portion of the data was: 0.9945

## Sliding Window Search
In order to create the right crops for image vehicle detection, I had to experiment a lot with the actual video file and test images.  The final implementation that I used can be seen in the following figure.
![sliding_windows.png](output_images/sliding_windows.png)

### Improving the Classifier
In order to make a more robust vehicle detection system, I needed to filter out the false positives while still maintaining the true positives.  In order to do this I used a heat-map and threshold.  For each bounding box, every pixel within the bounds would have it's "heat" increased by one.  So we are hoping for multiple detections on each vehicle and few or single detection on false positives.  In order to transition this to video, I included "memory from frame to frame.  For each new frame, the detections from the past 10 frames would be added to the heat map.  This helps track the cars through the image and also make the system more robust to "dropped frames" where there are no detections on a vehicle.  A visualization of this process can be seen in the following figure.
![pipeline.png](output_images/pipeline.png)
As you can see, in the last image, we have a single detection on the white car, but it is discarded because it does not meet the threshold.  This is where the frame to frame "memory" comes into play.  In the video, if we have a situation like this, the detections from the previous 10 frames should carry over to maintain the bounding box.

## Applying the Pipeline to Video
In order to apply this pipeline to the video, I created a new function and a VehicleDection class that stores the detections for the previous 10 frames.  The threshold value is set 10 an initial vale of 2, and increases by 2 for every frame that is "remembered" in the vehicle detection class.  The final video output can be seen here.

[Final Project Video](https://youtu.be/kYGCon68wII)

## Discussion
Wile working through this project I ran into a lot of issues with classifier accuracy in regard to false positives.  I first tried the LAB color space for the HOG features, but after switching to YUV I noticed a lot fewer false positives.  I also realized (a bit too late) that I was using far too many windows in my sliding window search application.  I had to set a very high threshold for the heat-map in order to filter out a all of the false positives.  If I were to continue this project I would implement some hard negative mining on the false positives with the hopes of re-training the classifier.  Overall I found this to be a very interesting implementation, but the speed is clearly not viable for a real-time application.  I am also very interested in seeing how a deep learning detector like SSD or YOLO would compare to this approach.  I would imagine the deep learning approach would be superior in almost every way - but it would be an interesting benchmark.  The one major downfall that I noticed with this implementation was the inability to distinguish between two cars that were close or overlapping.  This is obviously due to the heat-map implementation, but I still think this would be a difficult problem to solve.
