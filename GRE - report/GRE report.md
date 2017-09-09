# GIAO THỨC GRE - BÁO CÁO
## I. Lý thuyết
### 1. Khái niệm
GRE - Generic Routing Encapsulation là một giao thức do Cisco phát triển, cho phép đóng gói một số loại gói tin vào bên các trong đường hầm IP (IP tunnels) để tạo thành các kết nối point to point ảo tới Cisco router cần đến.
### 2. Đặc điểm
GRE thêm tối thiểu 24 byte vào gói tin, trong đó có 20 byte là IP header mới, 4 byte là GRE header, ngoài ra còn có tùy chọn thêm 12 byte mở rộng để có thêm các tùy chọn checksum, key chứng thực, sequence number
![Ảnh 1](http://www.vnpro.vn/wp-content/uploads/2016/05/H%C3%ACnh-4.1-%C4%90%E1%BB%8Bnh-d%E1%BA%A1ng-packet-%C4%91%C6%B0%E1%BB%A3c-%C4%91%C3%B3ng-g%C3%B3i-v%E1%BB%9Bi-GRE.jpg)

GRE có thể tạo IP tunnels cho bất kỳ gói tin lớp 3 nào

GRE cung cấp khả năng có thể định tuyến giữa những mạng riêng (private network) thông qua môi trường internet

GRE không có cơ chế mã hóa, không có cơ chế hash và không có cơ chế xác thực

GRE không có cơ chế bảo mật tốt. Do đó nhà quản trị thường kết hợp GRE với IPSec để tăng tính bảo mật, đồng thời cũng hỗ trợ IPSec trong việc định tuyến và truyền những gói tin có địa chỉ IP Muliticast.
### 3. Phân loại
GRE truyền thống là point to point, sau này mGRE mở rộng ra, cho phép một tunnel có thể đến được địa chỉ multicast
#### a. GRE point to point
![Ảnh 2](http://blog.ine.com/wp-content/uploads/2008/08/dmvpn-p12-gre-tunnels.jpg)

Đối với các tunnel GRE point-to-point thì trên mỗi router spoke (R2 & R3) cấu hình một tunnel chỉ đến HUB (R1) ngược lại, trên router HUB cũng sẽ phải cấu hình hai tunnel, một đến R2 và một đến R3. Mỗi tunnel như vậy thì cần một địa chỉ IP. Giả sử mô hình trên được mở rộng thành nhiều spoke, thì trên R1 cần phải cấu hình phức tạp và tốn không gian địa chỉ IP.

Nếu địa chỉ đích là multicast thì GRE không giải quyết được, khi đó ta cần mGRE
#### b. mGRE point to multipoint
![Ảnh 3](http://media.packetlife.net/media/blog/attachments/60/DMVPN_lab2.png)

mGRE dùng giao thức NHRP (Next Hop Resolution Protocol) ánh xạ từ địa chỉ multipoint sang địa chỉ cổng vật lý của router.
### 4. Kết hợp GRE và IPSec
#### a. Khái niệm IPSec
IPSec là viết tắt của IP Security, là giao thức giúp xác thực và mã hóa cho mỗi gói tin IP trong quá trình truyền thông, điều khiển truy nhập, bảo mật

IPSec được sử dụng như một chức năng xác thực và được gọi là Authentication Hearder (AH).

Được dùng trong việc chứng thực/mã hóa, kết hợp chức năng(authentication và integrity) gọi là Encapsulating Security Payload (ESP).

Đảm bảo tính nguyên vẹn của dữ liệu.

Chống quá trình replay trong các phiên bảo mật.

Trong các phạm vi sử dụng của Ipsec thì việc xác thực và mã hóa được chú ý và quan tâm nhiều nhất khi ứng dụng trên các mạng riêng ảo (VPN) để đảm bảo tính bảo mật cao cho người sử dụng.
#### b. GRE over IPSec
GRE không bảo mật nên cần kết hợp cùng IPSec để tăng tính bảo mật cho kênh truyền

IPSec khi kết hợp cùng GRE sẽ được khả năng định tuyến động trên kênh truyền, giúp tạo ra khả năng mở rộng hệ thống mạng
#### c. Cơ chế GRE over IPSec
Các gói tin GRE được truyền thông đi qua kênh truyền bảo mật do IPSec thiết lập, điều này thực hiện được là do IPSec đóng gói và thêm các thông tin bảo mật của mình vào gói tin GRE 
 
IP packet ban đầu được đóng gói bởi GRE header sau đó IPSec sẽ thêm các thông tin IPSec header để cung cấp thêm các tính năng bảo mật rồi mới truyền gói tin đi. Khi gói tin đến đích thì quá trình bóc tách sẽ diễn ra ngược lại 

GRE over IPSec có 2 chế độ hoạt động: Tunnel mode và Transport mode 
![Ảnh 4](http://img1.51cto.com/attachment/201205/174346171.png)
