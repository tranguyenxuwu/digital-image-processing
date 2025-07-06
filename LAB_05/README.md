# LAB 05 - Image Segmentation

## Phần bài tập trên lớp

bài tập này có sử dụng thêm thư viện `opencv-python` so với những bài trước

## Các phương pháp đã áp dụng:

1. **Otsu Thresholding** - Phân ngưỡng tự động

   - Tự động tìm ngưỡng tối ưu để phân tách foreground/background
   - Phù hợp với ảnh có histogram 2 đỉnh rõ ràng
   - **Functions**: `threshold_otsu()` từ scikit-image

2. **Adaptive Thresholding** - Phân ngưỡng thích ứng (commented)

   - Ngưỡng thay đổi theo từng vùng địa phương của ảnh
   - Hiệu quả với ảnh có độ sáng không đồng đều
   - **Functions**: `threshold_local()` từ scikit-image

3. **Region Segmentation** - Phân vùng theo region với watershed

   - Sử dụng thuật toán watershed để phân tách các đối tượng
   - Kết hợp erosion và distance transform để tối ưu kết quả
   - **Functions**: `cv2.threshold()`, `cv2.erode()`, `cv2.distanceTransform()`, `cv2.watershed()`, `label()` từ scipy

4. **Morphological Operations** - Các phép biến đổi hình thái:
   - **Binary Dilation**: Mở rộng vùng foreground, lấp đầy lỗ nhỏ (`nd.binary_dilation()`)
   - **Binary Opening**: Erosion + Dilation, loại bỏ nhiễu nhỏ (`nd.binary_opening()`)
   - **Binary Erosion**: Thu hẹp vùng foreground, tách đối tượng dính (`nd.binary_erosion()`)
   - **Binary Closing**: Dilation + Erosion, lấp đầy khe hở trong đối tượng (`nd.binary_closing()`)
