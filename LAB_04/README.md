# XỬ LÝ ẢNH SỐ BUỔI THỰC HÀNH 4

#### GIẢI THÍCH CODE [THỰC HÀNH](./TUAN_04.ipynb) TRÊN LỚP

### 1. Chọn Đối tượng bằng cách Cắt ảnh (Slicing)

**Mục đích:**

- Cắt một vùng chữ nhật từ ảnh gốc. Đây là thao tác cơ bản để tập trung vào một đối tượng cụ thể.

**Nguyên lý hoạt động:**

Hoạt động dựa trên kỹ thuật **slicing (cắt lát)** của NumPy. Khi ảnh được nạp, nó trở thành một lưới pixel (mảng NumPy), cho phép chúng ta chọn một vùng chữ nhật bằng cách chỉ định phạm vi hàng và cột.

Cú pháp `data[800:1200, 570:980]` có nghĩa là:

- **`800:1200`**: Lấy các hàng từ 800 đến 1199 (chiều dọc).
- **`570:980`**: Lấy các cột từ 570 đến 979 (chiều ngang).

Phần kênh màu (chiều thứ ba) được lấy toàn bộ một cách tự động. Kết quả là một ảnh mới chỉ chứa các pixel nằm trong vùng chữ nhật đã định.

**Lưu ý:**

- Hệ tọa độ trong NumPy có gốc (0,0) ở góc **trên-bên trái**, với trục Y hướng xuống.
- Kỹ thuật slicing là phương pháp nền tảng và rất hiệu quả để xử lý mảng trong NumPy.

**Code chính:**

```python
# Đọc ảnh gốc vào mảng NumPy
data = iio.imread('fruit.jpg')

# Dùng cú pháp slicing để cắt một vùng ảnh:
# array[hàng_bắt_đầu:hàng_kết_thúc, cột_bắt_đầu:cột_kết_thúc]
bmg = data[800:1200, 570:980]

# Hiển thị ảnh đã cắt
plt.imshow(bmg)
plt.show()
```

### 1.2. Tịnh tiến Ảnh (Shift)

**Mục đích:**

- Dịch chuyển toàn bộ ảnh sang một vị trí mới theo một hướng và khoảng cách nhất định.

**Nguyên lý hoạt động:**

Hoạt động dựa trên hàm `nd.shift` của thư viện SciPy, có chức năng dịch chuyển các phần tử của một mảng.

- Hàm này nhận đầu vào là ảnh và một tuple chỉ định độ dịch chuyển.
- Tuple `(100, 25)` có nghĩa là:
  - **`100`**: Dịch chuyển 100 pixel theo trục thứ nhất (trục Y, hàng), tức là **dịch xuống dưới**.
  - **`25`**: Dịch chuyển 25 pixel theo trục thứ hai (trục X, cột), tức là **dịch sang phải**.
- Vùng trống được tạo ra do sự dịch chuyển sẽ được lấp đầy bằng một giá trị không đổi, mặc định là 0 (màu đen).

**Lưu ý:**

- **`mode="F"`**: Tham số này khi đọc ảnh sẽ chuyển ảnh sang định dạng ảnh xám (grayscale) với kiểu dữ liệu là số thực (float). Vì vậy, mảng `data` chỉ có 2 chiều (cao, rộng) và tuple dịch chuyển cũng chỉ cần 2 giá trị `(dy, dx)`.
- **Thứ tự tọa độ:** Luôn nhớ rằng trong SciPy/NumPy, thứ tự dịch chuyển là `(shift_dọc, shift_ngang)`.

**Code chính:**

```python
# Đọc ảnh và chuyển sang dạng ảnh xám (grayscale), kiểu float
data = iio.imread("fruit.jpg", mode="F")

# Dịch chuyển ảnh: 100 pixel xuống dưới (trục Y), 25 pixel sang phải (trục X)
# Vùng trống mới sẽ được lấp đầy bằng màu đen (giá trị 0)
bdata = nd.shift(data, (100, 25))

# Hiển thị ảnh đã tịnh tiến
plt.imshow(bdata)
plt.show()
```

### 1.3. Thay đổi Kích thước ảnh (Zoom)

**Mục đích:**

- Phóng to hoặc thu nhỏ kích thước của ảnh. Đây là một thao tác phổ biến để chuẩn hóa kích thước ảnh hoặc để xem chi tiết hơn.

**Nguyên lý hoạt động:**

