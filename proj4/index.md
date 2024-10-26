<script type="text/javascript" async 
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/3.2.0/es5/tex-mml-chtml.js">
</script>
<script type="text/javascript" async>
  window.MathJax = {
    tex: {
      inlineMath: [['$', '$'], ['\\(', '\\)']],
      displayMath: [['$$', '$$'], ['\\[', '\\]']]
    }
  };
</script>

# Stitching Photo Mosaics - Part A

## Taking Images

Here are the images that I used in this project for the mosaics.

| Soda Breezeway 1 | Soda Breezeway 2 |
| :----: | :----: |
| <img src="media/breezeway1.jpg" width="300"/> | <img src="media/breezeway2.jpg" width="300"/> |

| Souvenir Coffee 1 | Souvenir Coffee 2 |
| :----: | :----: |
| <img src="media/im2_souvenir.jpg" width="300"/> | <img src="media/im1_souvenir.jpg" width="300"/> |

| My Kitchen 1 | My Kitchen 2 |
| :----: | :----: |
| <img src="media/kitchen1.jpg" width="300"/> | <img src="media/kitchen2.jpg" width="300"/> |

## Recover Homographies

First, I defined my correspondence points between pairs of images. For example, here are my correspondences for the Souvenir images.

| Souvenir Coffee 1 Correspondences | Souvenir Coffee 2 Correspondences |
| :----: | :----: |
| <img src="media/souvenir1_correspondences.jpg" width="300"/> | <img src="media/souvenir2_correspondences.jpg" width="300"/> |

Let the points in the source image (the first image) be $(x_{i}, y_{i})$ and the corresponding points in the target image be $(x'_{i}, y'_{i})$. We want the following 3x3 transformation matrix.
<br>

$$
\begin{bmatrix}
a & b & c \\
d & e & f \\
g & h & i
\end{bmatrix}
\begin{bmatrix}
x \\ 
y \\ 
1
\end{bmatrix}
= 
\begin{bmatrix}
wx' \\ 
wy; \\ 
w
\end{bmatrix}
$$ 

We can then expand this to solve the system of equations:
$$
\begin{cases}
ax + by + c = wx' \\
dx + ey + f = wy' \\
gx + hy + 1 = w
\end{cases}
$$

This eventually simplifies to:
$$
\begin{cases}
ax + by + c - gxx' - hyx' = x' \\
dx + ey + f - gxy' - hyy' = y' \\
\end{cases}
$$

We can finally solve for $x'$ and $y'$ using least squares. Since we have an overconstrained system, we can just stack all the corresponding points in the matrix and constant vector. We have the following equation:
<br>

$$
\begin{bmatrix}
x & y & 1 & 0 & 0 & 0 & -xx' & -yx' \\ 
0 & 0 & 0 & x & y & 1 & -xy' & -yy'
\end{bmatrix}
\begin{bmatrix}
a \\ 
b \\ 
c \\ 
d \\ 
e \\ 
f \\ 
g \\ 
h \\ 
\end{bmatrix}
=
\begin{bmatrix}
x' \\ 
y' \\
\end{bmatrix}
$$ 

## Warp the Images
The goal is to warp or project `im1` onto `im2`. We can wrp the source image's points and align them with the correspondence points in the target image. The calculations above were to get the estimated homography matrix H. I calculated H based on `im1_points` to `im2_points`. I computed the bounding box of the final image size by taking the 4 corners of `im1`. I then homogenized the coordinates into a 4x3 matrix so we can warp the bounding box by multiplying it with the transpose of `H`. By utilizing `skimage.draw.polygon`, similar to the method used in Project 3, I got the points that are within the bounding box. This polygon is where the final image will be placed. I then interpolated the pixel values in the warped image.

## Image Rectification

Using the warping function from before, we can rectify images that contain a rectangle shape. I picked 4 points for the corners in the image that represent the rectangular object. Then, we compute the homography between those selected points and a rectangle of a hardcoded size, like `[(100, 100), (250, 100), (100, 350), (250, 350)]`. This allows us to warp the rectangular object so it faces the camera. If the object is isolated by cropping out the warped background, we can see that it is now a rectangle.

| Original Image | Image After Rectification |
| :----: | :----: |
| <img src="media/laptop_torectify.jpg" width="300"/> | <img src="media/laptop_rectified.jpg" width="300"/> |
| <img src="media/ipad_torectify.jpg" width="300"/> | <img src="media/ipad_rectified.jpg" width="300"/> |


## Blend the images into a mosaic

To generate the mosaic, I defined the corners of both input images and warped the corners of `im1` using H. I then computed the bounding box for both images. I then calculated my new image dimensions based on the difference between the maximum and minimum x and y coordinates. I created a translation matrix to shift the image into a positive coordinate space. I then warped both images to the new dimensions using the translation matrix (where `im1` was warped by `translation_matrix @ H`). Naively, I took the maximum pixel value at a point to combine the two images into the final unblended mosaic.

Here are warped images and unblended mosaics (below all warped images) for my 3 examples.

