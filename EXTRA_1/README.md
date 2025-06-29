# XỬ LÝ ẢNH SỐ BUỔI THỰC HÀNH 4

#### GIẢI THÍCH CODE [THỰC HÀNH](./BT_TC.ipynb) PHẦN NÀY CHO BÀI TẬP BỔ SUNG

## 1. Hiệu ứng Sóng (Wave Effect)

### Mục đích:

- Biến dạng hình học của ảnh để tạo ra hiệu ứng gợn sóng uốn lượn.

### Nguyên lý hoạt động:

- Thuật toán này không thay đổi giá trị màu của pixel, mà thay đổi vị trí của chúng.

1. Tạo lưới tọa độ: Đầu tiên, tạo ra hai ma trận row_coords và col_coords. Mỗi ma trận có cùng kích thước với ảnh gốc. Tại vị trí (i, j):

- Ma trận `row_coords` chứa giá trị `i`.
- Ma trận `col_coords` chứa giá trị `j`.
- Đây chính là bản đồ tọa độ gốc của ảnh.

2. Tính toán tọa độ mới: Vị trí của mỗi pixel sẽ được dịch chuyển theo một hàm hình sin.

- Sóng ngang (thay đổi cột): Vị trí cột mới được tính bằng cách lấy vị trí cột cũ cộng với một lượng dịch chuyển phụ thuộc vào vị trí hàng.
- `new*col` = `old_col + A * sin(F \_ old_row)`
- Sóng dọc (thay đổi hàng): Tương tự, vị trí hàng mới cũng được tính.
- `new*row` = `old_row + A' * sin(F' \_ old_col)`

3. Ánh xạ điểm ảnh (Mapping): Dùng hàm `map_coordinates` để tạo ảnh mới. Với mỗi pixel ở tọa độ (r, c) trong ảnh kết quả, hàm sẽ:

- Nhìn vào bản đồ tọa độ mới `(wave_rows, wave_cols)` tại vị trí `(r, c)`.
- Lấy ra cặp tọa độ đã biến đổi, ví dụ `(new_r, new_c)`.
- Lấy giá trị màu từ ảnh gốc tại vị trí `(new_r, new_c)` và đặt nó vào ảnh kết quả tại `(r, c)`.
- Vì `(new_r, new_c)` thường là số thực, hàm sẽ dùng nội suy `(order=1)` để tính toán màu sắc một cách mượt mà từ các pixel nguyên lân cận.

### Lưu ý:

- `amplitude`: Biên độ sóng. Giá trị càng lớn, độ uốn lượn của ảnh càng mạnh.
- `frequency`: Tần số sóng. Giá trị càng lớn, các gợn sóng càng dày và gần nhau hơn.
- `map_coordinates` là một hàm rất mạnh của scipy.ndimage, chuyên dùng cho các phép biến đổi hình học phức tạp.
- `mode='reflect'`: Khi tọa độ mới tính ra nằm ngoài rìa ảnh, nó sẽ lấy pixel bằng cách "phản chiếu" lại từ mép, giúp viền ảnh không bị đen hoặc kéo dãn kỳ lạ.

### Ví dụ:

Giả sử ta xét pixel tại vị trí gốc `(row, col) = (10, 100)`. Với `amplitude=30, frequency=0.03`:

- Tính vị trí cột mới:
  `wave*cols = 100 + 30 * sin(0.03 _ 10)`
  `wave_cols = 100 + 30 _ sin(0.3) ≈ 100 + 30 _ 0.2955 ≈ 108.87`
- Tính vị trí hàng mới:
  `wave_rows = 10 + (30 _ 0.5) _ sin(0.03 _ 100)`
  `wave*rows = 10 + 15 * sin(3) ≈ 10 + 15 \* 0.1411 ≈ 12.12`
- Kết quả: Pixel trong ảnh mới tại vị trí (10, 100) sẽ lấy giá trị màu từ ảnh gốc tại tọa độ xấp xỉ (12.12, 108.87).

### Code chính:

