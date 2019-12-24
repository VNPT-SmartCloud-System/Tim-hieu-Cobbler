
# Tổng quan về Cobbler

# 1.Mục đích ra đời

- Khi cài đặt hệ điều hành linux cho một thiết bị thì phải qua các bước như chọn ngôn ngữ,thời gian,phân vùng ổ cứng,đặt password,... nhưng nếu muốn cài với một
lượng thiết bị lớn thì tốn rất nhiều thời gian. Và Cobbler đã xuất hiện để tự động hóa nhiều công đoạn khác nhau trong quá trình cài đặt hệ điều hành linux,dễ
dàng hơn trong việc cài đặt số lượng lớn hệ điều hành linux với những cấu hình khác nhau.

# 2.Cobbler là gì? Chức năng của Cobbler?

- Cobbler là một gói công cụ tiện ích cho phép triển khai hoàn chỉnh một máy chủ PXE server với khả năng cài đặt tự động các phiên bản Linux thông qua môi trường
 mạng
 
- Chức năng chính hỗ trợ cài đặt tự động các bản hệ điều hành linux thông quan mạng, sử dụng kickstart file để tự động hóa các bước cài đặt.

- Ngoài ra Cobbler còn support deploy OS qua mạng mà không cần sự hỗ trợ của DHCP truyền thống thích hợp cho một số trường hợp triển khai khác nhau.

# 3.Các thành phần chính của Cobbler

- `Kickstart file` :kickstart file là file chứa tất cả những câu trả lời cần thiết cho việc cài đặt các distro Linux nhờ vào file trả lời này mà toàn bộ quá trình
 cài đặt sẽ được tự hóa hoàn toàn. 
 
- `TFTP, FTP`: Là các giao thức mà cobbler sử dụng để truyển tải các file cài đặt từ cobbler server đến các client để cài linux 

- `DHCP server` : để phục vụ cho việc cài đặt qua mạng, máy trạm phải kết nối được đến server và được cấp 1 địa chỉ IP. Quá trình cấp địa chỉ này được thực hiện 
 bởi DHCP server
 
- `DNS server` : Giúp thể gán địa chỉ IP với 1 tên miền (là thành phần không bắt buộc).

- `Web server`: Cobbler cung cấp giao diện web cho phép người quản trị thông qua đó, quản lý các profile cũng như các máy trạm được cài đặt. 

# 4.Các đối tượng trong Cobbler.

- `Distro`: chứa các thông tin về kernel và initrd nào được sử dụng, bao gồm cả các dữ liệu dùng để cài đặt. Hiểu 1 cách đơn giản. Đấy chính là đĩa cài 
đặt của Linux của ta

- `System`: Gồm profile và MAC address. Đại diện cho các máy client được cung cấp, chỉ tới một profile hoặc một image và chứa thông tin về IP và địa chỉ MAC, 
quản lý tài nguyên và nhiều loại data chuyên biệt

- `Profile` = Distribution + kickstart file + các gói cài đặt thêm (nếu cần)

- `Repo` : là nơi chứa các gói cài đặt thêm.

- ![]( /image/cobbler1.png)

- Như vậy có thể thấy trình tự cấu hình cobbler như sau:
  + tạo distro
  + tạo profile
  + tạo repo
  + add System
  + boot máy trạm và chờ kết quả
  
  # Tham khảo 
  
  - [1] https://news.cloud365.vn/cobbler-tong-quan-ve-cobbler/
  - [2] https://github.com/hocchudong/thuctap012017/tree/master/TamNT/PXE-Kickstart-Cobbler
