# Cài đặt Cobbler


# Mục lục

- [1.Mô hình triển khai](#1)

- [2.Các bước cài đặt và cấu hình](#2)
  + [2. 1. Bước 1: Cài đặt EPEL-repo trên Centos](#2.1)

  + [2. 2. Bước 2: Cài đặt Cobbler và một số gói cần thiết](#2.2)

  + [2. 3. Bước 3: Kích hoạt các dịch vụ](#2.3)

  + [2. 4. Bước 4: Truy cập vào giao diện Web](#2.4)

  + [2. 5. Bước 5: Cấu hình Cobbler](#2.5)

  + [2. 6. Bước 6: Xác nhận lại distro được import vào cobbler](#2.6)

  + [2. 7. Bước 7: Tạo file kickstart](#2.7)

  + [2. 8. Bước 8: Sửa file cấu hình boot](#2.8)
  
- [3.Boot OS từ client](#3)

- [4.Tham khảo](#4)  

<a name = '1'></a>
# 1.Mô hình triển khai

- ![]( /image/mohinh.png)

-  Cobbler Server: cài trên hệ điều hành Centos 7 với địa chỉ IP là 192.168.149.128

-  Client: là  máy ảo KVM được tạo ra với một card mạng thuộc dải default-net. Chưa được cài đặt Hệ điều hành.

<a name = '2'></a>
# 2.Các bước cài đặt và cấu hình

<a name = '2.1'></a>
# 2.1 Bước 1: Cài đặt EPEL-repo trên Centos

- Cài đặt Epel-repo thực hiện lệnh sau:
	` yum install epel-release `

<a name = '2.2'></a>	
# 2.2 Bước 2: Cài đặt Cobbler và một số gói cần thiết

- `yum install cobbler cobbler-web dnsmasq syslinux xinetd bind bind-utils dhcp debmirror pykickstart fence-agents-all -y`

  Trong đó:
  
- `Cobbler`, `cobbler-web`: các gói phần mềm cài đặt chạy dịch vụ cobbler và giao diện web của cobbler.

- `Dnsmasq, bind, bind-utils, dhcp` : các gói phần mềm chạy dịch vụ quản lý DNS và quản lý DHCP cho các máy client boot OS từ cobbler.

-` Syslinux` : là một chương trình bootloader và tiện ích cho phép đẩy vào client cho phép client boot OS qua mạng. (trong trường hợp này nó được gọi là pxelinux)

- `Xinetd`: chịu trách nhiệm tạo socket kết nối với máy client. Dựa vào cổng và giao thức (tcp hay udp) nó biết được phải trao đổi dữ liệu mà nó nhận được với

 back-end nào dựa vào thuộc tính server trong file cấu hình. Được sử dụng để quản lý và tạo socket cho TFTP server truyền file boot cho client.
 
- `Debmirror`: gói phần mềm cài đặt cho phép tạo một mirror server chứa các gói phần mềm cài đặt của các distro trên một server local (ở đây cài luôn lên cobbler)

- `Pykickstart` : thư việc python cho phép đọc và chỉnh sửa nội dung file kickstart, hỗ trợ cobbler chỉnh sửa file kickstart thông qua giao diện web.


<a name = '2.3'></a>
# 2.3 Bước 3: Kích hoạt các dịch vụ

 Kích hoạt và khởi động các dịch vụ cobblerd và httpd:
 
-  systemctl start cobblerd
-  systemctl enable cobblerd
-  systemctl start httpd
-  systemctl enable httpd
-  Disable SELinux:
  + Thực hiện các lệnh sau để disable tính năng của SELinux:
  +	sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/sysconfig/selinux
  +	sed -i 's/\(^SELINUX=\).*/\SELINUX=disabled/' /etc/selinux/config++
  +	setenforce 0
  +	Khởi động lại máy và thực hiện bước tiếp theo.
  
-	Thực hiện các lệnh sau nếu OS chạy firewall:
  +	firewall-cmd --add-port=80/tcp --permanent
  +	firewall-cmd --add-port=443/tcp --permanent
  +	firewall-cmd --add-service=dhcp --permanent
  +	firewall-cmd --add-port=69/tcp --permanent
  +	firewall-cmd --add-port=69/udp --permanent
  +	firewall-cmd --add-port=4011/udp --permanent
  +	firewall-cmd –reload

<a name = '2.4'></a>
# 2.4 Bước 4: Truy cập vào giao diện Web

- Truy cập vào giao diên web của Cobbler: ` https://192.168.149.128/cobbler_web/ `

- ![]( /image/giaodien.png)

<a name = '2.5'></a>
# 2.5 Bước 5: Cấu hình Cobbler

### Sửa file cấu hình Cobbler /etc/cobbler/settings

- Sử dụng `openssl` để sinh ra mật khẩu cho cobbler đã được mã hóa như sau:

	```
	openssl passwd -1
	Password:
	Verifying - Password:
	$1$Na4V6QL0$f1RXxgBEqLdlJI.lXQVQI0

	```

	(Mật khẩu tôi dùng ở đây vẫn sử dụng là cobbler, sinh ra được đoạn password đã được mã hóa như trên)

- Sửa file `/etc/cobbler/settings` với các thông số `default_password_crypted` với password vừa sinh ra ở trên, và cập nhật các thông số của DHCP, DNS, PXE từ 0 lên 1 như sau: 

	```
	default_password_crypted: "$1$Na4V6QL0$f1RXxgBEqLdlJI.lXQVQI0 "
	manage_dhcp: 1
	manage_dns: 1
	pxe_just_once: 1
	next_server: 192.168.149.128
	server: 192.168.149.128
	```
	
- ### Cập nhật file cấu hình DHCP và DNSMASQ

- Sửa file cấu hình của DHCP như sau `vi /etc/cobbler/dhcp.template`

	```
	[...]
	subnet 192.168.149.0  netmask 255.255.255.0 {
	     option routers             192.168.149.1;
	     option domain-name-servers 8.8.8.8;
	     option subnet-mask         255.255.255.0;
	     range dynamic-bootp        192.168.149.120 192.168.149.125;
	     default-lease-time         21700;
	     max-lease-time             43100;
	     next-server                $next_server;

	     class "pxeclients" {
	          match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
	          if option pxe-system-type = 00:02 {
	                  filename "ia64/elilo.efi";
	          } else if option pxe-system-type = 00:06 {
	                  filename "grub/grub-x86.efi";
	          } else if option pxe-system-type = 00:07 {
	                  filename "grub/grub-x86_64.efi";
	          } else {
	                  filename "pxelinux.0";
	          }
	     } 
	}
	```
	
- Cập nhật dải địa chỉ IP được cấp phát cho client trong file `/etc/cobbler/dnsmasq.template` như sau:

	```
	[...]
	dhcp-range=192.168.149.120, 192.168.149.125
	```	
	
- Thực hiện comment `@dists` và `@arches` trong file `/etc/debmirror.conf` để hỗ trợ các distro debian

- Khởi động lại rsyncd, cobbler và xinetd, sau đố đồng bộ lại cobbler dùng các lệnh sau:

	```
	systemctl enable rsyncd.service
	systemctl restart rsyncd.service
	systemctl restart cobblerd
	systemctl restart xinetd
	systemctl enable xinetd
	cobbler get-loaders
	cobbler check
	cobbler sync
	```
- Thực hiện mount 2 file iso và import vào cobbler như sau:

	```
	mkdir  /mnt/centos
	mkdir /mnt/us
	mount -o loop CentOS-7-x86_64-DVD-1708.iso  /mnt/centos/
	cobbler import --arch=x86_64 --path=/mnt/centos --name=CentOS7
	umount /mnt/centos
	mount -o loop  ubuntu-16.04.2-server-amd64.iso  /mnt/us/
	cobbler import --arch=x86_64 --path=/mnt/us --name=US1604
	```
<a name = '2.6'></a>	
# 2.6.	Bước 6: Xác nhận lại distro được import vào cobbler

- ![]( /image/import1.png)

<a name = '2.7'></a>
# 2.7.	Bước 7: Tạo file kickstart

Thư mục chứa các file kickstart là `/var/lib/cobbler/kickstarts`. Tạo các file kickstart cho Centos 7 và Ubuntu server có nội dung như sau.

	- File kickstart cho Centos 7 với tên [CentOS7.ks](../config_files/Centos7-cobbler.ks)

	- File kickstart cho Ubuntu server 16.04 với tên [US1604.cfg.ks](../config_files/US1604-cobbler.ks)

- Đồng bộ và cập nhật file kickstart cho các profile của Centos 7 và Ubuntu server 16.04 như sau: 

	```
	cobbler profile edit --name=CentOS7-x86_64 --kickstart=/var/lib/cobbler/kickstarts/CentOS7.ks
	cobbler profile edit --name=US1604-x86_64 --kickstart=/var/lib/cobbler/kickstarts/US1604.cfg.ks
	cobbler sync
	```	

<a name = '3'></a>	
# 3. Boot OS từ các client

- Bật client 1 và client 2. Sau khi nhận IP, màn hình boot hiển thị nội dung như sau:

- ![]( /image/boot1.png)

- Chọn OS muốn cài đặt và ấn ENTER.

- ![]( /image/boot2.png)

- Sau khi boot thành công.

- ![]( /image/boot3.png)

- Kiểm tra trên cobbler server thấy đã cấp phát DHCP cho các client như sau:

- ![]( /image/boot4.png)

<a name = '4'></a>
# 4.Tham khảo

- [1] https://github.com/hocchudong/thuctap012017/blob/master/TamNT/PXE-Kickstart-Cobbler/docs/5.Cobbler-cai_dat.md