Hoạt động dựa trên hàm `nd.zoom` của SciPy. Khác với việc chỉ nhân đôi hay loại bỏ pixel, hàm này sử dụng kỹ thuật **nội suy (interpolation)**. Nó sẽ tính toán giá trị màu của các pixel mới dựa trên các pixel gốc lân cận.

Cách điều khiển việc phóng to/thu nhỏ:

1.  **Dùng một số duy nhất:** `nd.zoom(data, 2)`

    - Hệ số phóng `2` sẽ được áp dụng cho **tất cả các trục** của mảng (chiều cao, chiều rộng, và cả kênh màu). Điều này thường không phải là điều chúng ta muốn với ảnh màu vì nó sẽ cố gắng thay đổi số kênh màu.

2.  **Dùng một tuple:** `nd.zoom(data, (2, 2, 1))`
    - Đây là cách chính xác và được khuyên dùng cho ảnh màu.
    - Mỗi giá trị trong tuple tương ứng với hệ số phóng cho một trục: `(hệ_số_cao, hệ_số_rộng, hệ_số_kênh_màu)`.
    - ` (2, 2, 1)` có nghĩa là: Phóng to chiều cao và chiều rộng gấp 2 lần, nhưng **giữ nguyên** (nhân với 1) số kênh màu.

Ví dụ `nd.zoom(data, (0.5, 8.5, 1))` cho thấy khả năng thay đổi kích thước không đồng đều: thu nhỏ chiều cao còn một nửa (`0.5x`), kéo giãn chiều rộng ra 8.5 lần, và giữ nguyên kênh màu.

**Lưu ý:**

- Đối với ảnh màu (mảng 3 chiều), luôn ưu tiên dùng **tuple** để kiểm soát chính xác từng trục và tránh làm thay đổi số kênh màu.
- `nd.zoom` có thể xử lý các hệ số phóng là số thực (ví dụ `0.5`, `8.5`), giúp việc thay đổi kích thước rất linh hoạt.

**Code chính:**

```python
# Đọc ảnh gốc
data = iio.imread('fruit.jpg')

# Phóng to ảnh gấp đôi (cao x2, rộng x2) và giữ nguyên kênh màu (x1)
# Đây là cách làm đúng cho ảnh màu.
data2 = nd.zoom(data, (2, 2, 1))

# Thay đổi kích thước không đồng đều: thu nhỏ chiều cao, kéo giãn chiều rộng
data3 = nd.zoom(data, (0.5, 8.5, 1))

# Hiển thị kết quả
plt.imshow(data3)
plt.show()
```

### 1.4. Xoay ảnh (Rotate)

**Mục đích:**

- Thay đổi hướng của ảnh bằng cách xoay nó một góc nhất định quanh tâm.

**Nguyên lý hoạt động:**

Hoạt động dựa trên hàm `nd.rotate` của SciPy. Điểm mấu chốt của hàm này nằm ở tham số `reshape`, quyết định cách xử lý kích thước của ảnh đầu ra.

1.  **`reshape=True` (Mặc định):**

    - Khi không chỉ định, `reshape` mặc định là `True`.
    - Hàm sẽ tạo ra một khung ảnh mới, đủ lớn để chứa **toàn bộ** hình ảnh sau khi xoay.
    - Kết quả là kích thước của ảnh đầu ra sẽ lớn hơn ảnh gốc, và các vùng trống ở góc sẽ được lấp đầy bằng màu đen (giá trị 0).

2.  **`reshape=False`:**
    - Khi được đặt là `False`, hàm sẽ giữ nguyên kích thước ảnh đầu ra bằng với ảnh gốc.
    - Bất kỳ phần nào của ảnh bị xoay ra ngoài khung ban đầu sẽ bị **cắt bỏ**.

**Lưu ý:**

- Góc xoay được tính bằng đơn vị **độ**, theo thang 360.
- Màu dùng để lấp đầy các vùng trống có thể được thay đổi bằng tham số `cval` (ví dụ: `cval=255` cho màu trắng).

**Code chính:**

```python
# Đọc ảnh gốc
data = iio.imread('fruit.jpg')

# Xoay ảnh, tự động thay đổi kích thước để chứa toàn bộ ảnh (mặc định)
dl = nd.rotate(data, 20)
plt.imshow(dl)
plt.show()

# Xoay ảnh nhưng giữ nguyên kích thước gốc, phần thừa bị cắt bỏ
d2 = nd.rotate(data, 20, reshape=False)
plt.imshow(d2)
plt.show()
```

