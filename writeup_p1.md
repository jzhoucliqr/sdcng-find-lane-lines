# **Finding Lane Lines on the Road** 

The goals / steps of this project are the following:

* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

Here are the final results on test images:

![solidWhiteCurve] (./test_images_output/solidWhiteCurve.png)
![solidWhiteRight] (./test_images_output/solidWhiteRight.png "solidWhiteRight")
![solidYellowCurve] (./test_images_output/solidYellowCurve.png "solidYellowCurve")
![solidYellowCurve2] (./test_images_output/solidYellowCurve2.png "solidYellowCurve2")
![solidYellowLeft] (./test_images_output/solidYellowLeft.png "solidYellowLeft")
![whiteCarLaneSwitch] (./test_images_output/whiteCarLaneSwitch.png "whiteCarLaneSwitch")

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps:

1. convert to grayscale
2. apply gassian blur
3. do canny edege detection
4. identify region of interest with fillpoly
5. use hough transformation to extract and extrapolate lanes

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by figure out the average slope of both lanes, and average position (x, y) of the data points. Then I have the y-axis of both the top and bottom points, use formation: x2 = x1 + (y2-y1)/slope, we can calculate the x-axix of both the top and bottom points. Then we draw direct lines use the top and bottom points. 

#### Details about how draw_line function filter out noise data 
First I tried to just use the average position and average slope of left lines and right lines each, 
to extrapolate the whole line. But it doesn't work well on the yellow left video. I tried couple of different ways to figure out what's the noise. Eventually figured out just by looking at the generated video that the noise comes from some short segments on the other side the lane. 

Then I tried different ways to remove the noise. First tried to calculate the average of slope, then remove
the ones which differs too much with the average. But this method doesn't work well because its hard to
fine tune the threshold, sometimes all the lines have larger difference so it turns out all data got removed.

Then I went to a different but more simple way, just rely on the x axis of the data points together with the slope.
So if the line have a negative slope, it should be on the left line, but if it's x-axis larger than
the middle point of the poly, then it should be noise thus removed. Same idea apply to the right lane. The algorithm is like this:

```

    for line in lines:
        for x1,y1,x2,y2 in line:
            slope = (y2-y1)/(x2-x1)
            if slope < 0:
                # if x larger than middle, then skip
                if x1 < 500 and x2 < 500:
                    left_lines.append(line)
                    left_slope.append(slope)
            else:
                # if x smaller than middle, then skip
                if x1 > 450 and x2 > 450:
                    right_lines.append(line)
                    right_slope.append(slope)

```

### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when the noise data point lies within the same side of the lane, then my algorithm couldn't filter out the noise. 

Also that my pipeline is using some fixed value for the fillPoly, this should be somehow dynamically calculated.


### 3. Suggest possible improvements to your pipeline

A possible improvement would be combine the two ideas which I've tried. First we filter out using my current solution. Then for the data points which have similar slope and also lies within the same side of the line, we can do filter again based on the average of slope, those who have larger difference from the average slope should be filtered out. 

Another way is to use machine learning methods, like we treat each line with y=mx+b, then logistic regression or svm to find the two parameters m and b.  But the problem is we don't have enough data to train the algorithm.
