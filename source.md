# Xây dựng hệ thống truy vấn ảnh dựa trên đặc trưng cục bộ sử dụng SIFT
<br>
<br>
Nhóm sinh viên:

Đinh Quý Phiên - 20152821<br>
Phạm Minh Tú - 20154207<br>
Nguyễn Trọng Tuyền - 20154183
<br>
<br>
## Mục lục

*   [Giới thiệu SIFT](#gioi-thieu-sift)
*   [Phương pháp xác định đặc trưng cục bộ](#phuong-phap-xac-dinh-dac-trung-cuc-bo)
    *   [1. Phát hiện các điểm cực trị cục bộ trên không gian tỉ lệ (scale-space) ](#1-phat-hien-cac-diem-cuc-tri-cuc-bo-tren-khong-gian-ti-le-scale-space-)
    *   [2. Định vị chính xác các điểm đặc trưng](#2-dinh-vi-chinh-xac-cac-diem-dac-trung)
    *   [3. Loại bỏ các đặc trưng trên cạnh](#3-loai-bo-cac-dac-trung-tren-canh)
    *   [4. Xác định hướng cục bộ của điểm đặc trưng](#4-xac-dinh-huong-cuc-bo-cua-diem-dac-trung)
    *   [5. Mô tả đặc trưng](#5-mo-ta-dac-trung)
*   [Xây dựng hệ thống truy vấn ảnh dựa trên đặc trưng cục bộ](#xay-dung-he-thong-truy-van-anh-dua-tren-dac-trung-cuc-bo)
    *   [1. Truy vấn dựa trên so khớp đặc trưng](#1-truy-van-dua-tren-so-khop-dac-trung)
    *   [2. Truy vấn dựa trên đối sánh vector trọng số](#2-truy-van-dua-tren-doi-sanh-vector-trong-so)
    *   [3. Kết quả và đánh giá](#3-ket-qua-va-danh-gia)
        *   [3.1. Đánh giá phương pháp truy vấn dựa trên so khớp đặc trưng](#31-danh-gia-phuong-phap-truy-van-dua-tren-so-khop-dac-trung)
        *   [3.2. Đánh giá phương pháp truy vấn dựa trên đối sánh vector trọng số](#32-danh-gia-phuong-phap-truy-van-dua-tren-doi-sanh-vector-trong-so)
    *   [4. Nhận xét](#4-nhan-xet)
*   [Tài liệu tham khảo](#tai-lieu-tham-khao)

## Giới thiệu SIFT

SIFT (Scale-Invariant Feature Transform) là một phương pháp hiệu quả giúp tìm ra những điểm đặc trưng cục bộ với những tính chất sau:
-   Tính bất biến với các phép biến đổi hình học như phép thu phóng hay phép xoay
-   Tính bất biến bộ phận đối với những thay đổi về độ sáng và góc nhìn
-   Mức độ định vị và tính đặc trưng cao

Các bước trích rút đặc trưng cục bộ sử dụng phương pháp SIFT như sau:
1.  *Scale-space extrema detection*: Tìm kiếm ra các điểm bất biến với phép thu phóng và phép xoay bằng cách tiếp cận difference-of-Gaussian
1.  *Keypoint localization*: Định vị vị trí và scale của các điểm đặc trưng dựa trên tính ổn định của chúng trên một mô hình đánh giá
1.  *Orientation assignment*: Mỗi keypoint được gán với một hoặc nhiều hướng dựa vào hướng của gradient
1.  *Keypoint descriptor*: Tính gradient cục bộ của ảnh xung quanh điểm đặc trưng tại scale đã chọn. Thông số này giúp biểu diễn điểm đặc trưng một     cách ổn định trong điều kiện bị tác động bởi sự thay đổi hình dạng và màu sắc cục bộ

Thuật toán cho ra một số lượng lớn các điểm đặc trưng phủ kín bề mặt ảnh tại những vị trí khác nhau trên những scale khác nhau. Thông thường, một bức ảnh kích thước 500x500 pixels có thể cho ra khoảng 2000 điểm đặc trưng ổn định.

Trong bài toán so khớp ảnh, hình ảnh truy vấn được so khớp bằng cách so sánh lần lượt với từng ảnh có sẵn trong cơ sở dữ liệu thông qua khoảng cách   Euclide giữa hai vector điểm đặc trưng. Bài báo cáo cũng sẽ đề cập đến phương pháp từ điển hình ảnh giúp cải thiện tốc độ tính toán và độ chính xác của mô hình.

## Phương pháp xác định đặc trưng cục bộ

### 1. Phát hiện các điểm cực trị cục bộ trên không gian tỉ lệ (scale-space) 

**Cơ sở lý thuyết**

Phép scale ảnh được thực hiện bằng cách thực hiện phép tích chập giữa một bộ lọc Gaussian $$G(x, y, \sigma)$$ và ảnh đầu vào $$I(x, y)$$ với tham số  scale $$\sigma$$:

$$L ( x , y , \sigma ) = G ( x , y , \sigma ) * I ( x , y )$$

Sau đó, độ lệch của Gaussian giữa scale $$\sigma$$ và một scale lân cận $$k\sigma$$ được tính toán theo công thức:

$$\begin{aligned} D ( x , y , \sigma ) & = ( G ( x , y , k \sigma ) - G ( x , y , \sigma ) ) * I ( x , y ) \\ & = L ( x , y , k \sigma ) - L ( x ,    y , \sigma ) \end{aligned}$$

Giá trị $$D(x, y, \sigma)$$ được gọi là different-of-Gaussian (hay $$DoG$$) trên scale $$\sigma$$. Giá trị này được sử dụng để tính xấp xỉ giá trị hàm Laplacian of Gaussian:

$$LoG = \sigma ^ { 2 } \nabla ^ { 2 } G$$

Các cực trị của $$LoG$$ được chứng minh là các điểm đặc trưng ổn định nhất trên ảnh (*Mikolajczyk 2002*). Mối quan hệ xấp xỉ giữa $$DoG$$ và $$LoG$$ được chứng minh như sau:

Ta có: 

$$\frac { \partial G } { \partial \sigma } = \sigma \nabla ^ { 2 } G$$

Mặt khác, một cách xấp xỉ:

$$\frac { \partial G } { \partial \sigma } \approx \frac { G ( x , y , k \sigma ) - G ( x , y , \sigma ) } { k \sigma - \sigma }$$

Suy ra:

$$\begin{aligned}\sigma \nabla ^ { 2 } G &\approx \frac { G ( x , y , k \sigma ) - G ( x , y , \sigma ) } { k \sigma - \sigma } \\
\Leftrightarrow ( k - 1 ) \sigma ^ {2} \nabla ^ { 2 } G &\approx { G ( x , y , k \sigma ) - G ( x , y , \sigma ) } \\
\Leftrightarrow (k-1)LoG&\approx DoG\end{aligned}$$

**Triển khai cài đặt**

Thao tác tính toán $$DoG$$ được thực hiện lặp lại theo các *octave*. Trong đó, đầu vào của octave đầu tiên là ảnh gốc, đầu vào của các octave sau là đầu vào của octave trước được giảm các kích thước đi một nửa.

<div style="text-align:center"><img src ="https://docs.opencv.org/3.4/sift_dog.jpg"/></div>

Tại mỗi octave, ảnh đầu vào được nhân chập với mặt nạ Gaussian với scale tăng dần theo cấp số nhân công bội k, sinh ra tập ảnh:

$$\{L_1(x, y, \sigma), L_2(x, y, k\sigma), L_3(x, y, k^2\sigma), ...\}$$

Trong đó, số lượng ảnh được tạo ra là $$s+3$$ với $$s=[log_k2]$$.

Tiếp theo, ta tính $$DoG$$ giữa hai ảnh liên tiếp:

$$D_i = L_{i+1} - L{i}$$

Với mỗi điểm thuộc $$D_i$$, ta so sánh nó với 8 điểm trong ô 3x3 xung quanh, 9 điểm thuộc ô 3x3 ở vị trí tương ứng trong $$D_{i+1}$$ và 9 điểm thuộc ô 3x3 ở vị trí tương ứng trong $$D_{i-1}$$. Nếu điểm này là cực trị địa phương trong tổng số 27 điểm đang xét, ta lưu lại giá trị ***tọa độ*** và ***scale*** tương ứng của nó.

<div style="text-align:center"><img src ="https://docs.opencv.org/3.4/sift_local_extrema.jpg"/></div>

Sau khi tìm hết tất cả các cực trị địa phương của các ảnh $$D_i$$ tại octave thứ $$n$$, ta giảm kích thước mỗi chiều của ảnh đầu vào tại octave này đi một nửa và thực hiện lại các thao tác trên trong octave $$n+1$$.

Kết thúc pha đầu tiên,ta thu được tập các điểm cực trị địa phương của $$DoG$$ trên các scale khác nhau, với các kích thước khác nhau.

### 2. Định vị chính xác các điểm đặc trưng

Với mỗi điểm cực trị $$\mathbf{x_0}(x_0, y_0, \sigma)$$ thu được từ bước 1, ta tiến hành tìm điểm cực trị nội suy của hàm $$DoG$$ trong lân cận của nó. Thao tác này được thực hiện bằng cách sử dụng thi triển Taylor bậc 2 của $$DoG$$ xung quanh lân cận $$\mathbf{x_0}$$:

$$D ( \mathbf { x } ) \approx D + \frac { \partial D ^ { T } } { \partial \mathbf { x_0 } } \mathbf { (x - x_0) } + \frac { 1 } { 2 } \mathbf { (x - x_0) } ^ { \mathbf { T } } \frac { \partial ^ { 2 } D } { \partial \mathbf { x_0 } ^ { 2 } } \mathbf { (x - x_0) }$$

Cực trị $$\mathbf{\hat{x}}$$ của hàm số trên được tính bằng cách giải nghiệm phương trình đạo hàm bằng 0. Giá trị này được sử dụng để đánh giá độ ổn định của điểm đặc trưng như sau:
- Nếu $$max(|\mathbf{\hat{x} - x_0}}|) > 0.5$$, cực trị địa phương thực tế nằm gần với một điểm đặc trưng khác hơn là điểm hiện tại, ta loại bỏ điểm hiện tại khỏi danh sách các điểm đặc trưng
- Nếu $$|D(\mathbf{\hat{x} - x_0}})| < 0.03$$, cực trị tìm được là không ổn định do có độ tương phản thấp, ta cũng loại bỏ nó ra khỏi danh sách các điểm đặc trưng
- Ngược lại, ta lưu lại điểm đặc trưng mới $$\mathbf{\hat{x}}$$ thay cho $$\mathbf{x_0}$$

### 3. Loại bỏ các đặc trưng trên cạnh

Nhược điểm của hàm $$D$$ là nó phản ứng mạnh hơn đối với các đặc trưng trên cạnh, khiến cho nhiều điểm đặc trưng nằm trên cạnh không thực sự tốt và cần phải loại bỏ.

**Cơ sở lý thuyết**

Trên cạnh, độ cong chính theo chiều tiếp tuyến tại một điểm lớn hơn rất nhiều so với độ cong chính theo chiều pháp tuyến tại điểm đó. Cặp giá trị này chính là hai trị riêng $$\alpha$$ và $$\beta$$ ($$\alpha > \beta$$) của ma trận sau:

$$\mathbf { H } = \left[ \begin{array} { l l } { D _ { x x } } & { D _ { x y } } \\ { D _ { x y } } & { D _ { y y } } \end{array} \right]$$

Trường hợp $$\beta < 0$$ không cần xét đến vì khi đó điểm được chọn không phải là cực trị địa phương của $$DoG$$.

Xét trường hợp $$\alpha > \beta > 0$$, tỉ số:

$$r = \frac \alpha \beta$$

sẽ nhận giá trị rất lớn nếu điểm đặc trưng đang xét nằm trên cạnh. Do vậy, ta sẽ loại bỏ các điểm đặc trưng có $$r$$ quá lớn.

Tuy nhiên, thao tác tính trực tiếp trị riêng  $$\alpha, \beta$$ là tương đối tốn thời gian. Thay vào đó, ta có thể đánh giá giá trị của r một cách đơn giản hơn như sau:

Tính:

$$\begin{array} { c } { \operatorname { Tr } ( \mathbf { H } ) = D _ { x x } + D _ { y y } = \alpha + \beta } \\ { \operatorname { Det } ( \mathbf { H } ) = D _ { x x } D _ { y y } - \left( D _ { x y } \right) ^ { 2 } = \alpha \beta } \end{array}$$

Suy ra:

$$\frac { \operatorname { Tr } ( \mathbf { H } ) ^ { 2 } } { \operatorname { Det } ( \mathbf { H } ) } = \frac { ( \alpha + \beta ) ^ { 2 } } { \alpha \beta } = \frac { ( r \beta + \beta ) ^ { 2 } } { r \beta ^ { 2 } } = \frac { ( r + 1 ) ^ { 2 } } { r }$$

Do hàm số $$f(r) = \frac {(r+1)^2} r$$ là đồng biến theo $$r$$ với $$r>1$$ nên:

$$r \geq r_{thres} \Leftrightarrow \frac { \operatorname { Tr } ( \mathbf { H } ) ^ { 2 } } { \operatorname { Det } ( \mathbf { H } ) } \geq f_{thres}$$

Việc loại bỏ các điểm đặc trưng có giá trị $$r$$ lớn hơn hoặc bằng ngưỡng $$r_{thres}$$ tương đương với việc loại bỏ các điểm đặc trưng có $$\frac { \operatorname { Tr } ( \mathbf { H } ) ^ { 2 } } { \operatorname { Det } ( \mathbf { H } ) }$$ vượt quá $$f_{thres}$$ tương ứng.
 
**Triển khai**

Giá trị thực nghiệm $$r_{thres} = 10$$, tương đương với $$f_{thres} = 1.21$$

### 4. Xác định hướng cục bộ của điểm đặc trưng

Xét điểm bất kỳ $$\mathbf{x}(x, y, \sigma)$$ nằm trên ảnh Gaussian $$L$$, ta tính được độ lớn $$m$$ và hướng $$\theta$$ của vector gradient tại điểm đó như sau:

$$m = \sqrt{L^{\prime 2}_x + L^{\prime 2}_y}$$

$$\theta = \arctan(\frac {L^{\prime}_y} {L^{\prime}_x})$$

Trong đó:

$$\begin{aligned}L ^ { \prime }_x &= \lim _ { \Delta x \rightarrow 0 } \frac { L ( x + \Delta x, y ) - L ( x - \Delta x, y ) } { 2\Delta x } \\
&\approx \frac 1 2(L ( x + 1, y ) - L ( x - 1, y ))\end{aligned}$$

$$\begin{aligned}L ^ { \prime }_y &= \lim _ { \Delta y \rightarrow 0 } \frac { L ( x, y + \Delta y ) - L ( x, y - \Delta y ) } { 2\Delta y } \\
&\approx \frac 1 2(L ( x , y + 1 ) - L ( x , y - 1 ))\end{aligned}$$

Tạo ra một orientation histogram từ hướng của tất cả các điểm lân cận của điểm đặc trưng trong bán kính $$\sigma$$ gấp $$1.5$$ lần độ lớn của scale của điểm đặc trưng. Histogram này chứa 36 kênh, mỗi kênh tương ứng với 10 độ, bao phủ một góc 360 độ xung quanh điểm đặc trưng. Mỗi điểm được thêm vào histogram có trọng số là độ lớn vector gradient của nó.

Tất cả các điểm "peak" có độ cao lớn hơn 80% độ cao lớn nhất của histogram đều được chọn làm hướng cho điểm đặc trưng. Theo thống kê, chỉ có khoảng 15% số điểm đặc trưng được gán nhiều hướng, tuy nhiên chúng đều rất ổn định và đóng góp đáng kể trong việc thể hiện đặc trưng cục bộ của ảnh.

### 5. Mô tả đặc trưng

Qua các bước ở trên, các tham số tọa độ, scale và hướng được gán cho các điểm đặc trưng, giúp chúng có tính bất biến đối với những tham số này. Bước tiếp theo là tạo ra các tham số mô tả trong không gian cục bộ (local descriptor) xung quanh điểm đặc trưng giúp chúng trở nên bất biến với những biến đổi khác như thay đổi về độ sáng hay góc nhìn. Thao tác này được thực hiện như sau:
- Tính toán độ lớn và hướng của gradient trong ô 16x16 pixels xung quanh điểm đặc trưng
- Chia vùng 16x16 thành 16 phân vùng 4x4, trong mỗi phân vùng 4x4 tạo ra một orientation histogram chứa 8 kênh, mỗi kênh tương ứng với một góc 45 độ.
- Tất cả 16 histogram này được lưu lại làm descriptor cho điểm đặc trưng (16 x 8 = 128 tham số)

## Xây dựng hệ thống truy vấn ảnh dựa trên đặc trưng cục bộ

#### 1. Truy vấn dựa trên so khớp đặc trưng

Phép so khớp hai điểm đặc trưng được thực hiện bằng cách so sánh độ khác biệt giữa hai vector mô tả (descriptor) của chúng. Mỗi vector này có 128 chiều và được so sánh với nhau thông qua khoảng cách Euclide.

Ta kết luận hai điểm đặc trưng là khớp nhau nếu khoảng cách giữa chúng là nhỏ nhất, đồng thời khoảng cách này không vượt quá 0.8 lần khoảng cách nhỏ thứ hai. Ràng buộc sau có tác dụng tăng độ tin cậy cho kết luận so khớp, tránh việc nhiễu xảy ra làm cho một điểm đặc trưng có thể khớp với nhiều đặc trưng khác.

Mô hình hệ thống truy vấn dựa trên so khớp đặc trưng được xây dựng như sau:

<div style="text-align:center;"><img style="width: 75% "src ="https://github.com/tupm20154207/mmdb/blob/master/sokhopdactrung.png?raw=true"/></div>

- Xây dựng cơ sở dữ liệu đặc trưng: Lưu trữ đặc trưng cục bộ cho tất cả các ảnh trong cơ sở dữ liệu
- Truy vấn dựa trên so khớp: So khớp đặc trưng của ảnh với tất cả các ảnh trong cơ sở dữ liệu, xếp hạng kết quả theo trung bình khoảng cách giữa các điểm so khớp

#### 2. Truy vấn dựa trên đối sánh vector trọng số

Đối với cơ sở dữ liệu ảnh lớn, việc so khớp đặc trưng của ảnh truy vấn với lần lượt từng ảnh trong cơ sở dữ liệu là rất tốn thời gian. Phương pháp từ điển hình ảnh (Visual Bag of Words - VBOW) giúp giải quyết vấn đề này và cải thiện tốc độ tính toán một cách đáng kể. Các bước tổ chức xây dựng từ điển như sau:

1. Tính và tổng hợp tất cả các vector trọng số của tất cả các ảnh trong cơ sở dữ liệu
2. Sử dụng phương pháp phân cụm KMeans để gom tập dữ liệu đặc trưng thành 1000 nhóm. Coi mỗi nhóm đặc trưng là một "visual word"
3. Xây dựng visual word histogram cho từng ảnh, kết quả mỗi ảnh cho ra một vector 1000 chiều, mỗi chiều là số đặc trưng của ảnh thuộc cụm tương ứng, thể hiện số lần "visual word" tương ứng xuất hiện trong ảnh. 
4. Sử dụng phương pháp xây dựng trọng số tf-idf để tạo ra vector trọng số cho từng ảnh trong bộ dữ liệu và lưu trữ lại trong cơ sở dữ liệu trọng số

Sau khi đã xây dựng cơ sở dữ liệu trọng số, hệ thống sẽ thực hiện truy vấn cho ảnh như sau:

1. Tính vector trọng số của ảnh đầu vào theo phương pháp tf-idf
1. Tính khoảng cách giữa vector trọng số của ảnh đầu vào với lần lượt tất cả các vector trọng số trong cơ sở dữ liệu
1. Sắp xếp kết quả theo thứ tự khoảng cách tăng dần và hiển thị

Mô hình hệ thống tổng quát như sau:

<div style="text-align:center"><img style="width: 85%"src ="https://github.com/tupm20154207/mmdb/blob/master/tudienhinhanh.png?raw=true"/></div>

### 3. Kết quả và đánh giá

#### 3.1. Đánh giá phương pháp truy vấn dựa trên so khớp đặc trưng

Đối với phương pháp này, thời gian xử lý cho mỗi câu truy vấn dao động trong khoảng từ 20 giây đến 2 phút. Do hạn chế về năng lực tính toán, nhóm chưa có số liệu thống kê hoàn thiện để đánh giá độ chính xác của hệ thống một cách tốt nhất. Dưới đây là một vài tham số đã được các thành viên trong nhóm lựa chọn và cho là phù hợp với hệ thống truy vấn hiện tại.

- Phương pháp so khớp: So khớp vét cạn (`Brute-Force Matching`) hoặc các biến thể
- Hàm khoảng cách: Khoảng cách `Euclide`
- Độ đo đánh giá: Trung bình cộng khoảng cách của `10` matches tốt nhất

**Một vài kết quả**:

<div style="text-align:center"><img style="width: 85%"src ="https://github.com/tupm20154207/mmdb/blob/master/des1.jpg?raw=true"/></div>
<br>
<div style="text-align:center"><img style="width: 85%"src ="https://github.com/tupm20154207/mmdb/blob/master/des2.jpg?raw=true"/></div>

#### 3.2. Đánh giá phương pháp truy vấn dựa trên đối sánh vector trọng số

Đối với phương pháp này, thời gian xử lý trung bình cho mỗi câu truy vấn dao động trong khoảng 0.1 đến 0.2 giây. Tốc độ tính toán và độ chính xác cũng được cải thiện một cách đáng kể so với phương án thứ nhất. Các tham số chính được lựa chọn trong phương pháp này là:

- Số visual word: `1000`
- Phương pháp tính tf.idf theo hệ thống SMART: `ntn-ntn` (tính idf cho cả xâu truy vấn và văn bản, không chuẩn hóa kết quả)
- Hàm khoảng cách: Khoảng cách `L2 (Euclide)`, `L1` hoặc `Cosine`. Kết quả khảo sát mức độ hiệu quả của từng phương pháp tính khoảng cách được ghi lại bên dưới.

**Đánh giá mô hình**

Tập dữ liệu mẫu bao gồm 1000 bức ảnh, mỗi bức ảnh thuộc về 1 chủ đề cụ thể.  Người dùng mong đợi rằng khi tìm kiếm một bức ảnh, kết quả trả về sẽ bao gồm các bức ảnh tương tự thuộc cùng chủ đề. Do vậy, độ chính xác của mỗi phép truy vấn được đo bằng số lượng kết quả thuộc cùng chủ đề với hình ảnh truy vấn trong 10 kết quả trả về tốt nhất. Giá trị này hoàn toàn là đánh giá chủ quan của người quan sát, có thể không phù hợp với tất cả các mô hình trong thực tế.

*Độ chính xác của mô hình phụ thuộc vào hàm khoảng cách*

<div style="text-align:center"><img src ="https://github.com/tupm20154207/mmdb/blob/master/test_distance.png?raw=true"/></div>

**Một vài kết quả**

<div style="text-align:center"><img style="width: 85%"src ="https://github.com/tupm20154207/mmdb/blob/master/vbow1.jpg?raw=true"/></div>
<br>
<div style="text-align:center"><img style="width: 85%"src ="https://github.com/tupm20154207/mmdb/blob/master/vbow4.png?raw=true"/></div>

### 4. Nhận xét

SIFT là một phương pháp trích xuất đặc trưng cục bộ có độ tin cậy cao. Tuy nhiên, trong bài toán truy vấn ảnh, kích thước không gian descriptors là hạn chế so với kích thước của không gian ảnh. Điều này dẫn đến xuất hiện nhiễu lớn trong trường hợp ảnh chứa nhiều điểm đặc trưng, làm giảm độ chính xác của kết quả truy vấn.

Nhiều phương pháp đã được đề xuất để giải quyết vấn đề so khớp đặc trưng đối với cơ sở dữ liệu lớn. Tuy nhiên, chúng đều không thực sự hiệu quả trong bài toán hiện tại. Tốc độ tính toán chậm là một vấn đề lớn cần giải quyết đối với hệ thống truy vấn ảnh dựa trên so khớp đặc trưng.

Phương pháp từ điển hình ảnh cải thiện cả tốc độ lẫn độ chính xác của kết quả một cách đáng kể.

## Tài liệu tham khảo

[1] https://docs.opencv.org/3.4/da/df5/tutorial_py_sift_intro.html <br>
[2] https://people.eecs.berkeley.edu/~malik/cs294/lowe-ijcv04.pdf <br>
[3] https://docs.opencv.org/3.0-beta/doc/py_tutorials/py_feature2d/py_matcher/py_matcher.html