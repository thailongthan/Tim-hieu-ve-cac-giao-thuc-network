# Báo cáo DHCP

## Lý Thuyết
### DHCP là gì
DHCP (Dynamic Host Configuration Protocol) là 

## Building a test environment
Kịch bản: tạo 2 máy ảo centos 7, 1 máy làm DHCP server root@dhcpserver, 1 máy làm client root@cent2
Thực hiện
// Đặt hostname cho DHCP server

$ hostnamectl set-hostname dhcpserver

//Cài đặt DHCP cho dhcp server:

root@dhcpserver $ yum install dhcpd -y

//Đặt IP tĩnh cho DHCP server

root@dhcpserver $ vi /etc/sysconfig/network-scripts/ifcfg-enp0s3

[Ảnh 1](http://congchungbuiphon.com/wp-content/uploads/2017/09/anh1.jpg.png)

//Cấu hình DHCP server

root@dhcpserver $ vi /etc/dhcp/dhcpd.conf

//Ở đây ta nhập thử 2 domain name là "vccloud.vn" và "svr.vccloud.vn"

[Ảnh 2](http://congchungbuiphon.com/wp-content/uploads/2017/09/anh2.jpg.png)

//Kiểm tra lại file dhcpd.conf

root@dhcpserver $ /usr/sbin/dhcpd -t -cf /etc/dhcp/dhcpd.conf

//Trả lại kết quả báo 2 subnet có giao nhau, nhưng vẫn thử chạy xem sao

[Ảnh 3](http://congchungbuiphon.com/wp-content/uploads/2017/09/anh3.png)

//Khởi động dịch vụ dhcp

root@dhcpserver $ systemctl start dhcpd

//Kiểm tra trên cent2

//Cấu hình network cho cent2 bootproto=dhcp

root@cent2 $ vi /etc/sysconfig/network-scripts/ifcfg-enp0s3

[Ảnh 4](http://congchungbuiphon.com/wp-content/uploads/2017/09/anh4.png)

//Khởi động lại card mạng trên cent2 xem nhận IP là gì

root@cent2 $ ifdown enp0s3

root@cent2 $ ifup enp0s3

root@cent2 $ ip addr

