# GIẢI THÍCH CODE TRONG FILE [TUAN_3.ipynb](./TUAN_3.ipynb)

## 1. Phần thực hành trên lớp

### Bài tập 2:

- `fourier_menu()` có nhiệm vụ lấy ảnh đầu vào, ở đây lấy `bird.png` làm ví dụ, sau đó hiển thị menu và cho phép người dùng lựa chọn phép biến đổi. Khi người dùng chọn phép biến đổi, chương trình sẽ gọi đến function tương ứng, cho người dùng nhập vào giá trị biến đổi, nếu bỏ trống thì truyền vào giá trị mặc định và hiển thị ảnh kết quả.

- `show_reusults()` hiển thị ảnh gốc và ảnh đã biến đổi sử dụng `matplotlib`

  1. Fast Fourier Transform (`fast_fourier_transform(img_array)`)
  2. Lowpass Filter (`butterworth_lowpass_filter(img_array, d0=30, n=1)`)
  3. Highpass Filter (`butterworth_highpass_filter(img_array, d0=30, n=1)`)

  - Q. Thoát

### Bài tập 3:

- tạo một list `transformations = []` chứa các giá trị biến đổi
- fucntion `random_transformation()` hoạt động như sau:
  - Load ảnh: Mở file "bird.png", chuyển sang grayscale
  - chọn ngẫu nhiên trong list `transformations = []` dùng `random.choice()`
  - Áp dụng biến đổi: lặp qua các giá trị trong list để thực hiện transformation được chọn
  - gọi lại function `show_results()` từ bài tập trước để hiển thị kết quả
- function `random_transformation_menu()` hoạt động như sau:
  - hiển thị menu cho phép người dùng lựa chọn:
    1. Apply Random Transformation
    - Q. Quit
  - nếu người dùng chọn 1, thì gọi function `random_transformation()`
  - nếu người dùng chọn Q, thì thoát khỏi menu

### Bài tập 4:

- tạo một list `channel_orders = []` chứa các thứ tự kênh màu RGB có thể có (RGB, RBG, GRB, GBR, BRG, BGR)
- tạo một list `filter_combinations = []` chứa 2 tổ hợp phép lọc: 'Butterworth Lowpass + Min Filter' và 'Butterworth Highpass + Max Filter'

- function `change_rgb_order(img_array)` hoạt động như sau:

  - Chọn ngẫu nhiên một thứ tự kênh màu từ list `channel_orders` dùng `random.choice()`
  - Áp dụng thứ tự kênh màu được chọn: `reordered_img = img_array[:, :, selected_order]`
  - Trả về ảnh đã thay đổi thứ tự kênh màu và kênh màu đã áp dụng

- function `apply_random_filter_combination(img_array)` hoạt động như sau:

  - Chọn ngẫu nhiên một tổ hợp phép lọc từ list `filter_combinations` dùng `random.choice()`
  - Chuyển ảnh sang grayscale để thực hiện lọc
  - Áp dụng tổ hợp phép lọc được chọn:
    - Nếu là 'Butterworth Lowpass + Min Filter': thực hiện Butterworth Lowpass trước, sau đó áp dụng Min Filter lên kết quả
    - Nếu là 'Butterworth Highpass + Max Filter': thực hiện Butterworth Highpass trước, sau đó áp dụng Max Filter lên kết quả
  - Tạo tham số ngẫu nhiên cho Butterworth (d0, n) và kernel size cho Min/Max Filter
  - Trả về ảnh đã lọc và tên tổ hợp phép lọc

- function `min_filter(img_array, kernel_size)` và `max_filter(img_array, kernel_size)`:

  - Chuyển ảnh sang grayscale
  - Áp dụng padding cho ảnh
  - Duyệt qua từng pixel, lấy giá trị min/max trong kernel window

- function `rgb_random_processing()` hoạt động như sau:

  - Load ảnh RGB: Mở file "bird.png", giữ nguyên định dạng RGB
  - Bước 1: Gọi `change_rgb_order()` để thay đổi ngẫu nhiên thứ tự kênh màu
  - Bước 2: Gọi `apply_random_filter_combination()` để áp dụng ngẫu nhiên một tổ hợp phép lọc
  - Hiển thị kết quả: gọi `show_images()` để hiển thị ảnh gốc, ảnh đã thay đổi kênh màu, và ảnh đã áp dụng tổ hợp phép lọc

- function `show_images()` hiển thị 3 ảnh cạnh nhau sử dụng `matplotlib`

- function `exercise4_menu()` hoạt động như sau:
  - hiển thị menu cho phép người dùng lựa chọn:
    1. Apply Random RGB Channel Order + Random Filter Combination
    - Q. Quit
  - nếu người dùng chọn 1, thì gọi function `rgb_random_processing()`
  - nếu người dùng chọn Q, thì thoát khỏi menu
