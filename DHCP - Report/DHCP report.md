# Báo cáo DHCP

## I. Lý Thuyết
### 1. DHCP là gì
DHCP (Dynamic Host Configuration Protocol) là dịch vụ tự động gán IP cho các client khi chúng vào trong mạng. Nếu không có DHCP, cấu hình IP phải được thực hiện một cách thủ công cho các máy tính mới, các máy tính di chuyển từ mạng con này sang mạng con khác, và các máy tính được loại bỏ khỏi mạng.

DHCP giúp quản lý IP một cách tự động và tập trung, tránh tình trạng trùng lặp IP

### 2. Các bước hoạt động
Mô hình hoạt động
![Ảnh 1](https://camo.githubusercontent.com/053e7de1db3fb086f807d54fd993d1f8a0060eea/687474703a2f2f692e696d6775722e636f6d2f79666b50544c782e706e67)

## II. Building a test environment
Kịch bản: tạo 2 máy ảo centos 7, 1 máy làm DHCP server root@dhcpserver, 1 máy làm client root@cent2
Thực hiện
// Đặt hostname cho DHCP server

$ hostnamectl set-hostname dhcpserver

//Cài đặt DHCP cho dhcp server:

root@dhcpserver $ yum install dhcpd -y

//Đặt IP tĩnh cho DHCP server

root@dhcpserver $ vi /etc/sysconfig/network-scripts/ifcfg-enp0s3

![Ảnh 1](http://congchungbuiphon.com/wp-content/uploads/2017/09/anh1.jpg.png)

//Cấu hình DHCP server

root@dhcpserver $ vi /etc/dhcp/dhcpd.conf

//Ở đây ta nhập thử 2 domain name là "vccloud.vn" và "svr.vccloud.vn"

![Ảnh 2](http://congchungbuiphon.com/wp-content/uploads/2017/09/anh2.jpg.png)

//Kiểm tra lại file dhcpd.conf

root@dhcpserver $ /usr/sbin/dhcpd -t -cf /etc/dhcp/dhcpd.conf

//Trả lại kết quả báo 2 subnet có giao nhau, nhưng vẫn thử chạy xem sao

![Ảnh 3](http://congchungbuiphon.com/wp-content/uploads/2017/09/anh3.png)

//Khởi động dịch vụ dhcp

root@dhcpserver $ systemctl start dhcpd

//Kiểm tra trên cent2

//Cấu hình network cho cent2 bootproto=dhcp

root@cent2 $ vi /etc/sysconfig/network-scripts/ifcfg-enp0s3

![Ảnh 4](http://congchungbuiphon.com/wp-content/uploads/2017/09/anh4.png)

//Khởi động lại card mạng trên cent2 xem nhận IP là gì

root@cent2 $ ifdown enp0s3

root@cent2 $ ifup enp0s3

root@cent2 $ ip addr