```python
# Tạo lưới tọa độ gốc
row_coords, col_coords = np.meshgrid(np.arange(rows), np.arange(cols), indexing='ij')
# Tính toán bản đồ tọa độ mới theo công thức sóng sin
wave_cols = col_coords + amplitude _ np.sin(frequency _ row_coords)
wave_rows = row_coords + amplitude _ 0.5 _ np.sin(frequency \* col_coords)
# Áp dụng biến đổi: tạo ảnh mới bằng cách lấy pixel từ ảnh cũ theo bản đồ tọa độ mới
wave_image[:,:,i] = nd.map_coordinates(image[:,:,i], [wave_rows, wave_cols], order=1, mode='reflect')
```

## 2. Tô màu Gradient và ghép ảnh vào vị trí

**Mục đích:**

- Thay thế màu sắc gốc của một đối tượng trong ảnh bằng một dải màu gradient

**Nguyên lý hoạt động:**

Thuật toán này tính toán màu mới cho mỗi pixel dựa trên một "tỷ lệ" (`ratio`) được pha trộn từ hai yếu tố: vị trí dọc và độ sáng gốc của pixel đó.

1.  **Tạo "Mặt nạ" Grayscale:**

    - Ảnh gốc được chuyển thành grayscale (grayscale).
    - Mục đích: Lấy giá trị độ sáng (từ 0 đến 255) tại mỗi pixel. Giá trị này sẽ đóng vai trò là một "trọng số" để điều khiển sự chuyển màu. Một vùng sáng trên ảnh gốc sẽ có xu hướng nghiêng về màu kết thúc nhiều hơn.

2.  **Chuẩn hóa (Normalization):**

    - Giá trị độ sáng của grayscale được đưa về khoảng `[0, 1]`. Điều này cần thiết để có thể sử dụng nó trong các phép tính tỷ lệ phần trăm.

3.  **Tính toán Tỷ lệ chuyển màu (Ratio Calculation):**

    - Đây là trái tim của thuật toán. Với mỗi pixel, một giá trị `ratio` từ 0 đến 1 được tính toán.
    - **Công thức:**
      > `ratio = (i / height) * 0.7 + gray_norm[i, j] * 0.3`
    - **Phân tích công thức:**
      - `(i / height)`: Tỷ lệ vị trí theo chiều dọc. Bằng 0 ở hàng trên cùng và tiến dần tới 1 ở hàng dưới cùng. Thành phần này tạo ra một dải màu chuyển cơ bản từ trên xuống dưới.
      - `gray_norm[i, j]`: Tỷ lệ độ sáng gốc của pixel (đã chuẩn hóa). Thành phần này làm cho những vùng sáng hơn trên vật thể gốc sẽ "bắt" màu kết thúc nhanh hơn.
      - `0.7` và `0.3`: Đây là các trọng số. Trong code này, hiệu ứng chuyển màu sẽ phụ thuộc **70% vào vị trí dọc** và **30% vào độ sáng gốc**.

4.  **Nội suy màu (Color Interpolation):**
    - Dựa trên `ratio` đã tính, màu cuối cùng được nội suy tuyến tính giữa màu bắt đầu (`start_color`) và màu kết thúc (`end_color`).
    - **Công thức:**
      > `Màu_mới = Màu_bắt_đầu * (1 - ratio) + Màu_kết_thúc * ratio`
    - Nếu `ratio` gần 0, màu mới sẽ gần với `Màu_bắt_đầu`.
    - Nếu `ratio` gần 1, màu mới sẽ gần với `Màu_kết_thúc`.

**Lưu ý:**

- Trọng số `0.7` và `0.3` có thể thay đổi. Nếu đặt là `1.0` và `0.0`, hiệu ứng sẽ trở thành một gradient thuần túy từ trên xuống dưới, không phụ thuộc vào nội dung ảnh.
- Dòng `if gray[i, j] > 0:` là một cách đơn giản để bỏ qua các pixel nền (giả định nền có màu đen hoàn toàn `(0,0,0)`).
- Ảnh kết quả được tạo ở định dạng `RGBA` (4 kênh), với kênh Alpha được đặt là 255 (hoàn toàn không trong suốt) cho các pixel của đối tượng. Điều này rất quan trọng cho việc ghép ảnh trên nền trong suốt sau này.

