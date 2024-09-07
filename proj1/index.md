# CS 180 Project 1 - Ramya Chitturi

## Project Overview

Sergei Mikhailovich Prokudin-Gorskii was a Russian photographer. He traveled across the vast Russian Empire and took color photographs of everything he saw. Since there was no way to print color photos at the time, he recorded three exposures of every scene onto a glass plate using a red, a green, and a blue filter. Years later, his RGB glass plate negatives were found, and the Library of Congress was able to digitize these photographs and release them to the public.

## Approach

First, I isolated the blue, green, and red plates (in that order) by dividing the image into thirds lengthwise. I removed 10% of the border from each side, using only the middle 80% of the image to minimize any errors from the borders. For each of the basic/smaller images, I tested displacements in the range of [-15, 15) in both the x and y directions. I will talk about the metrics used later, but I would keep track of the best score seen so far. If a displacement would result in a better score, I would update the best score and displacement accordingly.

For the bigger images, going through all of the possible displacements is too slow and consuming. Thus, I implemented an image pyramid, as per the directions. My pyramid had 5 levels. First, I shrunk down the image by 2^5 (rescaling the image to be half its size for each pyramid level). Once I had the smallest image, I checking for the starting displacement using 5% the image size. After, I doubled the size of the image and also the displacement. From there, I checked for an updated displacement in the [-5, 5) range and added the new displacement to the previous value. 

### Alignment Metrics
I tried 3 main alignment metrics, with the fourth being part of the bells and whistles. 

1. Sum of squared differences/Euclidean distance (SSD): As suggested, I tried using SSD first. 
2. Sum-absolute error (SAE): This worked better for some of the larger images than SSD, especially emir.tif.
3. Normalized cross correlation (NCC): Using NCC helped improve the results of some of my larger images with the pyramid. To implement, I first normalized the values of the red or green images before computing the dot product sum with the blue image. I also negated the final metric so that I could continue to find the minimum score for the optimal displacement.
4. SSIM (see below)

The final results of all of the given images are provided at the end.

## Bells and Whistles

### Structural Similarity
I used the structural similarity index (SSIM) at the end, mainly to align the emir image. NCC worked much more quickly than SSIM and produced fairly similar results for the rest of the images. I used the `structural_similarity` function from `skimage`'s `metrics` package. I also negated this result so that I could continue to minimize the score. SSIM works better for emir than the other 3 metrics because it operates in the L\*a\*b\* color space rather than RGB, and emir has very little information in the red and green image channels. Below is the result of emir.tif with the four metrics I tried:

| Emir with SAE | Emir with SSD | Emir with NCC | Emir with SSIM
| :---: |  :----: | :---: | :---: |
| ![](media/out_emir_sae.jpg) | ![](media/out_emir_ssd.jpg) | ![](media/out_emir_ncc.jpg) | ![](media/emir_SSIM.jpg) |
| G: (40, 26) <br> R: (88, 37) | G: (37, 20) <br> R: (101, -263) | G: (37, 20) <br> R: (101, -263) | G: (57, 13) <br> R: (121, 25) |

### Automatic Contrasting
I first rescaled the pixel values to be in the 0 to 1 range. Then I applied the main function I used, which was `equalize_adapthist` from `skimage`'s `exposure` package. Then, I rescaled the image back to values from 0 to 255 and converted it to the `uint8` type. Below is a selection of images before and after applying the automatic contrasting:

| Name | Without contrast | With contrast |
| :---: | :----: | :----: |
| Church | ![](media/out_church1.jpg) | ![](media/out_church_contrast1.jpg) |
| Three Generations | ![](media/out_threegenerations.jpg) | ![](media/out_threegenerations_contrast.jpg) |

## Final Results

The following images were generated using the NCC metric.

| Name | Image and Offset | Name | Image and Offset |
| :---: |  :----: | :---: | :---: |
| Cathedral | ![](media/out_cathedral.jpg) <br> G: (5, 2) <br> R: (12, 3) | Church | ![](media/out_church1.jpg) <br> G: (40, 26) <br> R: (88, 37) |
| Emir | ![](media/out_emir_ncc.jpg) <br> G: (37, 20) <br> R: (101, -263) | Harvesters | ![](media/out_harvesters.jpg) <br> G: (47, 14) <br> R: (118, 8) |
| Icon | ![](media/out_icon.jpg) <br> G: (34, 15) <br> R: (79, 21) | Lady | ![](media/out_lady_nocontrast.jpg) <br> G: (33, 8) <br> R: (96, 6) |
| Melons | ![](media/out_melons.jpg) <br> G: (82, 2) <br> R: (179, 8) | Monastery | ![](media/out_monastery.jpg) <br> G: (-3, 2) <br> R: (3, 2) |
| Onion Church | ![](media/out_onionchurch1.jpg) <br> G: (40, 26) <br> R: (97, 33) | Sculpture | ![](media/out_sculpture.jpg) <br> G: (30, -8) <br> R: (143, -24) |
| Self-Portrait | ![](media/out_selfportrait.jpg) <br> G: (70, 25) <br> R: (173, 34) | Three Generations | ![](media/out_threegenerations.jpg) <br> G: (53, 11) <br> R: (118, 10) |
| Tobolsk | ![](media/out_tobolsk.jpg) <br> G: (3, 3) <br> R: (6, 3) | Train | ![](media/out_train.jpg) <br> G: (41, 0) <br> R: (97, 10) |
