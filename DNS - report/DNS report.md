
# DNS SERVER – BIND9 
## Lý thuyết
### Khái niệm
* DNS ( Domain Name System) là một dịch vụ quan trọng trong hệ thống, thường được triển khai nhằm mục đích hỗ trợ cho việc phân giải tên miền (domain) sang địa chỉ IP và ngược lại
* DNS server trong doanh nghiệp được tạo ra để phân giải một số tên miền quan trọng, giúp tăng tốc độ truy cập, đỡ tốn băng thông, hoặc phân giải một số tên miền chỉ sử dụng nội bộ vì mục đích bảo mật...

VD: Một máy tính vào trình duyệt truy cập vccloud.vn, các bước sẽ là:

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
Là bí danh hay tên truy cập khác của một A
[Tên bí danh] IN CNAME [tên A record]
MX: Mail exchange record
Khai báo mail server
[Tên domain] IN MX [Độ ưu tiên] [Tên mail server]
PTR: Pointer record
Ánh xạ từ địa chỉ sang tên, dùng cho phân giải nghịch
[Địa chỉ IP] IN PTR [Tên máy tính]
