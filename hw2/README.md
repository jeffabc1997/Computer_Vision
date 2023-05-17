# CV_hw2
Computer Vision course in NTHU
Epipolar Geometry, Eight-point algorithm, Normalized eight-point algorithm, Fundamental matrix, Rank-2 Constraint, 
Homography, Image Rectification
# Part 1
## HW2-1.py
## Fundamental Matrix Estimation from Point Correspondences: 
In this problem, you will implement both the linear least-squares version of the eight-point algorithm and its normalized version to estimate the fundamental matrices. You will implement the methods and complete the following:
- (a) Implement the linear least-squares eight-point algorithm and report the returned fundamental matrix.
Remember to enforce the rank-two constraint for the fundamental matrix via singular value
decomposition. Briefly describe your implementation in your written report.
- (b) Implement the normalized eight-point algorithm and report the returned fundamental matrix. Remember to enforce the rank-two constraint for the fundamental matrix via singular value
decomposition. Briefly describe your implementation in your written report.
- (c) Plot the epipolar lines for the given point correspondences determined by the fundamental matrices computed from (a) and (b). Determine the accuracy of the fundamental matrices by computing the
average distance between the feature points and their corresponding epipolar lines.

## a. Implement the linear least-squares eight-point algorithm and report the returned fundamental matrix with rank-2 constraint
- Step 1: Compute Fundamental Matrix by SVD
設每一 correspondence point p 與 p’為(u,v,1), (u’,v’,1)，A 的每一列為 p 和 p’的 Kronecker product (uu’, uv’, u,..., u’, v’, 1)，Dimension 為 1x9，把每個 product 疊起來變成 n x 9 的矩陣。
SVD得UD(V^T)，取f為V的最後一列，也就是對應到最小的singular value。將f再SVD一次取UDV^T，把D的最小singular value設為0，再將UD'V^T乘起來、reshape即為Fundamental Matrix。最後再把fundamental matrix的F33(或者是F22)設為1。
- Step 2: Find epipolar line
line equation l: = ax + by + c = 0。利用Fp'(或F^Tp) 可求出左圖(或右圖)的epipolar line的係數(a, b, c)。公式推導: p^TFp' = 0，已知F和p'，所以只要滿足此式的p即為所求之線。
- Step 3: Plot epipolar line
設定好image size(512, 512)，因此我們只要在這個範圍內畫線即可，因此宣告x_line array，元素都是[0, 512]，拿去和各個line equation去計算符合的y，最後再標記出助教給的46個interest points。
- Step 4: Compute average distance between feature points and epipolar lines 
Point-to-Line Distance 公式: ||ax + by + c|\| / ( a<sup>2</sup> + b<sup>2</sup> )<sup>1/2</sup> 

## b. The normalized eight-point algorithm with rank-2 contraint fundamental matrix

1. 取得點 p 的平均值 m1，其中 m1 = (mean_x, mean_y, 1)。m2 為點 p’的平均值，算法相 同。
2. 將所有的點 p 拿去和 m1 相減算出 distance 的平方。
3. 將所有的點 p’拿去和 m2 相減算出 distance 的平方。
4. s1 = (第二點的總和 / 點 p 的個數)開根號，再乘上根號二分之一 。
5. s2 = (第二點 / 點 p’的個數)開根號，再乘上根號二分之一。
6. 見右圖(T’想成 T2)，然後得 new_p = T1p，new_p’ = T2p’。
7. 用新的座標重複 1(a)的步驟獲得 fundamental matrix F’。
8. 得還原 Fundamental Matrix F = (T1^T)F’T2。
9. F = F / F[2,2] 也就是把 F 的第三列第三欄的值設成 1。

# Part 2
## HW2-2.py or HW2-2_new.py
## Homography transform: 
You need to determine a homograpgy transformation for plan-to-plane transformation. The homography transformation is determined by a set of point correspondences between the source image and the target image.
- (a) Implement a function that estimates the homography matrix H that maps a set of interest points to a
new set of interest points. Describe your implementation.
- (b) Specify a set of point correspondences for the source image of the Delta building and the target one.
Compute the 3X3 homography matrix to rectify the front building of the Delta building image. The rectification is to make the new image plane parallel to the front building as best as possible. Please select four corresponding straight lines to compute the homograph matrix. Describe your implementation and show the selected correspondence line pairs, the homography matrix, and the rectified image. (Please use backward warping and bilinear interpolation)

## a. Estimate the homography matrix H that maps a set of interest points to a new set of interest points.
每一對 correspondence point 滿足 x’ = Hx，經過一些轉換後得到式子 Ah = 0，A 為 2n x 9 的矩陣，h 為 1 x 9 的 vector。為了求 least square solution，minimize ||Ah - 0|\|，我 們知道要做 SVD 分解，這和 part 1.a 的步驟二一樣，h 即為 UD(V^T)中 V^T 的最後一列，h 對應的是singular value 0。最後再把 vector h reshape 成 3 x 3 的 homography matrix。
## b. Compute the homography matrix to rectify the front building of the image, making the new image plane parallel to the front building as best as possible.
 (Please use backward warping and bilinear interpolation)
 1. 要找 new image plane 的四個點，也就是原圖台達館的正面，從原圖可以找到正面的左上、右上、左下、右下四個點 (下圖左)。首先利用 matplotlib 秀出圖片的座標，可以快速找到右圖 4 個點的位置。(要注意 matplotlib 的 x, y 和 array 的 row, column 的對應方式，不然成像會旋轉錯誤)
2. 因為是 backward warping，所以 part 2.(a)中所述的算式 x’ = Hx 中的 x’是原圖我挑選的 4 個座標，x 是 new image plane 的四個角落 (左上、右上、左下、右下)。接著用 part 2.(a)的 方式算出 homography matrix H，利用 H 把 new image plane 上面的每個 x 對到原圖 x’上 面。
3. 由於 x’必定是落在原圖的某四個點當中，例如:設 x’ = (u’, v’, 1)，那個 x’必定落在(i, j), (i+1, j), (i+1, j+1), (i, j+1)四個點中，利用 bilinear interpolation 的方式我們算出 x’是上述四點的 RGB 的某種混合，再將這個 RGB 組合呈現在 x 的位置(也就是 new image plane 上面)。

