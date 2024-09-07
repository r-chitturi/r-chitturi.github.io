# CS 180 Project 1 - Ramya Chitturi

## Project Overview

Sergei Mikhailovich Prokudin-Gorskii was a Russian photographer. He traveled across the vast Russian Empire and took color photographs of everything he saw. Since there was no way to print color photos at the time, he recorded three exposures of every scene onto a glass plate using a red, a green, and a blue filter. Years later, his RGB glass plate negatives were found, and the Library of Congress was able to digitize these photographs and release them to the public.

## Approach

First, I isolated the blue, green, and red plates (in that order) by dividing the image into thirds lengthwise. I removed 10% of the border from each side, using only the middle 80% of the image to minimize any errors from the borders. For each of the basic/smaller images, I tested displacements in the range of [-15, 15) in both the x and y directions. I will talk about the metrics used later, but I would keep track of the best score seen so far. If a displacement would result in a better score, I would update the best score and displacement accordingly.

For the bigger images, going through all of the possible displacements is too slow and consuming. Thus, I implemented an image pyramid, as per the directions. My pyramid had 5 levels. First, I shrunk down the image by 2^5 (rescaling the image to be half its size for each pyramid level). Once I had the smallest image, I checking for the starting displacement using 5% the image size. After, I doubled the size of the image and also the displacement. From there, I checked for an updated displacement in the [-5, 5) range and added the new displacement to the previous value. 

### Alignment Metrics
I tried 3 main alignment metrics, with the fourth being part of the bells and whistles. 

1. Sum of squared differences/Euclidean distance (SSD): As suggested, I tried using SSD first. 
2. Sum-absolute error (SAE): This worked better for some of the larger images than SSD. However, the cathedral image was still not aligned when I used this metric.
3. Normalized cross correlation (NCC): Using NCC helped my cathedral image align a lot better, and it also improved the results of some of my larger images with the pyramid. To implement, I first normalized the values of the red or green images before computing the dot product sum with the blue image. I also negated the final metric so that I could continue to find the minimum score for the optimal displacement.

## Bells and Whistles

### Structural Similarity
I used the structural similarity index (SSIM) at the end, mainly to align the emir image. NCC worked much more quickly than SSIM and produced fairly similar results for the rest of the images. I used the `structural_similarity` function from `skimage`'s `metrics` package.

### Automatic Contrasting
I first rescaled the pixel values to keep them in the 0 to 1 range. Then I applied the main function I used, which was `equalize_adapthist` from `skimage`'s `exposure` package. Then, I rescaled the image back to values from 0 to 255 and converted it to the `uint8` type.

## Final Results