### 1.5. Dilation và Erosion (Giãn nở và Co lại)

**Mục đích:**

- Thay đổi hình dạng của các đối tượng trong ảnh, cụ thể là làm dày (dilation) hoặc làm mỏng (erosion) các vùng sáng.

**Nguyên lý hoạt động:**

Đây là hai phép toán hình thái học (Morphological Operations) cơ bản, hoạt động trên cấu trúc của các vùng pixel.

1.  **Dilation (`binary_dilation`):**

    - Kết quả là các đối tượng sáng trở nên lớn hơn, các lỗ nhỏ bên trong đối tượng được lấp đầy, và các đối tượng riêng lẻ nằm gần nhau có thể được kết nối lại.

2.  **Erosion (`binary_erosion`):**
    - Kết quả là các đối tượng sáng trở nên nhỏ hơn, các chi tiết nhiễu nhỏ màu trắng sẽ bị loại bỏ, và các cầu nối mỏng manh giữa hai đối tượng có thể bị phá vỡ.
    - **`iterations=3`**: Tham số này chỉ định rằng phép erosion sẽ được thực hiện 3 lần liên tiếp, làm cho hiệu ứng co lại mạnh hơn rất nhiều.

**Lưu ý:**

- Các hàm `binary_dilation` và `binary_erosion` được thiết kế cho ảnh nhị phân (đen/trắng).
- Khi áp dụng lên ảnh xám (`mode="F"`), SciPy sẽ ngầm coi các pixel có giá trị khác 0 là "đối tượng" (màu trắng) và các pixel có giá trị 0 là "nền" (màu đen).

**Code chính:**

```python
# Đọc ảnh dưới dạng ảnh xám (grayscale)
data = iio.imread('world_cup.jpg', mode="F")

# Dilation: Làm các vùng sáng dày và lan rộng ra
a = nd.binary_dilation(data)
plt.imshow(a)
plt.show()

# Erosion: "Bào mòn" các vùng sáng 3 lần liên tiếp
b = nd.binary_erosion(data, iterations=3)
plt.imshow(b)
plt.show()
```

### 1.6. Ánh xạ Tọa độ (Coordinate Mapping)

**Mục đích:**

- Tạo ra các hiệu ứng biến dạng hình ảnh phức tạp và tùy chỉnh, không thể thực hiện bằng các phép biến đổi hình học đơn giản như xoay hay tịnh tiến.
- Hiệu ứng trong code này là làm cho các pixel bị "xáo trộn" ngẫu nhiên tại vị trí của chúng.

**Nguyên lý hoạt động:**

Cốt lõi của kỹ thuật này là **tạo ra một "bản đồ"** chỉ dẫn cho mỗi pixel của ảnh mới rằng nó nên lấy màu từ tọa độ nào của ảnh gốc.

1.  **Tạo Bản đồ Tọa độ Gốc (`np.indices`):**

    - `M = np.indices((V, H))` tạo ra một "bản đồ" của chính bức ảnh. `M` chứa hai lớp: lớp đầu tiên (`M[0]`) là tọa độ Y của mỗi pixel, lớp thứ hai (`M[1]`) là tọa độ X. Về cơ bản, nó là một bản đồ chưa bị biến dạng.

2.  **Tạo Bản đồ Nhiễu (`q`):**

    - Một mảng `q` có cùng kích thước với `M` được tạo ra, chứa các giá trị số thực ngẫu nhiên trong khoảng từ `-d` đến `+d` (ở đây là -5 đến +5).
    - `q` chính là một "bản đồ nhiễu" hay "bản đồ độ lệch", nó sẽ cho biết mỗi pixel cần bị dịch đi bao nhiêu một cách ngẫu nhiên.

3.  **Tạo Bản đồ Đích (`mp`):**

    - `mp = M + q.astype(int)` là bước quyết định. Chúng ta lấy bản đồ gốc `M` và cộng thêm bản đồ nhiễu `q`.
    - Kết quả `mp` là một bản đồ đã bị làm méo. Ví dụ, nó có thể chỉ định rằng pixel ở vị trí (100, 100) trong ảnh mới nên lấy màu từ pixel ở vị trí (100+2, 100-4) của ảnh gốc.

4.  **Thực hiện Ánh xạ (`nd.map_coordinates`):**
    - Hàm này nhận đầu vào là ảnh gốc (`data`) và bản đồ đích (`mp`).
    - Nó sẽ tạo ra ảnh mới bằng cách: với mỗi pixel, nó nhìn vào bản đồ `mp` để xem cần "hút" màu từ tọa độ nào trong ảnh `data`.

