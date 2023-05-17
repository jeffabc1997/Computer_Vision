# CV_hw1 
Gaussian Blur, Sobel Operator, Harris Corner Detection, SIFT
# Part 1
## HW1-1.py
- A. Corner Detection:
    - a. Gaussian Smooth: Show the results of Gaussian smoothing for 𝜎=5 and kernel size=5 and 10 respectively.
    - b. Intensity Gradient (Sobel edge detection): Apply the Sobel filters to the blurred images and compute the magnitude (2 images) and direction (2 images) of gradient. (You should eliminate weak gradients by proper threshold.)
    - c. Structure Tensor: Use the Sobel gradient magnitude (with Gaussian kernel size=10) above to compute the structure tensor 𝐻 of each pixel. Show the images of the smaller eigenvalue of 𝐻 with window size 3x3 and 5x5.
    - d. Non-maximal Suppression: Perform non-maximal suppression on the results above along with appropriate thresholding for corner detection.

- B. Experiments (Rotate and Scale): Apply the same corner detection algorithm to the rotated (by 30°) and scaled (to 0.5x) images.
## A.a Gaussian Blur
一開始以為要用彩色的高斯模糊，所以用kernel把RGB三個array都convolve一遍，最後再把三個array stack起來變成彩色的圖片。
雖然kernel size 10不是奇數，但是可以利用 -4.5 ~ 4.5來製造出一個對稱的kernel，亦即kernel column和row是0到9，則中間的(x, y): (4, 4) (4, 5)會擁有一樣的值。

## A.b Intensity Gradient
用兩個function去運算gradient。第一個function `sobel_magnitude_angle(image_read)` 就是把a小題的圖利用sobel operator運算gradient (因為圖像是discrete而非continue，無法直接微分)，並且使用arctan得到angle。
Magnitude的呈現則是把上一個function算出來的gradient直接輸出在image。
接著用 `MagAngle_to_HSV(sobel_Mag_Angle)` 把上一個function的magnitude (gradient)當作pixel的亮度(hsv中的value)，angle當作hue，來呈現。

## A.c Structure Tensor (with Gaussian kernel size=10)

`sobel_derivative_xy(image_read)`這個function會計算出x方向的derivative和y方向的derivative，也就是講義上說的Ix和Iy。
`structure_tensor_make(sobel_gradient)`利用上述的Ix和Iy算出每個pixel的structure tensor。

`SSD_make(structure_tensor, window_size)` 用window掃過每個pixel，把該pixel的structure tensor都加總起來成為一個2x2 Matrix。最後的output size是(image.shape, 2, 2)

`little_eigenvalue(SSD, threshold)` 把上述的每個2x2矩陣都算出它的較小的eigenvalue = det(H) / tr(H)，用threshold篩過之後呈現在畫面上。

## A.d Corner Detection with Non-Maximum Suppression
`harris_response(SSD_structure_tensor, threshold)` 是從SSD_make 的output去運算harris response = = det(H) - k * (tr(H)^2) 然後再通過threshold
再來用`non_max_suppression(image_read, window_size)` 做NMS，window size為3x3或5x5。
最後使用`overlay_img(orignal_img, nms_img, w1, w2)` 把彩色的原圖和NMS所找到的corner去做重疊

## B. Experiments (Rotate and Scale)

基本上就是多做一個 `rotate_img(img, rotate_angle, scale)` 把原圖逆時針旋轉30度且縮小為0.5。剩下的步驟如同A的a至d。

# Part 2
## HW1-2.py
## A. SIFT interest point detection
- a. Apply SIFT interest point detector (functions from OpenCV) to the following two images
- b. Adjust the related thresholds in SIFT detection such that there are around 100 interest points detected in each image .
- c. Plot the detected interest points on the corresponding images.

`SIFT(img, nfeat)` 使用`cv2.SIFT_create(nfeatures = nfeat)` 選出keypoint並且用nfeatures決定keypoint數量。Part A即可利用output出來的keypoint畫出來。

## B. SIFT feature matching
- a. Compare the similarity between all the pairs between the detected interest points from each of the two images based on a suitable distance function between two SIFT feature vectors.
- b. Implement a function that finds a list of interest point correspondences based on nearest-neighbor matching principle.
- c. Plot the point correspondences (from the previous step) overlaid on the pair of original images.

PartB用 `SIFT(img, nfeat)`中的`detectAndCompute()` 計算出keypoint的descriptor。
計算原理：對圖像做高斯模糊、對圖片降維，透過與下一個維度的圖片相減，再尋找上下相鄰維度的26個點中的local maxima...(略)，然後利用magnitude與orientation建立histogram...(略)，最後建立出一個128維的descriptor。
接著用`matcher(kp1, descriptor1, kp2, descriptor2)`將左圖的descriptor當作一個點，去向右圖的descriptor計算Euclidean Distance，我們先計算出兩個最短和次短的descriptor distance。
然後用 `ratio_test(matches, threshold, kp1, kp2)` 比對最短和次短的distance的差距大不大，如果差距很大，代表這個最短的distance是顯著的，也就是我們要的真正的對應點。
最後再用 `plot_matches(matches, total_img)`把兩張原圖和有matching的灰階圖疊出來。
