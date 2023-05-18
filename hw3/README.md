Image Alignment, RANSAC, SIFT, Image Segmentation, K-means, K-means++, Computer Vision

# Part 1. Image Alignment with RANSAC
## HW3-1.py
- A. (25%) Use OpenCV function or publicly available program(such as the one from http://www.vlfeat.org/) for SIFT interest point detection to extract the SIFT feature points from the above 4 images. Establish point correspondences between the SIFT feature points detected from the single-book images and the cluttered-book image by using the distance between the SIFT feature vectors as the matching score. Show your best point correspondence results with different distance thresholds (the number of correspondences should be larger than 500).
- B. (25%)Apply the RANSAC program in problem1 to find the best matching homography transformation between each single-book image and the input image by using the SIFT features computed from problem 1. Show the detection result by showing the deviation vectors between the transformed feature points and the corresponding feature points on the input image. You may need to try different parameter setting in RANSAC to find the best matching result. Compare the parameter settings in SIFT feature and RANSAC and discuss the result.

## a. 
`sift_img_concatenate` 利用作業2的SIFT來計算兩張圖的interest point，把兩張的圖片的大小(y軸)對齊後用`plot_matches`畫出對應的點和線。

## b.
`ransac`步驟
1. 把(a)的matches用`random_point`隨機取4個出來做homography matrix
2. `get_error` 用該matrix把所有的左圖matches算到右圖的點，並和原本的右圖的matches算距離差。return 每個點的error和右圖新算的點。
3. 對每個點的error經過threshold篩選後，剩下的為inlier，記錄下最好的matrix、inlier數目和新算的右圖inlier。
4. 重複1~3，return matches' inliers、能留下最多inlier的matrix和計算出的右圖inliers
最後用`plot_deviation_vector`畫出來

# Part 2. Image segmentation:
## HW3-2.py
- A. (10 pts) Apply K-means on the image (RGB color space) and try it with three different K values (your K should be > 3) and show the results (3 images). You should use 50 random initial guesses to select the best result based on the objective function for each K. Please discuss the difference between the results for different K’s.
- B. (10 pts) Implement K-means++ to have better initial guess (3 images). Please discuss the difference between (A) and (B).

## a.

`k_means`
1. 用`np.random.seed(int(time.time()))`製造隨機，然後隨機選出k個cluster的centroids
2. assign這些centroid的rgb和所屬的cluster
3. 將每個點去對這些centroid計算距離，最小的距離者，就把該點assign到該cluster
4. 計算每個cluster的mean，並將該cluster的centroid設成mean值
5. 把新的clusters' centroids和舊的相減，重複1~4，如果沒變則代表到了local minima
6. 把image的每個點assign成該cluster的顏色

## b.
`k_means_pp_center`
1. 隨機選一個centroid
2. 將所有data與現存的所有centroid算距離，用`random.choices()`裡面的weight去做權重選取下一個centroid
3. 選到直到k個centroid為止
return `k_means_Plus`要用的centers

## c.
`uniform_mean_shift_3d`
1. 只讀image一半的data point以減少運算時間
2. 將所有data point去和cluster centroid的rgb相減後square再sum，得到該centroid和所有data point的距離
3. 利用threshold得到認定為比較近的點後取mean，將該centroid設為mean值後，再做步驟2，直到centroid不會再變動為止
4. output_datavector則是記錄要畫在圖檔上的總data point，兩個相鄰點會是一樣的色彩(因為步驟1)

## d.
`uniform_mean_shift_5d`和(c)差別在於把長、寬、rgb都設成0~1之間，在步驟2的時候才不會有太大的差異