**Lưu ý:**

- **Sức mạnh của `map_coordinates`:** Bằng cách thay đổi công thức tính `mp`, có thể tạo ra vô số hiệu ứng biến dạng khác nhau (làm xoáy, gợn sóng, hiệu ứng thấu kính, v.v.).
- **Ngẫu nhiên:** Do sử dụng `np.random.ranf`, mỗi lần chạy lại code, hiệu ứng xáo trộn sẽ khác nhau.

**Code chính:**

```python
# Tạo bản đồ tọa độ gốc: M[0] là tọa độ Y, M[1] là tọa độ X
M = np.indices((V, H))

# Tạo một bản đồ chứa các độ lệch ngẫu nhiên trong khoảng [-5, +5]
d = 5
q = 2 * d * np.random.ranf(M.shape) - d

# Tạo bản đồ đích bằng cách cộng nhiễu vào bản đồ gốc
mp = M + q.astype(int)

# Dùng bản đồ đích để "hút" các pixel từ ảnh gốc, tạo ra ảnh mới
d1 = nd.map_coordinates(data, mp)

plt.imshow(d1)
plt.show()
```

### 1.7. Biến đổi Hình học Tổng quát (Generic Transformation)

**Mục đích:**

- Áp dụng các phép biến đổi hình học phức tạp được định nghĩa bởi một hàm toán học.
- Đây là một cách tiếp cận linh hoạt, cho phép tạo ra các hiệu ứng tùy chỉnh (như gợn sóng, xoắn,...) bằng cách cung cấp một "quy tắc" biến đổi thay vì một "bản đồ" tọa độ có sẵn.

**Nguyên lý hoạt động:**

Hoạt động dựa trên hàm `nd.geometric_transform`. Khác với `map_coordinates` (cần một bản đồ tọa độ đầy đủ), hàm này nhận đầu vào là một **hàm** (`GeoFun`). Hàm này hoạt động theo cơ chế **ánh xạ ngược (inverse mapping)**.

1.  `geometric_transform` sẽ duyệt qua từng pixel trong **ảnh đầu ra** mà nó đang tạo.
2.  Với mỗi tọa độ của pixel đầu ra (`outcoord`), nó sẽ gọi hàm `GeoFun` mà cung cấp.
3.  Hàm `GeoFun` của có nhiệm vụ tính toán và trả về tọa độ tương ứng trong **ảnh gốc**. Nó trả lời câu hỏi: "Pixel ở vị trí `outcoord` của ảnh mới nên lấy màu từ đâu trong ảnh gốc?".
4.  Trong code này, hàm `GeoFun` thêm một giá trị từ hàm `cos` vào tọa độ gốc. Vì `cos` là một hàm tuần hoàn, nó tạo ra một hiệu ứng gợn sóng, uốn lượn trên toàn bộ ảnh.

**Lưu ý:**

- **Sự khác biệt với `map_coordinates`:** `map_coordinates` cần tạo trước một mảng lớn chứa toàn bộ bản đồ tọa độ. `geometric_transform` chỉ cần một hàm quy tắc, giúp tiết kiệm bộ nhớ hơn khi các quy tắc biến đổi phức tạp nhưng có thể tính toán được.
- **Tư duy ngược:** Khi viết hàm cho `geometric_transform`, cần suy nghĩ ngược: "Nếu tôi đang ở điểm `(y, x)` của ảnh mới, tôi cần lấy màu từ điểm nào của ảnh cũ?".

**Code chính:**

```python
# Định nghĩa một "hàm quy tắc" cho việc ánh xạ ngược.
# Nó nhận tọa độ từ ảnh ĐẦU RA (outcoord)
# và trả về tọa độ tương ứng trong ảnh GỐC.
def GeoFun (outcoord) :
    # Thêm một độ lệch dạng sóng cos vào tọa độ Y
    a = 10 * np.cos(outcoord[0]/10.0) + outcoord[0]
    # Thêm một độ lệch dạng sóng cos vào tọa độ X
    b = 10 * np.cos(outcoord[1]/10.0) + outcoord[1]
    return a, b

# Đọc ảnh gốc
data = iio.imread("world_cup.jpg", mode="F")

# Áp dụng quy tắc biến đổi GeoFun lên toàn bộ ảnh
d1 = nd.geometric_transform(data, GeoFun)

plt.imshow(d1)
plt.show()
```
