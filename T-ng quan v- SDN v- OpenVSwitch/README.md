# Tổng quan về SDN và OpenVSwitch 
## 1. SDN - Software Defined Networking 
SDN hay mạng điều khiển bằng phần mềm được dựa trên cơ chế tách riêng việc kiểm soát một luồng mạng với luồng dữ liệu (control plane &  data plane) 

SDN dựa trên giao thức luồng mở (OpenFlow) 

SDN tách định tuyến và chuyển các luồng dữ liệu riêng rẽ và chuyển kiểm soát luồng sang thành phần mạng riêng có tên là thiết bị kiểm soát luồn - Flow controler 

--> Luồng các gói dữ liệu đi qua mạng được kiểm soát theo lập trình 

Trong SDN, control plane được tách ra từ các thiết bị vật lý và chuyển đến các bộ điều khiển. Bộ điều khiển này có thể nhìn thấy toàn bộ mạng lưới và do đó cho phép kỹ sư mạng làm cho chính sách chuyển tiếp được tối ưu dựa trên toàn bộ mạng. Các bộ điều khiển tương tác với các thiết bị mạng vật lý thông qua một giao thức chuẩn OpenFlow 

Kiến trúc của SDN gồm 3 lớp riêng 

* Lớp ứng dụng: là các ứng dụng kinh doanh được triển khai trên mạnkg lưới, được kết nối tới lớp điều khiển thông qua các API, cung cấp khả năng cho phép lớp ứng dụng lập trình lại (cấu hình lại) mạng (điều chỉnh các tham số độ trễ, băng thông, định tuyến,...) thông qua lớp điều khiển 
* Lớp điều khiển: Là nơi tập trung các bộ điều khiển thực hiện việc điều khiển cấu hình mạng theo các yêu cầu từ lớp ứng dụng, khả năng của mạng. Các bộ điều khiển này có thể là các phần mềm được lập trình. 
* Lớp cơ sở hạ tầng: Là các thiết bị mạng thực tế (vật lý hay ảo hóa) thực hiện việc chuyển tiếp gói tin theo sự điều khiển của lớp điều khiển. Một thiết bị mạng có thể hoạt động theo sự điều khiển của nhiều bộ điều khiển khác nhau, điều này giúp tăng cường khả năng ảo hóa của mạng. 
 
## 2. OpenFlow 
OpenFlow là tiêu chuẩn đầu tiên, cung cấp khả năng truyền thông gữa các giao diện của lớp điều khiển và lớp chuyển tiếp trong kiến trúc SDN. OpenFlow cho phép truy cập trực tiếp và điều khiển mặt phẳng chuyển tiếp của thiết bị mạng như Switch và Router, cả thiết bị vật lý và thiết bị ảo, do đó giúp di chuyển phần điều khiển mạng ra khỏi các thiết bị chuyển mạch thực tế tới phần mềm điều khiển trung tâm. 

Các quyết định về các luồng traffic sẽ được quyết định tập trung tại OpenFlow controller giúp đơn giản trong việc quản trị cấu hình cho toàn hệ thống. 

Một thiết bị OpenFlow bao gồm ít nhất 3 thành phần: 
 
* Secure channel: Kênh kết nối thiết bị tới bộ điều khiển (controller) cho phép các kênh và các gói tin được gửi giữa bộ điều khiển và thiết bị. 
* OpenFlow protocol: giao thức cung cấp phương thức tiêu chuẩn và mở cho một bộ điều khiển truyền thông tới thiết bị. 
* Flow table: Là một liên kết hành động với mỗi luồng, giúp thiết bị biết cách xử lý các luồng như thế nào. 
 
## 3. OpenVSwitch 
OpenVSwitch (ovs) là một dự án về chuyển mạch ảo đa lớp (Multi-layer) Mục đích chính của OpenVSwitch là cung cấp lớp chuyển mạch cho môi trường ảo hóa phần cứng, trong khi hỗ trợ nhiều giao thức và tiêu chuẩn được sử dụng trong hệ thống chuyển mạch thông thường. OpenVSwitch có hỗ trợ nhiều công nghệ ảo hóa dựa trên nền tảng Linux như Xen, Xenserver, KVM, Virtual Box. 
 
OpenVSwich hỗ trợ các tính năng sau 
* VLAN tagging & 802.1q trunking 
* Standard Spanning Tree Protocol (802.1D) 
* LACP 
* Port monitoring (SPAN / RSPAN) 
* Tunneling Protocol 
* QoS 
 
Các thành phần chính của OpenVSwitch 
* ovs-vswitchd: thực hiện chuyển đổi các luồng chuyển mạch. 
* ovs-server: là một lightweight database server, cho phép ovs-switchd thực hiện các truy vấn đến cấu hình 
* ovs-dpctl: công cụ để cấu hình các switch kernel module 
* ovs-vsctl: tiện ích để truy vấn và cập nhật cấu hình ovs-switchd 
* ovs-appctl: tiện ích để gửi command chạy OpenVSwitch 
