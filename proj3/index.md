# Face Morphing

## Part 1. Defining Correspondences

I used this [correspondence tool](https://cal-cs180.github.io/fa23/hw/proj3/tool.html) from a previous semester's student to find corresponding points between my two faces. I loaded in the points from the resulting JSON file and used `scipy.spatial.Delaunay` to produce `tri` for both images. Then I used `plt.triplot` on `tri.simplices` to view the resulting triangles.

| Input Images | Correspondences | Triangulation |
| :----: | :----: | :----: |
| <img src="media/kavya.png" width="300"/> | ![](media/q1_points_kavya.jpg) | ![](media/q1_triangles_kavya.jpg) |
| <img src="media/ramya.jpg" width="300"/> | ![](media/q1_points_ramya.jpg) | ![](media/q1_triangles_ramya.jpg) |

## Part 2. Computing the "Mid-way Face"

To create the midway image, I needed both images and their correspondence points, along with the warp amount and dissolve amount. The warp and dissolve amounts are both 0.5 for this part of the project. I computed the midway points by doing `(1.0 - warp_amt) * im1_points + warp_amt * im2_points`. Then, I computed the Delunay triangles. For each of the `N` corresponding triangles for both images, I computed the affine transformation for each and solved for the 6 unknowns. 

Then, I used `skimage.draw.polygon` to find all the points that were within the triangle. I multiplied these coordinates by the inverse affine matrix. I also clipped the resulting x and y values to ensure that they remained within the image. I didn't select many correpondence points below shoulder level, so this explains why the midway face there is more overlapped. 

| Input Image 1 (Kavya) | Input Image 2 (Ramya) | Midway Face |
| :----: | :----: | :----: |
| ![](media/kavya.png) | <img src="media/ramya.jpg" width="550"/> | ![](media/q1_midway_ramya_kavya.jpg) |

## Part 3. The Morph Sequence

I used 100 frames in my morph sequence. The warp amount and dissolve amount is determined by `i / 100`, where `i` is the current frame number. 

![](media/q2_morph_ramya_kavya.gif)

## Part 4. The "Mean face" of a population

I used the FEI Face Database as my dataset. I downloaded the points from `frontalshapes_manuallyannotated_46points` and the datasets from `frontalimages_spatiallynormalized`. I combined both folders (parts 1 and 2) into one final folder. To generate the average correspondence points of the population, I averaged the coordinate values. The images marked with an `a` had a neutral expression, while the images marked with a `b` had a happy/smiling expression. Thus, I generated two mean images so we could have separate ones per expression. For the mean population image, I used the points from each image and the average image to create the warped image. I used a warp amount of 0.5 and a dissolve amount of 0.

Here are some images (both neutral and happy) from the dataset that were warped to fit the average face shape.

| Original Neutral Image | Warped Neutral Image | Original Happy Image | Warped Happy Image |
| :----: | :----: | :----: |  :----: |
| ![](media/50a.jpg) | ![](media/q4_dataset_warp_50a.jpg) | ![](media/50b.jpg) | ![](media/q4_dataset_warp_50b.jpg) |
| ![](media/53a.jpg) | ![](media/q4_dataset_warp_53a.jpg) | ![](media/53b.jpg) | ![](media/q4_dataset_warp_53b.jpg) |
| ![](media/68a.jpg) | ![](media/q4_dataset_warp_68a.jpg) | ![](media/68b.jpg) | ![](media/q4_dataset_warp_68b.jpg) |

Here are the images of the mean faces, as well as me warped into the mean face shape and vice versa.

| Mean Face Neutral | Mean Face Happy |
| :----: | :----: |
| ![](media/q4_mean_face_neutral.jpg) | ![](media/q4_mean_face_happy.jpg) |

| Mean Face Neutral -> Ramya | Ramya -> Mean Face Neutral |
| :----: | :----: |
| ![](media/q4_avg_to_ramya.jpg) | ![](media/q4_ramya_to_avg.jpg) |

## Part 5. Caricatures: Extrapolating from the mean

To generate the caricature, I changed the warp amount to no longer be in between 0 and 1. The warp amount is factored into the midpoints as such: `(1.0 - warp_amt) * im1_points + warp_amt * im2_points`. I warped my face based on the neutral mean image. I got the following results with larger warp values:

| Warp Amount = 1.5 | Warp Amount = 2 |
| :----: | :----: |
| ![](media/q5_ramya_caricature_warpis1point5.jpg) | ![](media/q5_ramya_caricature_warpis2.jpg) |

Two noticeable differences are that my eyes are larger/further apart, and my chin is smaller.

## Bells and Whistles

### Changing Gender

I chose an image of an average Indian man that I found online to try and change my gender.

| Me | Average Indian Man |
| :----: | :----: |
| <img src="media/q4_ramyacrop_forpts.jpg" width="300"/>  | ![](media/q5data_average_indian_man.png) |

| Change Appearance |  Change Shape | Change Shape + Appearance |
| :----: | :----: | :----: |
| ![](media/bw_gender_appearance.jpg) | ![](media/bw_gender_shape.jpg) | ![](media/bw_gender_shape_appearance.jpg) |