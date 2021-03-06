
# DNS SERVER – BIND9 
## I. Lý thuyết
### 1. Khái niệm
* DNS ( Domain Name System) là một dịch vụ quan trọng trong hệ thống, thường được triển khai nhằm mục đích hỗ trợ cho việc phân giải tên miền (domain) sang địa chỉ IP và ngược lại
* DNS server trong doanh nghiệp được tạo ra để phân giải một số tên miền quan trọng, giúp tăng tốc độ truy cập, đỡ tốn băng thông, hoặc phân giải một số tên miền chỉ sử dụng nội bộ vì mục đích bảo mật...

* VD: Một máy tính vào trình duyệt truy cập vccloud.vn, các bước sẽ là:
   * Tìm trong file host xem có ip nào trỏ trực tiếp đến domain vccloud.vn hay không

   * Nếu không có, nó hỏi trong DNS server mà mình được config, DNS server này sẽ tìm trong cache của nó để trả lời, nếu có thì trả về luôn, nếu không có hoặc đã hết TTL bị xóa cache thì nó sẽ hỏi root server.

   * Root server sẽ tách tên miền ra thành nhiều phần, ở đây là vccloud và .vn, nó sẽ đi hỏi root name server quản lý tên miền .vn (top-level-domain), sau đó root name server lại tìm trong database xem vccloud.vn ở đâu và trả về cho DNS server, DNS server lại trả về cho resolver trên máy tính để máy tính truy cập

* Primary DNS server: chứa và duy trì cơ sở dữ liệu về thông tin tên miền mà nó quản lý

* Secondary DNS server: là bản backup của Primary DNS server, dùng khi Primary bị lỗi

* Zone: là tập hợp miền hoặc tập hợp các miền con mà DNS server có quyền quản lý, cũng chia thành primary zone và secondary zone

* FQDN: Full Qualifed Domain Name: Tên miền đầy đủ đã được chứng nhận

* SOA: Start of Authority: Record SOA là duy nhất và là nơi cung cấp thông tin tin cậy từ dữ liệu có trong zone, đặt ở đầu file cấu hình zone

Cú pháp: 

    [ Tên miền ] IN SOA [ Tên server dns ] [địa chỉ email] (

              serial number;

              refresh number;

              retry number;

              expire number;

              time to leave number;
    )

* NS: Name server record: Cũng bắt buộc phải có nhưng không giới hạn số lượng

Cú pháp:

    [Tên miền] IN NS [máy DNS server]
             A: Address record: [Tên máy tính] IN A [Địa chỉ IPv4]
             AAAA: cũng giống A nhưng dùng cho IPv6
             CNAME: Canonical name record
             >Là bí danh hay tên truy cập khác của một A
             [Tên bí danh] IN CNAME [tên A record]
             MX: Mail exchange record
                 Khai báo mail server
                 [Tên domain] IN MX [Độ ưu tiên] [Tên mail server]
             PTR: Pointer record
                 Ánh xạ từ địa chỉ sang tên, dùng cho phân giải nghịch 
                 [Địa chỉ IP] IN PTR [Tên máy tính]
## II. Cài đặt Test Environment: BIND9
### 1. Kịch bản
* 3 máy ảo centos7, trong đó tạo 1 primary DNS server (hostname cent1), 1 slave DNS server (hostname cent2) và 1 client (hostname cent3), môi trường mạng ở công ty, các máy ảo dùng card mạng Bridge và Internal phòng khi Bridge bị lỗi.
  * IP của cent1: 192.168.25.32
  * IP của cent2: 192.168.25.50
  * IP của cent3: 192.168.25.45
* Mở 3 tab terminal trên máy thật và ssh đến các máy ảo: root@cent1, root@cent2, root@cent3
### 2. Cài đặt BIND9 trên cent1 và cent2
    $ yum install bind bind-utils
### 3. Cấu hình Primary DNS server
#### a. Cấu hình file named.conf
    root@cent1$ vi /etc/named.conf
