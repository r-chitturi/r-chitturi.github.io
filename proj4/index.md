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
Here was my method to warp/project `im1` onto `im2`.
1. The calculations above were to get the estimated homography matrix H. We calculate H based on `im1_points` to `im2_points`.
2. We compute the bounding box of the final image size by taking the 4 corners of `im1`. Then we homogenize the coordinates into a 4x3 matrix. We then warp this bounding box by multiplying the matrix with `H.T`.
3. Using `skimage.draw.polygon` like in Project 3, we get the points that are within the bounding box. This polygon is where the final image will be placed.
4. We then interpolate the pixel values in the warped image.

This procedure allows us to warp the source image's points and align them with the correspondence points in the target image.

## Image Rectification

Using the warping function from before, we can rectify images that contain a rectangle shape. I picked 4 points for the corners in the image that represent the rectangular object. Then, we compute the homography between those selected points and a rectangle of a hardcoded size, like `[(100, 100), (250, 100), (100, 350), (250, 350)]`. This allows us to warp the rectangular object so it faces the camera. The final images are warped weirdly in the background, but this ensures that the rectification of the selected object occurs properly. If the object is isolated by cropping out the background, we can more clearly see that it is now a rectangle.

| Original Image | Image After Rectification |
| :----: | :----: |
| <img src="media/laptop_torectify.jpg" width="300"/> | <img src="media/laptop_rectified.jpg" width="300"/> |
| <img src="media/ipad_torectify.jpg" width="300"/> | <img src="media/ipad_rectified.jpg" width="300"/> |


## Blend the images into a mosaic

To generate the mosaic, I defined the corners of both input images and warped the corners of `im1` using H. I then computed the bounding box for both images, which is where the final warped image will lay. I then calculated my new image dimensions based on the difference between the maximum and minimum x and y coordinates. I created a translation matrix to shift an image into a positive coordinate space. I then warped both images to the new dimensions and using the translation matrix (where `im1` was warped by `translation_matrix @ H`). Naively, I took the maximum pixel value at a point to combine the two images into the final mosaic.

### Two-Band Blending

I used two-band blending with a distance transform mask to blend my images together.
1. I computed the distance transform mask for both my images using `scipy.ndimage.distance_transform_edt` to find the Euclidean distance to the nearest edge. I normalized each mask.
2. Similar to Project 2, I found the low and high frequencies of each image. To get the low frequency image, I used a blurring Gaussian filter with kernel size 7 and sigma 2. I combined the two low frequencies by doing `(lowPass1*mask1 + lowPass2*mask2) / (mask1 + mask2 + 1e-8)` wherever either mask was nonzero. Adding `1e-8` prevented division by 0.
3. The high frequency image was obtained by subtracting the original image by the low frequency image. I combined the high frequencies by choosing the corresponding pixel where the mask is greater.
4. I added the two blends together for the final blended mosaic.

Here are warped images and blended mosaics (below all warped images) for my 3 examples. The original images for the mosaics can be found at the top of the website.

| Warped Image 1 | Warped Image 2 |
| :----: | :----: |
| <img src="media/breezeway1_bigger_warp1.jpg" width="400"/> | <img src="media/breezeway2_bigger_warp2.jpg" width="400"/> |
| <img src="media/kitchen12_noblend_bigger_warp1.jpg" width="400"/> | <img src="media/kitchen12_noblend_bigger_warp2.jpg" width="400"/> |
| <img src="media/souvenir1_warped.jpg" width="400"/> | <img src="media/souvenir2_warped.jpg" width="400"/> |

<br> <img src="media/breezeway12_bigger_twoband.jpg" width="600" style="display: block; margin: 0 auto;"/>
<br> <img src="media/kitchen12_twoband_bigger1.jpg" width="600" style="display: block; margin: 0 auto;"/>
<br> <img src="media/souvenir12_twoband.jpg" width="600" style="display: block; margin: 0 auto;"/>

Here is an example mask image from a warped kitchen image.

| Warped Image | Distance Transform Mask |
| :----: | :----: |
| <img src="media/kitchen12_noblend_bigger_warp2.jpg" width="300"/> | <img src="media/kitchen2_mask_correct.jpg" width="300"/> |

### Laplacian Blending

I also used the Gaussian and Laplacian stacks from Project 2 in order to blend the two images in my mosaics together. I used 5 levels with a kernel size of 12 and sigma of 2. The exception was for the mask's Gaussian stack, where I used a kernel size of 60 for extra blending. Otherwise, there was a harsher line where my mask was. Here is an example that compares the kitchen mosaic using two-band blending and Laplacian blending.

| Using Laplacian Stacks | Using Two-Band Blending |
| :----: | :----: |
| <img src="media/kitchen12_laplacianblend.jpg" width="300"/> | <img src="media/kitchen12_twoband_bigger1.jpg" width="300"/> |

Here is the mask I used for Laplacian blending. I created binary masks for both images where the pixel in the image was greater than 0. Then, I used `distance_transform_edt` on both masks. I combined the two masks by seeing where the mask after the distance transform for `im1` was greater than the transform mask for `im2`.

<br> <img src="media/kitchen12_laplacianmask.jpg" width="300"/>

## Reflection - Part A

It was interesting to see how we had to translate the images to fit within the bounding box and how we could use matrix multiplication to apply both the homography and translation to an image. It was also cool to apply aspects of Project 2, like the low/high pass filters and Laplacian pyramids, to blend the mosaic images together.