**Ví dụ:**

- Xét một pixel ở giữa ảnh (`i/height ≈ 0.5`) và tại một vùng rất sáng (`gray_norm ≈ 0.9`).
- Màu bắt đầu là Đỏ `[255, 0, 0]`, màu kết thúc là Xanh lá `[0, 255, 0]`.
- Tính `ratio`: `ratio = (0.5 * 0.7) + (0.9 * 0.3) = 0.35 + 0.27 = 0.62`.
- Nội suy màu:
  - Kênh Red: `255 * (1 - 0.62) + 0 * 0.62 ≈ 97`
  - Kênh Green: `0 * (1 - 0.62) + 255 * 0.62 ≈ 158`
  - Kênh Blue: `0`
- Kết quả: Pixel này sẽ có màu xanh ô-liu ngả vàng `[97, 158, 0]`, thể hiện sự pha trộn nghiêng về màu Xanh lá.

**Code chính:**

```python
# Tính tỷ lệ chuyển màu, là sự kết hợp giữa vị trí dọc và độ sáng gốc
ratio = (i / height) * 0.7 + gray_norm[i, j] * 0.3

# Nội suy tuyến tính để tìm màu mới dựa trên tỷ lệ
# Kênh R
gradient_img[i, j, 0] = start_color[0] * (1 - ratio) + end_color[0] * ratio
# Kênh G
gradient_img[i, j, 1] = start_color[1] * (1 - ratio) + end_color[1] * ratio
# Kênh B
gradient_img[i, j, 2] = start_color[2] * (1 - ratio) + end_color[2] * ratio
# Đặt kênh Alpha là không trong suốt
gradient_img[i, j, 3] = 255
```

## 3. Xoay và Phản chiếu ảnh

**Mục đích:**

- Thực hiện các phép biến đổi hình học cơ bản: xoay ảnh theo một góc nhất định và tạo hiệu ứng phản chiếu đối xứng.

**Nguyên lý hoạt động:**

Thuật toán được chia thành ba bước chính: Xoay, Phản chiếu, và Ghép ảnh.
Trước khi xử lý, ảnh được resize về 1920x1080 để đồng bộ hóa hơn khi xử lý và ghép ảnh

1.  **Phép xoay ảnh `nd.rotate`:**

    - Hàm này xoay mảng NumPy (tức là ảnh) quanh tâm của nó một góc cho trước.
    - **`reshape=False`:** Đây là tham số quan trọng. Nó đảm bảo ảnh đầu ra có cùng kích thước (chiều cao, chiều rộng) với ảnh gốc. Phần ảnh bị xoay ra ngoài khung sẽ bị cắt bỏ. Ngược lại, nếu `reshape=True`, hàm sẽ tạo ra một ảnh mới lớn hơn để chứa toàn bộ phần ảnh đã xoay.
    - **`mode='constant', cval=255`:** Khi xoay, các vùng trống sẽ xuất hiện ở các góc của ảnh. Tham số này chỉ định cách lấp đầy các vùng trống đó: `'constant'` nghĩa là lấp bằng một giá trị không đổi, và `cval=255` chỉ định giá trị đó là 255 (màu trắng).

2.  **Tạo phản chiếu dùng `np.flipud` và `np.vstack`:**

    - Hàm này tạo ra hiệu ứng gương dọc.
    - **`np.flipud(image)`:** `flipud` là viết tắt của "Flip Up-Down". Hàm này lật ngược ảnh theo chiều dọc bằng cách đảo ngược thứ tự các hàng của mảng. Hàng đầu tiên trở thành hàng cuối cùng, hàng thứ hai trở thành hàng áp chót, và cứ thế.
    - **`np.vstack((image, flipped))`:** `vstack` là viết tắt của "Vertical Stack". Nó xếp chồng hai (hoặc nhiều) mảng lên nhau theo chiều dọc. Bằng cách xếp chồng ảnh gốc lên trên ảnh đã lật, chúng ta tạo ra một ảnh mới có chiều cao gấp đôi, với nửa dưới là hình ảnh phản chiếu hoàn hảo của nửa trên.

