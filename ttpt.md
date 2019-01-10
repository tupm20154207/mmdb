# Ôn tập tinh toán phân tán

## Leader Election

### 1. Định nghĩa

-   Là quá trình hệ thống phân tán thống nhất chọn ra một tiến trình duy nhất thực hiện một công việc nào đó

### 2. Tinh chất

-   Các tiến trình chỉ nhận một trong hai trạng thai: được bầu và không được bầu
-   Kết quả bầu cử là không thể thay đổi
-   Cuối cùng, luôn có duy nhất một tiến trình được bầu ra làm leader
-   Các giả định:
  -   Mạng có topo vòng
  -   Các id (nếu có) là các giá trị nguyên dương duy nhất

### 3. Các giải thuật leader election

-   Ring with IDs
    -   Asynchronous Algorithms
      -   O(n<sup>2</sup>) Algorithm
      -   O(nlog(n)) Algorithm
    -   Synchronous Algorithms
      -   Non-uniform Algorithm
      -   Uniform Algorithm
-   Anonynous Ring
    -   Impossibility
    -   Randomized Algorithm

#### 3.1. Async - O(n<sup>2</sup>) Algorithm

*>>> Phương pháp:* Trên vòng có IDs, các tiến trình:

1.  Gửi ID của mình trong một thông điệp tới nút bên trái
2.  Hủy gói tin có ID nhỏ hơn ID của mình
3.  Chuyển tiếp gói tin có ID lớn hơn ID của mình

*>>> Đánh giá:*

1.  Tính đúng đắn:
    -   Tiến trình có ID nhỏ nhất luôn trở thành leader vì gói tin của nó luôn được chuyển tiếp trên vòng
    -   Các gói tin của các tiến trình còn lại luôn không vượt qua được tiến trình có ID nhỏ nhất -> Không có 2 leaders
2.  Time-complexity: **O(n)** 
3.  Message-complexity: **O(n<sup>2</sup>)** - Không tối ưu

#### 3.2. Async - O(nlog(n)) Algorithm

*>>> Phương pháp:*

Với mỗi tiến trình p, tại bước thứ k:
1.  p gửi thông điệp tranh cử tới 2<sup>k</sup> nút kế trước và 2<sup>k</sup> nút kế sau nó trên vòng
1.  Nếu trong số các tiến trình nhận được thông điệp của p tại bước thứ k, có một tiến trình có id lớn hơn id của p, thủ tục kết thúc
1.  Ngược lại, chúng gửi lại thông điệp trả lời tới cho p
1.  Nếu p nhận đủ 2<sup>k</sup> thông điệp trả lời, nó chuyển sang bước k + 1
1.  Nếu p nhận lại thông điệp tranh cử mà nó gửi -> tự bầu làm leader

*>>> Đánh giá:*

1.  Tính đúng đắn: Tương tự phương pháp O(n<sup>2</sup>)
2.  Time-complexity: **O(n)**
3.  Message-complexity:
    -   Số thông điệp mà mỗi tiến trình gửi đi/nhận về ở bước k: 4.2<sup>k</sup>
    -   Số tiến trình gửi thông điệp tối đa ở mức k: <img align="center" src="https://tex.s2cms.ru/svg/%5Cfrac%20%7Bn%7D%20%7B2%5E%7Bk-1%7D%20%2B%201%7D"/>