Khai báo địa chỉ IP sẽ tiếp nhận các yêu cầu
```
listen-on-port 53 { 127.0.0.1; 192.168.25.32; };
```
Khai báo vị trí chứa các file cấu hình zone
```
directory "/var/named/";
```
Giới hạn các client được phép truy vấn DNS
```
allow-query { localhost; 192.168.25.0/24; };
```
Sử dụng DNS đệ quy (DNS đệ quy - recursive DNS có nghĩa là nếu DNS server không thể trả lời cho một truy vấn thì nó sẽ đi hỏi các DNS server bên ngoài đến khi nào trả lời được thì thôi hoặc thông báo lỗi, DNS tương tác - interactive DNS có nghĩa là DNS sẽ trả về thông tin tốt nhất mà nó có được lúc đó mà không đi hỏin chỗ khác)
```
rucursive yes;
```
Đăng ký các slaves có thể nhận thông tin từ file cấu hình Primary DNS.
```
allow-transfer {192.168.25.50; }; 
```
Đây là zone mặc định khai báo các root DNS server
```
zone "." IN {
type hint;
file "named.ca";
};
```

Khai báo zone phân giải thuận cho tên miền vccloud.vn
```
zone "vccloud.vn" IN {
      #Trên primary kiểu zone là master
      type master;
      #Tên file cấu hình cho zone vccloud.vn
      file "forward.vccloud.vn";
      #Tắt chức năng dynamic update trong zone
      allow-update { none; };
};
```
Khai báo zone phân giải nghịch cho mạng 192.168.25.0/24
```
zone "25.168.192.in-addr.arpa" IN {
      #Trên Primary kiểu zone là master
      type master;
      #Tên file cấu hình cho zone 168.25.192.in-addr.arpa
      file "reserve.vccloud.vn";
      #Tắt dynamic update
      allow-update { none; };
};
```
Gõ :wq để lưu lại và thoát

Kiểm tra lại file named.conf
```
root@cent1 $ named-checkconf /etc/named.conf
```
#### b. Cấu hình zone
Tạo file forward.vccloud.vn trong thư mục /var/named/
    
    $ vi /var/named/forward.vccloud.vn
Gõ i (insert) để edit
```
$TTL 1D

@                   IN SOA  pridns.vccloud.vn. root.vccloud.vn. (
0       ; serial
1D     ; refresh
1H     ; retry
1W    ; expire
3H    ; minimum
)
IN NS              pridns.vccloud.vn.
IN NS              sladns.vccloud.vn.

IN A                192.168.25.45
pridns                IN A                 192.168.25.32
sladns                IN A                 192.168.25.50
websvr               IN A                 192.168.25.45

www                  IN CNAME      websvr.vccloud.vn.

```
Gõ :wq (write-quit) để lưu lại và thoát

Tạo file reverse.vccloud.vn trong thư mục /var/named/

    $ vi /var/named/reverse.vccloud.vn
Gõ i (insert) để edit
```
$TTL 1D
@                   IN SOA pridns.vccloud.vn. root.vccloud.vn. (
0       ; serial
1D     ; refresh
1H     ; retry
1W    ; expire
3H    ; minimum
)

IN NS            pridns.vccloud.vn.
IN NS            sladns.vccloud.vn.
32                     IN PTR          pridns.vccloud.vn.
50                     IN PTR          sladns.vccloud.vn.
45                     IN PTR          websvr.vccloud.vn.
```
Gõ :wq để lưu lại và thoát

Kiểm tra lại file cấu hình zone

    root@cent1 $ named-checkzone vccloud.vn /var/named/forward.vccloud.vn