3.  **Ghép ảnh :**
    - **Tạo Canvas:** Một mảng NumPy lớn chứa toàn màu trắng (`np.ones(...) * 255`) được tạo ra. Kích thước của nó được tính toán để đủ chứa cả hai ảnh đã qua xử lý và một khoảng trống ở giữa.
    - **Dán ảnh:** Sử dụng kỹ thuật "slicing" của NumPy để chọn một vùng hình chữ nhật trên canvas và gán trực tiếp dữ liệu của ảnh cần dán vào vùng đó. Đây là một cách cực kỳ hiệu quả để đặt một ảnh vào một vị trí cụ thể trên một ảnh khác.

**Lưu ý:**

- Để tạo hiệu ứng phản chiếu ngang (trái-phải), có thể dùng `np.fliplr` (Flip Left-Right) và `np.hstack` (Horizontal Stack).
- Việc chọn `cval=255` khi xoay là rất hợp lý vì canvas nền cũng có màu trắng, giúp các góc xoay hòa lẫn vào nền một cách tự nhiên.
- Kỹ thuật "slicing" `canvas[y1:y2, x1:x2] = image` là phương pháp nền tảng và rất mạnh mẽ để xử lý và ghép các mảng trong NumPy.

**Ví dụ (cho Phép phản chiếu):**

Giả sử ảnh gốc là một mảng 2x2:
`image = [[1, 2], [3, 4]]`

1.  **`flipud(image)`** sẽ cho kết quả: `flipped = [[3, 4], [1, 2]]`
2.  **`vstack((image, flipped))`** sẽ xếp chồng chúng lại:
    ```
    [[1, 2],  # Từ ảnh gốc
     [3, 4],  # Từ ảnh gốc
     [3, 4],  # Từ ảnh đã lật
     [1, 2]]  # Từ ảnh đã lật
    ```
    Kết quả là một ảnh phản chiếu dọc hoàn hảo.

**Code chính:**

```python
# Xoay ảnh, giữ nguyên kích thước, lấp đầy góc trống bằng màu trắng (255)
nui_rotated = nd.rotate(data_nui, 45, reshape=False, mode='constant', cval=255)

# Lật mảng theo trục dọc (hàng đầu thành hàng cuối và ngược lại)
flipped = np.flipud(image)

# Ghép mảng gốc và mảng đã lật theo chiều dọc để tạo hiệu ứng gương
mirrored = np.vstack((image, flipped))

# Dán ảnh vào vùng chỉ định trên canvas bằng phương pháp slicing
canvas[y_offset_nui:y_offset_nui + nui_mirror.shape[0], :nui_mirror.shape[1]] = nui_mirror
```

## 4. Biến dạng Thấu kính và Uốn cong ảnh

**Mục đích:**

- Mô phỏng các hiệu ứng quang học hoặc nghệ thuật phức tạp bằng cách thay đổi vị trí của từng pixel.
- Cụ thể là tạo hiệu ứng "phồng" ảnh (barrel distortion) và hiệu ứng "gợn sóng" (wave warp).

**Nguyên lý hoạt động:**

Khác với các phép xoay hay lật ảnh, các phép biến đổi này phức tạp hơn vì mỗi pixel được di chuyển đến một vị trí mới không đồng nhất. Cốt lõi của thuật toán là **tái ánh xạ pixel (pixel remapping)**.

**1. Tạo "Bản đồ" tọa độ (`x_map`, `y_map`):**

Đây là bước quan trọng nhất, nơi logic của từng hiệu ứng được định nghĩa. Thay vì di chuyển pixel, chúng ta tạo ra hai "bản đồ" (hai mảng 2D có cùng kích thước với ảnh gốc) để chỉ định nguồn gốc của từng pixel trong ảnh mới.

