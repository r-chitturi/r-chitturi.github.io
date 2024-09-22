# Fun with Filters and Frequencies

## Part 1 - Fun with Filters

### Part 1.1: Finite Difference Operator
I used the following finite difference operators: 
`dx = np.array([[1, -1]])` and `dy = np.array([[1], [-1]])`. I convolved these with the cameraman input image by using `scipy.signal.convolve2d` with `mode='same'`. I combined the partial derivatives by squaring each, adding them together, and then taking the square root. The threshold for my binary image was 35.

| | dx | dy |
| :----: | :----: | :----: |
| Derivative | ![](media/out_1.1/dx_derivative.jpg) | ![](media/out_1.1/dy_derivative.jpg) |
| Binarized | ![](media/out_1.1/dx_binarized.jpg) | ![](media/out_1.1/dy_binarized.jpg) |

| Combined Gradient | Combined Binarized |
| :----: | :----: |
| ![](media/out_1.1/gradient_magnitude.jpg) | ![](media/out_1.1/gradient_magnitude_binarized.jpg) |

### Part 1.2: Derivative of Gaussian (DoG) Filter
I used the Gaussian kernels to blur the images before using the finite difference operator to generate the partial derivatives. Here, `kernel_size = 10` and `sigma = kernel_size / 6` Then, we combine them into a single edge image using the same idea from 1.1. 

| | dx | dy |
| :----: | :----: | :----: |
| Derivative | ![](media/out_1.2/dx_gaussian.jpg) | ![](media/out_1.2/dy_gaussian.jpg) |
| Binarized | ![](media/out_1.2/dx_binarized.jpg) | ![](media/out_1.2/dy_binarized.jpg) |

| Combined Gradient | Combined Binarized |
| :----: | :----: |
| ![](media/out_1.2/gaussian_magnitude.jpg) | ![](media/out_1.2/gaussian_magnitude_binarized.jpg) |


#### Comparing Finite Difference and DoG
The results are similar except for small variations in length and shape of smaller edges due to noise. The main difference is that the edges of the DoG filter are thicker and also a bit smoother. Since the Gaussian filter is a low-pass filter, this makes sense since less high frequency components mean less noise and better edge detection.

| Binarized Finite Difference | Binarized Derivative of Gaussian |
| :----: | :----: |
| ![](media/out_1.1/gradient_magnitude_binarized.jpg) | ![](media/out_1.2/gaussian_magnitude_binarized.jpg) |


## Part 2 - Fun with Frequencies

### Part 2.1: Image "Sharpening"
To sharpen an image, we convolve the input image with a gaussian kernel. This acts as a low-pass filter, allowing us to get rid of the higher frequencies. To get the high-frequency details, we use the following operation: `details = target_im - blurred_im`. This removes the lower frequencies from the original image. We then choose some alpha in order to create the final sharpened image using `sharpened_result = target + alpha*details`. In the following results, `alpha = 1`.

| Original Image | Sharpened Image | Details |
| :----: | :----: | :----: |
| ![](media/out_2.1/taj.jpg) | ![](media/out_2.1/sharpened_taj.jpg) | ![](media/out_2.1/sharpened_taj_details.jpg) |
| ![](media/out_2.1/tree.jpg) | ![](media/out_2.1/sharpened_tree.jpg) | |

Next, we blur the image of this orange cat using `kernel_size = 10` and `sigma = kernel_size / 6`. We sharpen the image normally with `alpha = 1`, and then we sharpen the image after blurring with `alpha = 2`.

| Original Image | Sharpened Image | Blurred Image | Blurred then Sharpened |
| :----: | :----: | :----: | :----: |
| ![](media/out_2.1/orangecat.jpg) | ![](media/out_2.1/sharpened_orangecat.jpg) | ![](media/out_2.1/orangecat_blur.jpg) | ![](media/out_2.1/orangecat_blur_then_sharpened.jpg) |

### Part 2.2: Hybrid Images
We take in two input images, `im_high` and `im_low`, and align them using the provided function. We use a gaussian blur on the high frequency image using `kernel_size = 6 * sigma_high` to generate `blurred_im_high`. Then, we get the high frequency image using `high = im_high - blurred_im_high`. After that, we blur the low frequency image using `kernel_size = 6 * sigma_low` to get `blurred_im_low`. We average the pixels of `high` and `blurred_im_low` to get our hybrid image!

| Low Frequency Image | High Frequency Image | Hybrid Image |
| :----: | :----: | :----: |
| ![](media/out_2.2/DerekPicture.jpg) <br> Derek | ![](media/out_2.2/nutmeg.jpg) <br> Nutmeg | ![](media/out_2.2/hybrid_derek_nutmeg.jpg) <br> sigma_low = 8 <br> sigma_high = 11 |
| ![](media/out_2.2/basketball.jpg) <br> Basketball | ![](media/out_2.2/orangecat.jpg) <br> Orange Cat | ![](media/out_2.2/hybrid_cat_basketball.jpg) <br> sigma_low = 4 <br> sigma_high = 10 |
| ![](media/out_2.2/tragedy.png) <br> Tragedy | ![](media/out_2.2/comedy.png) <br> Comedy | ![](media/out_2.2/hybrid_comedy_tragedy.jpg) <br> sigma_low = 3 <br> sigma_high = 6 |

