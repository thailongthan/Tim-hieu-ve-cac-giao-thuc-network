# GIAO THUC ARP - BAO CAO
## I. Lý thuyết
### 1. ARP là gì
Giao thức ARP (Address Resolution Protocol - Giao thức phân giải địa chỉ) là một cơ chế giúp chuyển đổi từ địa chỉ logic IP sang địa chỉ vật lý MAC, do trên thực tế các máy tính chỉ có thể giao tiếp với nhau thông qua địa chỉ MAC của card mạng.

Giao thức RARP (Reverse Address Resolution Protocol) giúp chuyển đổi từ địa chỉ MAC sang địa chỉ IP
### 2. Nguyên tắc hoạt động
#### a. Trong mạng LAN
Sơ đồ

![Ảnh 1](https://letonphat.files.wordpress.com/2010/11/image_thumb51.png?w=309&h=242)

Giả sử host A và host B trong cùng 1 mạng LAN, host A muốn biết địa chỉ MAC của host B.

Các bước sẽ là:
* Host A gửi broadcast ARP request trên toàn bộ mạng LAN, gói tin bao gồm địa chỉ MAC của A và địa chỉ IP của B
* Mỗi thiết bị trong mạng LAN này sẽ phải so sánh địa chỉ IP trên với địa chỉ IP của mình, host B có IP trùng với Request của host A sẽ gửi ngược lại cho host A (đã có địa chỉ MAC trong gói tin host A gửi đi) một gói tin trong đó chứa địa chỉ MAC của mình.
* Như vậy host A đã có địa chỉ IP và địa chỉ MAC của host B
#### b. Trong mạng WAN hoặc lớn hơn
Sơ đồ


Giả sử host A và host B ở 2 mạng khác nhau, host A muốn biết địa chỉ MAC của host B. 2 host chung router C.

Các bước sẽ là:
* Host A gửi broadcast ARP request trên toàn bộ mạng LAN, tìm địa chỉ MAC của port X qua default gateway
* Router C trả lời, gửi lại cho A địa chỉ MAC của port X
* Host A gửi gói tin chứa địa chỉ MAC của A và địa chỉ IP của B đến port X
* Router C đọc thông tin địa chỉ IP của B, chuyển gói tin qua port Y nối với mạng của B, sau đó gửi broadcast ARP request tìm địa chỉ MAC của B
* Host B trả lời thông tin địa chỉ MAC của mình cho router C, Router C chuyển tiếp thông tin này về host A