- **`x_map`**: Mảng này chứa tọa độ **X** của ảnh gốc mà mỗi pixel của ảnh mới sẽ lấy màu từ đó.
- **`y_map`**: Tương tự, mảng này chứa tọa độ **Y** của ảnh gốc.

Cách các bản đồ này được tạo ra sẽ quyết định hình dạng của hiệu ứng:

- **Với hiệu ứng Thấu kính (`barrel_distortion`):**

  - **Mục tiêu:** Làm cho ảnh bị phồng ra ở trung tâm, giống như nhìn qua một ống kính.
  - **Logic tính toán:**
    1.  Tìm tọa độ tâm của ảnh (`cx`, `cy`).
    2.  Với mỗi pixel, tính khoảng cách từ nó đến tâm (`r`).
    3.  Tạo ra một "hệ số phồng" (`factor`) tỷ lệ thuận với khoảng cách đến tâm. Pixel càng xa tâm, hệ số này càng lớn.
    4.  Tọa độ mới (`x_new`, `y_new`) được tính bằng cách "đẩy" tọa độ cũ ra xa tâm một khoảng dựa trên `factor`. Tham số `strength` sẽ điều khiển độ "phồng" của ảnh.

- **Với hiệu ứng (`wave_warp`):**
  - **Mục tiêu:** Tạo ra các gợn sóng uốn lượn trên toàn bộ bức ảnh, giống như hình ảnh phản chiếu trên mặt nước.
  - **Logic tính toán:**
    1.  Lấy lưới tọa độ gốc (`x`, `y`).
    2.  Thay đổi tọa độ của mỗi pixel bằng cách cộng thêm một giá trị từ các hàm lượng giác `sin` và `cos`.
    3.  Vì giá trị của `sin`/`cos` thay đổi tuần hoàn, nó tạo ra hiệu ứng gợn sóng tự nhiên. Các tham số `ax`, `ay` (biên độ) và `f` (tần số) cho phép điều khiển hình dạng và mật độ của các gợn sóng.

**2. Ánh xạ tọa độ (`nd.map_coordinates`):**

Đây là hàm của `scipy.ndimage` thực hiện việc tái ánh xạ.

- Nó nhận đầu vào là ảnh gốc và hai bản đồ tọa độ `[y_map, x_map]`.
- Với mỗi vị trí trên ảnh đầu ra, nó sẽ nhìn vào `x_map` và `y_map` để biết cần phải "hút" pixel từ tọa độ nào của ảnh gốc.
- **`order=1`**: Nếu tọa độ cần lấy là một số không nguyên (ví dụ 100.5, 200.2), hàm sẽ nội suy (trung bình có trọng số) từ các pixel lân cận để tạo ra kết quả mượt mà, tránh bị răng cưa.
- **`mode='reflect'`**: Khi tọa độ tính toán nằm ngoài rìa ảnh gốc, `'reflect'` sẽ lấy các pixel đối xứng qua cạnh ảnh, tạo ra một sự chuyển tiếp liền mạch.

**3. Xử lý ảnh màu (`apply_distortion`):**

- `nd.map_coordinates` chỉ hoạt động trên mảng 2D (grayscale), trong khi ảnh màu của chúng ta là mảng 3D (chiều cao, chiều rộng, 3 kênh màu R-G-B).
- Hàm `apply_distortion` được tạo ra để giải quyết vấn đề này. Nó lặp qua từng kênh màu (R, G, B), áp dụng `map_coordinates` trên từng kênh, sau đó dùng `np.stack` để ghép 3 kênh kết quả lại thành một ảnh màu hoàn chỉnh.

**Lưu ý:**

- **`np.clip(...)`**: Sau khi tính toán, một số tọa độ mới có thể nằm ngoài kích thước thực của ảnh. `np.clip` dùng để "cắt" các giá trị này, đảm bảo chúng luôn nằm trong phạm vi hợp lệ.
- **`nd.zoom`**: Code đã phóng to ảnh lên 5 lần trước khi biến dạng. Điều này cung cấp nhiều pixel hơn cho các phép biến đổi, giúp kết quả cuối cùng trông mượt mà và chi tiết hơn.
- **Chuỗi hiệu ứng**: Code đã áp dụng hiệu ứng uốn cong (`wave_warp`) lên kết quả của hiệu ứng thấu kính (`img_barrel`). Đây là một ví dụ về việc kết hợp nhiều bộ lọc để tạo ra một kết quả độc đáo.

