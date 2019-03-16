# **Finding Lane Lines on the Road** 

## Writeup




[//]: # (Image References)

[image1]: ./writeup/step1.png "Step 1"
[image2]: ./writeup/step2.png "Step 2"
[image3]: ./writeup/step3.png "Step 3"
[image3_2]: ./writeup/step3_2.png "Step 3 applied on the image"
[image4]: ./writeup/step4.png "Step 4"
[image5]: ./writeup/step5.png "Step 5"
[image5_2]: ./writeup/step5_2.png "Step 5_2"
[image6]: ./writeup/step6.png "Step 6"
[image7]: ./writeup/step7.png "Step 7"
[debug]: ./writeup/debug.png "Debugging"
[debug2]: ./writeup/debug2.png "Debugging"

---

### Pipeline

To identify the lanes every frame was pushed through 7 different stepts in the pipeline.

1. The image is blured with a kernel size 5. This is done to reduce fine noise
    ![Step 1][image1]
2. To easily detect edges in the image, a canny edge filter was set. The parameters were set in a way that the edges of the road are properly detected most of the time. Though there can be issues with extreme lighting.
    ![Step 2][image2]
3. Since most information on the image are actually not important to us, a mask was created to cover just the street. The size of the mask works really good for straight long streets. If there are extreme curves the mask may not cover the entire area of interest.
    ![Step 3][image3]
    ![Step 3][image3_2]
4. This step simply blacks out everything that is not covered by the mask and gets rid of the unimportant areas
    ![Step 4][image4]
5. Since the last step reduced the information to a minimum it is now possible to run the 'houghlines' algorithm. The output of the algorithm is drawn to the array 'line_image' and can be shown in the final image if the debug mode is on.
The parameters for the Hough-Line transformation were determined empirical and should work in most cases.
    ![Step 5][image5]
    ![Step 5][image5_2]
6. The line informations from step 5 can contain a lot of noise. This is especially true if you experience extreme light conditions. Therefor every line with a slope above 0.85 or below 0.4 was ignored. The resulting array contains mostly lines that either describes the right or the left lane. This array was split into one for either lane based on the sign of the slope. Normally you could use this information to calculate a linear regression but the resulting lines would still be quite unstable when the lighting changes or the line markings get interrupted for a little bit too long. To prevent shaky lane predictions, the slope and bias information from past frames are buffered and used to calculate an extrapolated line (The average of 100 different slope points is used what is equivalent to a 2-5 second long buffer )
    ![Step 6][image6]
7. The last steps consists of the merging of the lane information with the original image
    ![Step 7][image7]

If debugging is turned on at the beginning of the notebook, further information can  merged onto the image.
```python
logger.setLevel(logging.DEBUGGING)
```
![debug][debug]

The debug output is written to


```test_videos_output/debug```

![debug][debug2]

### Shortcomings 


This pipeline has multiple shortcomings:
- winding roads are a problem due to the static mask
- extreme light and shadow can confuse the system and the prediction may go wrong
- all parameter are adjusted for good weather and clear visibility. It will probably not work at night or when there is fog or rain.
- a lane change is not assumed
- it depends on lane markings on both sides!. Without them it can't detect its current lane properly.
- the mask and the adjustment just work for the current camera adjustment. If the camera gets move to a different location or gets a different lens, all parameters need to be check.
- the lane extrapolation is done by linear regression. This becomes an issue if the street is not straight.
