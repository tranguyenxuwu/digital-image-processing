# LAB 06 - Xác định đối tượng trong ảnh

## Phần bài tập trên lớp

1.  **Gắn nhãn ảnh**

    - Sử dụng phương pháp ngưỡng `Otsu` để tự động phân tách đối tượng khỏi nền.
    - Gắn nhãn cho các vùng ảnh liên thông đã được phân tách.
    - Sử dụng `regionprops` để đo lường các thuộc tính (như diện tích, tâm, hộp giới hạn) và vẽ hộp chữ nhật bao quanh từng đối tượng.

2.  **Dò tìm cạnh theo chiều dọc**

    - **Dịch chuyển ảnh:** Phát hiện biên cạnh theo chiều dọc bằng cách trừ ảnh gốc cho phiên bản dịch chuyển của nó.

3. **Dò tìm cạnh theo chiều dọc, sử dụng Sobel Filter**

    - Áp dụng toán tử Sobel để tính toán gradient theo hướng x và y, từ đó làm nổi bật các biên cạnh trong ảnh. 

4.  **Dò tìm Góc (Corner Detection)**

    - **Thuật toán Harris:** Triển khai và áp dụng thuật toán Harris để xác định các điểm góc trong ảnh, là những vùng có sự thay đổi lớn về cường độ điểm ảnh theo mọi hướng.

5.  **Hough Transform**

    - Triển khai biến đổi `Hough` để phát hiện các đường thẳng trong một ảnh

6.  **Trích xuất Đặc trưng với `skimage`**
    - Sử dụng hàm `corner_harris` từ thư viện `scikit-image` để thực hiện tìm góc một cách hiệu quả trên ảnh.