**Code chính:**

```python
# Cốt lõi của việc tái ánh xạ trên một kênh màu
# tại các tọa độ được chỉ định trong `y_map` và `x_map`."
nd.map_coordinates(image_channel, [y_map, x_map], order=1, mode='reflect')

# Tính toán "bản đồ" tọa độ mới cho hiệu ứng phồng
# Đẩy các pixel ra xa tâm
r_norm = r / np.sqrt(cx**2 + cy**2)
factor = 1 + strength * r_norm**2
x_new = x_c * factor + cx

# Tính toán "bản đồ" tọa độ mới cho hiệu ứng sóng
# Cộng thêm các hàm sin/cos để tạo độ uốn lượn
x_new = x + ax * np.sin(f * y) + ...
y_new = y + ay * np.sin(f * x) + ...

# Áp dụng hiệu ứng theo chuỗi
img_barrel = barrel_distortion(img_zoom, strength=0.2)
img_warped = wave_warp(img_barrel, ax=30, ay=20, f=0.008)
```

Tất nhiên rồi! Đây là phần giải thích cho chương trình menu xử lý ảnh, được viết theo phong cách của.

---

## 5. Menu Tương tác Chọn Cách Xử lý Ảnh

**Mục đích:**

- Xây dựng một chương trình CLI có menu tương tác, cho phép người dùng chọn và áp dụng các hiệu ứng xử lý ảnh khác nhau lên một ảnh gốc.

**Nguyên lý hoạt động:**

Chương trình được xây dựng dựa trên hai thành phần chính: một **luồng tương tác** điều khiển menu và một **bộ các hàm xử lý ảnh** độc lập.

### 1. Cấu trúc chương trình và Luồng tương tác

Đây là "bộ não" của chương trình, điều khiển việc hiển thị menu và gọi các hàm xử lý tương ứng.

- **Vòng lặp vô tận (`while True`):**

  - Toàn bộ menu được đặt trong một vòng lặp `while True`. Điều này đảm bảo rằng sau khi thực hiện xong một chức năng, menu sẽ tự động hiển thị lại, cho phép người dùng tiếp tục thực hiện các thao tác khác mà không cần chạy lại chương trình.
  - Vòng lặp chỉ kết thúc khi người dùng chọn '6' (Thoát), câu lệnh `break` sẽ được thực thi.

- **Nhận Lựa chọn (`input()`):**

  - Hàm `input()` được sử dụng để hiển thị các lựa chọn cho người dùng và chờ họ nhập một giá trị.
  - Các tham số cho mỗi hiệu ứng (ví dụ: góc xoay, hệ số phóng) cũng được thu thập từ người dùng bằng chính hàm này.

- **logic (`if/elif/else`):**
  - Dựa trên lựa chọn của người dùng (`choice`), chương trình sẽ thực thi một khối lệnh tương ứng.
  - Mỗi khối lệnh sẽ:
    1.  Hỏi người dùng các tham số cần thiết cho phép biến đổi đó.
    2.  Gọi hàm xử lý ảnh tương ứng với các tham số vừa nhận.
    3.  Hiển thị ảnh kết quả bằng hàm `show_image()`.

### 2. Chi tiết các hàm xử lý ảnh

Mỗi hàm đóng gói một thuật toán xử lý ảnh cụ thể, nhận đầu vào là ảnh và các tham số, trả về ảnh đã qua xử lý.