| Warped Image 1 | Warped Image 2 |
| :----: | :----: |
| <img src="media/breezeway1_bigger_warp1.jpg" width="400"/> | <img src="media/breezeway2_bigger_warp2.jpg" width="400"/> |
| <img src="media/kitchen12_noblend_bigger_warp1.jpg" width="400"/> | <img src="media/kitchen12_noblend_bigger_warp2.jpg" width="400"/> |
| <img src="media/souvenir1_warped.jpg" width="400"/> | <img src="media/souvenir2_warped.jpg" width="400"/> |

<br> <img src="media/breezeway12_bigger_noblend.jpg" width="600" style="display: block; margin: 0 auto;"/>
<br> <img src="media/kitchen12_noblend_bigger.jpg" width="600" style="display: block; margin: 0 auto;"/>
<br> <img src="media/souvenir12_noblend.jpg" width="600" style="display: block; margin: 0 auto;"/>

### Two-Band Blending

I used two-band blending with a distance transform mask to blend my images together. I generated the distance transform mask for both my images using `scipy.ndimage.distance_transform_edt` to find the Euclidean distance to the nearest edge, which was then normalized. Similar to the approach taken in Project 2, I found the low and high frequencies of each image. To get the low frequency image, I used a blurring Gaussian low-pass filter with `kernel_size=7` and `sigma=2`. I combined the two low-frequency images by doing `(lowPass1*mask1 + lowPass2*mask2) / (mask1 + mask2 + 1e-8)` wherever either mask was nonzero. Adding `1e-8` prevented division by 0 but still ensured a very small pixel value. The high frequency image was obtained by subtracting the original image by the low frequency image. To merge the high frequency images, I selected the corresponding pixels based on the greater value in the mask. For the final blended mosaic, I added the two blends together.

Here are the blended mosaics (using two-band blending) for my 3 examples. The original images for the mosaics can be found at the top of the website, and the warped images and unblended mosaics are above.

<br> <img src="media/breezeway12_bigger_twoband.jpg" width="600" style="display: block; margin: 0 auto;"/>
<br> <img src="media/kitchen12_twoband_bigger1.jpg" width="600" style="display: block; margin: 0 auto;"/>
<br> <img src="media/souvenir12_twoband.jpg" width="600" style="display: block; margin: 0 auto;"/>

Here is an example mask image from a warped kitchen image.

| Warped Image | Distance Transform Mask |
| :----: | :----: |
| <img src="media/kitchen12_noblend_bigger_warp2.jpg" width="300"/> | <img src="media/kitchen2_mask_correct.jpg" width="300"/> |

### Laplacian Blending

I also tried using the Gaussian and Laplacian stacks from Project 2 in order to blend the two images in my mosaics together. I used 5 levels with `kernel_size=12` and `sigma=2`. The exception was for the mask's Gaussian stack, where I used a `kernel size=60` for extra blending. Otherwise, there was a harsher line where my mask was. Here is an example that compares the kitchen mosaic using two-band blending and Laplacian blending.

| Using Laplacian Stacks | Using Two-Band Blending |
| :----: | :----: |
| <img src="media/kitchen12_laplacianblend.jpg" width="300"/> | <img src="media/kitchen12_twoband_bigger1.jpg" width="300"/> |

Here is the mask I used for Laplacian blending. I created binary masks for both images where the pixel in the image was greater than 0. Then, I used `distance_transform_edt` on both masks. I combined the two masks by seeing where the mask after the distance transform for `im1` was greater than the transform mask for `im2`.

<br> <img src="media/kitchen12_laplacianmask.jpg" width="300"/>

I prefer my two-band blending output, so I chose to use that for all of my final mosaics.

## Reflection - Part A

It was interesting to see how we had to translate the images to fit within the bounding box and how we could use matrix multiplication to apply both the homography and translation to an image. It was also cool to apply aspects of Project 2, like the low/high pass filters and Laplacian pyramids, to blend the mosaic images together.

# Feature Matching for Autostitching - Part B

## Harris Interest Point Detector
To find the interest points, we want to focus on the corners of the image. Using the Harris interest point detector provided, we can find the "peaks" in the matrix and use them as corners. Displayed below are the top 300 points found by the Harris detector, along with the Harris matrix for the first kitchen image.

| Top 300 Harris Points | Harris Matrix |
| :----: | :----: |
| <img src="media/partb/harris_top300_kitchen1.jpg" width="300"/> | <img src="media/partb/harrismatrix_kitchen1.jpg" width="300"/> |

## Adaptive Non-Maximal Suppression (ANMS)

ANMS resolves the issue of the selected points being too close together/clustered, as we can see in the top Harris points image above. ANMS allows us to identify the strongest corners from the image, but we can distribute the points across the image for better feature matching later on. We use the `dist2()` function provided to find pairwise distances between points. Then, we create a mask based on `c_robust` to ensure that (1) we keep corners that are both sufficiently spread out and (2) also have strong enough scores relative to each other. We sort the corners in decreasing order of radii size, with any points from before that did not satisfy our mask condition being set to infinity. After this minimization, we use these indices to sort the original list of corners passed in and return the top `num_corners`.

