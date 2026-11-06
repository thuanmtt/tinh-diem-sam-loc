# Ứng dụng Tính Điểm

Link: https://thuanmtt.github.io/tinh-diem-sam-loc/

Ứng dụng web đơn giản giúp tính toán và quản lý điểm số trong các trò chơi, với khả năng tự động tính toán việc chia tiền giữa những người chơi.

![image](https://github.com/user-attachments/assets/13da6db7-8ee7-4650-b104-c26f20fe3815)

## Tính năng

- ✅ Thêm nhiều người chơi
- ✅ Nhập điểm số cho mỗi ván đấu
- ✅ Tự động tính toán điểm tổng
- ✅ Xem lịch sử các ván đấu
- ✅ Tính toán và hiển thị cách chia tiền công bằng
- ✅ Chế độ xem tối/sáng
- ✅ Lưu dữ liệu trên thiết bị (localStorage)
- ✅ Giao diện thân thiện với người dùng, responsive

## Cách sử dụng

### Thêm người chơi

1. Nhập tên người chơi vào ô "Nhập Tên Người Chơi"
2. Nhấn nút "Thêm" để thêm người chơi vào danh sách

### Nhập điểm

1. Nhấn nút "Nhập ván mới"
2. Chọn người thắng ván đó
3. Lần lượt nhập số điểm của từng người thua, nhấn "Lưu" để sang người tiếp theo (để trống là 0)
4. Đến người cuối, nhấn "Lưu ván" là xong. Bấm vào tên ở hàng trên hoặc "Quay lại" để sửa người đã nhập

Người thắng tự động nhận tổng điểm của những người thua. Ví dụ A thắng, B thua 5, C thua 3 thì A được +8.

### Xem lịch sử

Nhấn nút "Xem Lịch Sử" để xem tất cả các ván đấu đã ghi nhận.

### Chia tiền

Nhấn nút "Chia Tiền" để xem gợi ý về cách chia tiền công bằng giữa các người chơi dựa trên điểm số.

### Xóa dữ liệu

Nhấn nút "Xóa Điểm" để xóa tất cả dữ liệu người chơi và lịch sử đấu.

## Cài đặt

Không cần cài đặt phức tạp. Ứng dụng chạy hoàn toàn trên trình duyệt.

1. Tải tất cả các file về máy
2. Mở file `index.html` trong trình duyệt web

## Công nghệ sử dụng

- HTML5
- CSS3
- JavaScript (jQuery)
- Bootstrap 5
- Font Awesome 6
- SweetAlert2
- LocalStorage API