- **Tịnh tiến (`translate_image()`):**

  - Sử dụng hàm `nd.shift` để dịch chuyển toàn bộ các pixel của ảnh theo một vector `(dx, dy)`.
  - `shift=(dy, dx, 0)`: Tham số này là một tuple. Lưu ý thứ tự `(dy, dx)` vì mảng NumPy xử lý theo thứ tự (hàng, cột). Số `0` cuối cùng có nghĩa là không dịch chuyển theo trục màu sắc.
  - `mode='constant', cval=255`: Vùng trống được tạo ra sau khi dịch chuyển sẽ được lấp đầy bằng một giá trị không đổi là 255 (màu trắng).

- **Xoay ảnh (`rotate_image()`):**

  - Dùng hàm `nd.rotate` đã quen thuộc.
  - Cho phép người dùng tùy chọn `reshape` (thay đổi kích thước khung để chứa toàn bộ ảnh xoay) hay không, tạo ra sự linh hoạt trong xử lý.

- **Phóng to (`zoom_image()`):**

  - **Cách tiếp cận khác biệt:** Hàm này không dùng `nd.zoom` để nội suy pixel. Thay vào đó, nó thực hiện "phóng to kỹ thuật số" (digital zoom) bằng cách **cắt ra một vùng ảnh nhỏ hơn từ chính giữa ảnh gốc**.
  - **Logic:**
    1.  Tính toán kích thước của vùng ảnh sẽ được cắt (`new_h`, `new_w`) dựa trên `factor` (ví dụ: `factor=2` sẽ tạo ra một vùng cắt có kích thước bằng một nửa ảnh gốc).
    2.  Xác định tọa độ góc trên bên trái (`start_y`, `start_x`) để đảm bảo vùng cắt nằm ngay chính giữa.
    3.  Sử dụng kỹ thuật slicing của NumPy (`img[start_y:end_y, start_x:end_x]`) để trích xuất vùng ảnh này. Kết quả là một ảnh có kích thước nhỏ hơn nhưng tạo cảm giác "phóng to" vào trung tâm.

- **Làm mờ Gaussian (`blur_image()`):**

  - Sử dụng bộ lọc `nd.gaussian_filter`. Bộ lọc này làm mượt ảnh bằng cách lấy trung bình có trọng số của các pixel lân cận, với trọng số tuân theo phân phối Gaussian.
  - `sigma`: Tham số này điều khiển "độ mạnh" của hiệu ứng mờ. `sigma` càng lớn, bán kính ảnh hưởng của bộ lọc càng rộng và ảnh càng mờ.
  - **Xử lý ảnh màu:** Vì `gaussian_filter` chỉ làm việc trên mảng 2D, code phải áp dụng bộ lọc này lần lượt trên từng kênh màu (R, G, B) rồi dùng `np.stack` để ghép chúng lại.

- **Biến đổi Sóng (`wave_transform()`):**
  - Hoạt động dựa trên nguyên lý **tái ánh xạ pixel** với `nd.map_coordinates`, tương tự như bài 4.
  - Tọa độ mới của mỗi pixel được tính bằng cách cộng tọa độ cũ với một giá trị từ hàm `sin`/`cos`, tạo ra hiệu ứng uốn lượn.
  - `amplitude` (biên độ) và `frequency` (tần số) là các tham số do người dùng nhập để điều khiển "độ mạnh" và "độ dày" của các gợn sóng.

**Lưu ý:**

- **Cách `zoom_image` hoạt động:** Điều quan trọng cần nhớ là hàm `zoom_image` này thực hiện zoom-in bằng cách cắt ảnh. Nó không thể thực hiện zoom-out (thu nhỏ) và khác hoàn toàn với hàm `nd.zoom` (sử dụng nội suy để thay đổi kích thước).
- **Xử lý ảnh màu:** Một kỹ thuật chung được sử dụng trong `blur_image` và `wave_transform` là tách ảnh màu thành các kênh 2D, xử lý riêng lẻ rồi ghép lại. Đây là một phương pháp phổ biến khi làm việc với các thư viện xử lý ảnh.
- **Tính tương tác:** Sức mạnh của chương trình này nằm ở menu tương tác, giúp người dùng dễ dàng thử nghiệm và thấy ngay kết quả của các thuật toán với những tham số khác nhau.