| All Corners | ANMS Corners |
| :----: | :----: |
| <img src="media/partb/allcoords_kitchen1.jpg" width="300"/> | <img src="media/partb/anms_top120_kitchen1.jpg" width="300"/> |

## Feature Descriptor Extraction

For each corner found by ANMS, we find a 40x40 pixel region around the point found. We then resize it to an 8x8 patch and normalize it by subtracting the mean and dividing by the standard deviation. We flatten the 8x8 region and stack it into an `Nx192` matrix (where each array flattened has length 64, multiplied by 3 due to the color channels) of flattened feature descriptors. We repeat this for both images.

Here are some example 8x8 feature descriptor patches found:

<img src="media/partb/patch3_extract_kitchen1.jpg" width="225"/>
<img src="media/partb/patch11_extract_kitchen1.jpg" width="225"/>
<img src="media/partb/patch16_extract_kitchen1.jpg" width="225"/>
<img src="media/partb/patch18_extract_kitchen1.jpg" width="225"/>

## Feature Matching

We have a set of features for both images from the previous part. Now, we need to match the features from each image together. First, we calculate the pairwise differences between each (flattened) feature descriptor. We find the sum of squared differences over the last dimension. We find the nearest-neighbor distances by sorting the SSD found before. We use Lowe's technique in order to minimize outliers. I used a Lowe's threshold of 0.5, which means that the nearest neighbor should be a significantly better match than the second-nearest neighbor. The matched points are shown below.

Here are some of the matches between feature descriptors.

<img src="media/partb/patch0_matched_kitchen.jpg" width="500" style="display: block; margin: 0 auto;"/>
<img src="media/partb/patch7_matched_kitchen.jpg" width="500" style="display: block; margin: 0 auto;"/>
<img src="media/partb/patch8_matched_kitchen.jpg" width="500" style="display: block; margin: 0 auto;"/>
<img src="media/partb/patch16_matched_kitchen.jpg" width="500" style="display: block; margin: 0 auto;"/>

Here are the following matching correspondences, which I got by indexing into the original detected corners.

| Threshold = 0.5 | Threshold = 0.8 |
| :----: | :----: |
| <img src="media/partb/corrpoints_matched_thresh05_kitchen12.jpg" width="500"/> | <img src="media/partb/corrpoints_matched_thresh08_kitchen12.jpg" width="500"/> |

## RANSAC

RANSAC is used to increase least-squares' robustness when computing the homography. The input points are the ANMS-determined points, where the matches are then filtered from Lowe's method before. We randomly sample 4 pairs of points and compute the homographies. We track the inliers, which are points that land within a given threshold. At each iteration of the algorithm, we see if there are more inliers than the previous best homography computed. If so, we update our best points accoridngly. This helps us minimize outliers. I used a threshold of 0.8 and 2500 iterations for my RANSAC algorithm. Below are the points matched by RANSAC for the first and second kitchen images.

<img src="media/partb/ransac_matched_kitchen12.jpg" width="600" style="display: block; margin: 0 auto;"/>

## Results

Here are the results of using RANSAC and automatic feature matching on the images from before. I similarly provided the warped images, unblended mosaics, and the two-band blended mosaics. I compare the mosaics created with manual correspondences from Part A and the mosaics created with automatic correspondences.

| Warped Image 1 | Warped Image 2 |
| :----: | :----: |
| <img src="media/partb/ransac_warp_kitchen1.jpg" width="400"/> | <img src="media/partb/ransac_warp_kitchen2.jpg" width="400"/> |
| <img src="media/partb/ransac_warp_souvenir1.jpg" width="400"/> | <img src="media/partb/ransac_warp_souvenir2.jpg" width="400"/> |
| <img src="media/partb/ransac_warp_breezeway1.jpg" width="400"/> | <img src="media/partb/ransac_warp_breezeway2.jpg" width="400"/> |

<br> <img src="media/partb/ransac_mosaic_noblend_kitchen12.jpg" width="600" style="display: block; margin: 0 auto;"/>
<br> <img src="media/partb/ransac_noblend_souvenir12.jpg" width="600" style="display: block; margin: 0 auto;"/>
<br> <img src="media/partb/ransac_noblend_breezeway12.jpg" width="600" style="display: block; margin: 0 auto;"/>

| Manual Correspondences | Automatic Correspondences |
| :----: | :----: |
| <img src="media/kitchen12_twoband_bigger1.jpg" width="600"/> | <img src="media/partb/ransac_blended_kitchen12.jpg" width="600"/> |
| <img src="media/souvenir12_twoband.jpg" width="600"/> | <img src="media/partb/ransac_blended_souvenir12_larger.jpg" width="600"/> |
| <img src="media/breezeway12_bigger_twoband.jpg" width="600"/> | <img src="media/partb/ransac_blended_breezeway12.jpg" width="600"/> |

## Reflection - Part B

It was really interesting to see how the automatic correspondences were more precise than my manual ones. There were small features that didn't align in my mosaics since I was slightly off, but there were resolved during Part B. The coolest part of this project is being able to plot the automatic correspondences side-by-side after each step to see how they improved and how outliers were minimized.