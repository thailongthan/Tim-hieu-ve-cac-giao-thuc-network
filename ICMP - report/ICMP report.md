# GIAO THỨC ICMP - BÁO CÁO

## I. Lý thuyết

### 1. ĐỊNH NGHĨA

Giao thức ICMP (Internet Control Message Protocol) là một giao thức cho phép kiểm tra và xác định lỗi trên layer 3 - network trong mô hình OSI bằng cách định nghĩa ra các loại thông điệp chuyên dụng khác nhau có thể dùng để xem mạng hiện tại có thể truyền được gói tin hay không.

ICMP không giải quyết được vấn đề unreliability của IP

Các loại ICMP message được đóng gói với header đều có 3 trường sau
  * Type (8 bit): Chỉ ra kiểu của ICMP hay chính là thông điệp được gửi về
  * Code (8 bit): Chứa thông điệp con, bổ sung thông tin cho trường type
  * Checksum (16 bit)
  
Sau đó là data.

Ví dụ: 
  * ICMP echo request có type = 0, code =0
  * ICMP echo reply có type = 8, code = 0

### 2. Các loại thông điệp ICMP

* ICMP echo messages: Gồm ICMP echo request và ICMP echo reply, dùng trong lệnh ping
* ICMP destination unreachable message: xảy ra lỗi khiến gói tin không đi được đến đúng đích, khi này thiết bị trung gian sẽ gửi lại một thông điệp Destination Unreachable về sender. Thông điệp này có nhiều loại và ứng với các nguyên nhân khác nhau như
  * Type = 3, code = 0 -> Network unreachable: Bị lỗi định tuyến
  * Type = 3, code = 1 -> Host unreachable: địa chỉ IP, subnet hoặc default gateway không đúng khiến gói tin không đến được địa chỉ. Hoặc đến được nhưng bị firewall chặn
* ICMP redirect: Là một loại thông điệp điều khiển. Ví dụ host A gửi gói tin ra ngoài qua default gateway đã được cấu hình sẵn là tại router B, nhưng router B biết một đường đi từ A ra ngoài ngắn hơn là qua router C dựa vào bảng định tuyến của mình. Router B gửi thông điệp này tới host A để host A bổ sung vào bảng định tuyến của mình, lần sau gửi thẳng qua C luôn mà không đi qua B nữa
* ICMP Time Exceeded: Gói tin bị hủy do thời gian truyền tin quá dài, xác định bởi trường TTL. Gói tin được đặt trước TTL, mỗi khi đi qua một hop thì TTL giảm đi 1. Khi TTL bị giảm về 0 thì gói tin bị hủy để tránh gói tin bị gửi mãi gây ra tắc nghẽn trong mạng