The following hybrid image failed, likely because of the paper plane's thick black lines compared to the airplane.

| Low Frequency Image | High Frequency Image | Hybrid Image |
| :----: | :----: | :----: |
| ![](media/out_2.2/paperplane.jpg) <br> Paper Plane | ![](media/out_2.2/plane.jpg) <br> Airplane | ![](media/out_2.2/hybrid_plane_paperplane_bad.jpg) <br> sigma_low = 4 <br> sigma_high = 10 |

#### Fourier Analysis
I liked the way that comedy and tragedy turned out, so I performed Fourier analysis below.

|  |  |
| :----: | :----: |
| ![](media/out_2.2/hybrid_fft_low.jpg) <br> Paper Plane FFT | ![](media/out_2.2/hybrid_fft_high.jpg) <br> Airplane FFT |
| ![](media/out_2.2/hybrid_fft_hybridlow.jpg) <br> Low Frequency FFT | ![](media/out_2.2/hybrid_fft_hybridhigh.jpg) <br> High Frequency FFT |

| |
| :----: |
| ![](media/out_2.2/hybrid_fft_hybrid.jpg) <br> Hybrid Image FFT |

#### Part 2.2 - Adding Color (Bells and Whistles)
I experimented with using color on Derek and Nutmeg.

| No Color | Low Frequency Color | High Frequency Color | Both Frequency Color |
| :----: | :----: | :----: | :----: |
| ![](media/out_2.2/hybrid_derek_nutmeg.jpg) | ![](media/out_2.2/hybrid_derek_nutmeg_color_low.jpg) | ![](media/out_2.2/hybrid_derek_nutmeg_color_high.jpg) | ![](media/out_2.2/hybrid_derek_nutmeg_color.jpg) |

The color really only helps the low frequency image, considering that just using color on Nutmeg (high frequency) makes no difference. Using color on the low frequency image or both images makes it look different but not necessarily a lot better.

### Part 2.3: Gaussian and Laplacian Stacks

We create Gaussian and Laplacian stacks for the input images. At every level of the Gaussian stack, we blur the previous image level with a kernel to get the current level’s image. We are using a Gaussian stack, which means that the image size remains the same. We can generate the Laplacian stack by using the Gaussian stack. At level `i`, `l_stack[i] = g_stack[i] - g_stack[i+1]`. At the last level `i`, `l_stack[i] = g_stack[i]`. This allows both stacks to have the same length. Each stack has 5 layers/levels.

Here are levels 0 through 4 of the input images' Gaussian and Laplacian stacks. The Laplacian stack images here are normalized over the entire image (for visual purposes), but the un-normalized versions are using during computation.

| |
| :----: |
| ![](media/out_2.4/2.4_apple_gstack.jpg) <br> Apple Gaussian Stack|
| ![](media/out_2.4/2.4_apple_lstack.jpg) <br> Apple Laplacian Stack|
| ![](media/out_2.4/2.4_orange_gstack.jpg) <br> Orange Gaussian Stack|
| ![](media/out_2.4/2.4_orange_lstack.jpg) <br> Orange Laplacian Stack|

### Part 2.4: Multiresolution Blending

After I had the Gaussian and Laplacian stacks for the 2 input images as well as the Gaussian stack for the mask, I could blend everything together. I created the blended image by doing: `blended_stack = g_stack_mask * l1_stack + (1 - g_stack_mask) * l2_stack`.
In addition to the oraple, I blended two other images (where the baby + bread blended image has an irregular mask). I used `kernel_size = 12` and `N = 5`, with the exception of the Gaussian stack of the mask which had `kernel_size = 60`.

| Image 1 | Image 2 | Mask | Blended Image |
| :----: | :----: |  :----: | :----: |
| ![](media/out_2.4/apple.jpeg) | ![](media/out_2.4/orange.jpeg) | ![](media/out_2.4/2.4_oraple_mask.jpg) | ![](media/out_2.4/2.4_oraple1.jpg) |
| ![](media/out_2.4/watermelon_slice.jpg) | ![](media/out_2.4/orange_slice.jpg) | ![](media/out_2.4/2.4_oraple_mask.jpg) | ![](media/out_2.4/2.4_orangemelon.jpg) |
| ![](media/out_2.4/baby.jpg) | ![](media/out_2.4/bread.jpg) | ![](media/out_2.4/2.4_baby_bread_mask.jpg) Irregular Mask Example | ![](media/out_2.4/2.4_baby_bread.jpg) |

#### Figure 3.42 Recreated

I created Figure 3.42 from the textbook on the oraple.

| |
| :----: |
| ![](media/out_2.4/2.4_oraple_level0.jpg) <br> Level 0 |
| ![](media/out_2.4/2.4_oraple_level2.jpg) <br> Level 2 |
| ![](media/out_2.4/2.4_oraple_level4.jpg) <br> Level 4 |
| ![](media/out_2.4/2.4_oraple_levelcollapsed.jpg) <br> Collapsed |

Here is the final result again:

| Image 1 | Image 2 | Blended Image |
| :----: | :----: | :----: |
| ![](media/out_2.4/apple.jpeg) | ![](media/out_2.4/orange.jpeg) | ![](media/out_2.4/2.4_oraple1.jpg) |