![Ảnh 1](http://congchungbuiphon.com/wp-content/uploads/2017/09/dns-anh1.png)
    
    root@cent1 $ named-checkzone 25.168.192.in-addr.arpa /var/named/reverse.vccloud.vn
![Ảnh 2](http://congchungbuiphon.com/wp-content/uploads/2017/09/dns-anh2.png)

#### c. Gán quyền, tắt firewall và chạy
Gán quyền read và execute cho các file trong /var/named
    
    $ chmod -R 755 /var/named/
Tắt firewall

    $ systemctl stop firewalld
    $ systemctl disable firewalld
Chạy named
```
$ systemctl enable named
$ systemctl start named
```
Setsebool
```
$ setsebool -P named_tcp_bind_http_port on
$ setsebool -P named_write_master_zones on

$ getsebool -a | grep named
```
![Ảnh 3](http://congchungbuiphon.com/wp-content/uploads/2017/09/dns-anh3.png)
### 4. Cấu hình slave DNS server
#### a. Cấu hình file named.conf
    root@cent2 $ vi /etc/named.conf
Khai báo địa chỉ IP sẽ tiếp nhận các yêu cầu
    
    listen-on-port 53 { 127.0.0.1; 192.168.25.32; };
Khai báo vị trí chứa các file cấu hình zone

    directory "/var/named/";
Giới hạn các client được phép truy vấn DNS

    allow-query { localhost; 192.168.25.0/24; };
Sử dụng DNS đệ quy 

    rucursive yes;
Đây là zone mặc định khai báo các root DNS server
```
zone "." IN {
type hint;
file "named.ca";
};
```
Khai báo zone phân giải thuận cho tên miền vccloud.vn
```
zone "vccloud.vn" IN {
#Trên slave kiểu zone là slave
type slave;
#Tên file cấu hình cho zone vccloud.vn sao chép từ primary DNS server
file "slaves/forward.vccloud.vn";
#Chỉ định primary DNS server để sao chép
masters { 192.168.25.32; };
};
```
Khai báo zone phân giải nghịch cho mạng 192.168.25.0/24
```
zone "25.168.192.in-addr.arpa" IN {
#Trên slaves kiểu zone là slave
type slave;
#Tên file cấu hình cho zone 168.25.192.in-addr.arpa sao chép từ primary DNS server
file "slaves/reserve.vccloud.vn";
#Chỉ định primary DNS server để sao chép
masters { 192.168.25.32; };
};
```
Gõ :wq để lưu lại và thoát

Kiểm tra lại file named.conf giống như đã kiểm tra trên primary DNS server
    
    root@cent2 $ named-checkconf /etc/named.conf
#### b. Gán quyền, tắt firewall và chạy
Gán quyền read và execute cho các file trong /var/named

    $ chmod -R 755 /var/named/
Tắt firewall

    $ systemctl stop firewalld
    $ systemctl disable firewalld
Chạy named
```
$ systemctl enable named
$ systemctl start named
```
Setsebool
    
    $ getsebool -a | grep named
![Ảnh 4](http://congchungbuiphon.com/wp-content/uploads/2017/09/dns-anh4.png)
```
$ setsebool -P named_tcp_bind_http_port on
$ setsebool -P named_write_master_zones on

$ getsebool -a | grep named
```
### 5. Kiểm tra trên host
Cấu hình DNS server cho resolver

    root@cent3 $ vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
    
Ấn i, Thêm dòng cấu hình DNS
    
    DNS1=192.168.25.32
![Ảnh 5](http://congchungbuiphon.com/wp-content/uploads/2017/09/dns-anh5.png)

Gõ :wq để lưu lại và thoát
    
    root@cent3 $ nslookup vccloud.vn
![Ảnh 6](http://congchungbuiphon.com/wp-content/uploads/2017/09/dns-anh6.png)

Thử với google

    root@cent3 $ nslookup 8.8.8.8
![Ảnh 7](http://congchungbuiphon.com/wp-content/uploads/2017/09/dns-anh7.png)
    
    root@cent3 $ nslookup google.com
![Ảnh 8](http://congchungbuiphon.com/wp-content/uploads/2017/09/dns-anh8.png)

**Như vậy chúng ta đã cài đặt và kiểm tra thành công DNS server – BIND9 trên centos7.** :smile:
