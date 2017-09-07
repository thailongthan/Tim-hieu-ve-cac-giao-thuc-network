# GIAO THUC ARP - BAO CAO
## I. Lý thuyết
### 1. ARP là gì
Giao thức ARP (Address Resolution Protocl - Giao thức phân giải địa chỉ) là một cơ chế giúp chuyển đổi qua lại giữa địa chỉ vật lý MAC và địa chỉ logic IP, do trên thực tế các máy tính chỉ có thể giao tiếp với nhau thông qua địa chỉ MAC của card mạng.
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
