
# VXLAN – Virtual eXtensible Local Aria Network – Mạng LAN ảo mở rộng 

## I. VXLAN - Documentation
 
Giao thức VXLAN là một giao thức tunnetling được thiết kế để giải quyết vấn đề giới hạn VLAN IDs (2 ^ 12 = 4096) trong chuẩn IEE 802.1q. Với VXLAN, độ dài của IDs có thể mở rộng lên 24 bit (16777216) 
 
VXLAN được miêu tả trong IETF RFC 7348, và được thực hiện bởi nhiều nhà cung cấp thiết bị mạng trên thế giới. Giao thức này chạy trên nền UDP sử dụng một cổng đích duy nhất. RFC 7348 miêu tả một thiết bị Linux kernel tunnel, nó cũng tách riêng việc thực thi VXLAN cho OpenvSwitch 
 
Không giống với hầu hết các tunnels khác, một VXLAN là một mạng điểm - đa điểm, không phải điểm điểm. Một thiết bị VXLAN có thể học được địa chỉ IP của các thiết bị đầu cuối khác hoặc tự động làm việc theo cách tương tự như một learning bridge. Hoặc VXLAN cũng có thể được sử dụng như một cổng chuyển tiếp được thiết lập thủ công. 
 
Cuối cùng, VLAN được quyết định làm việc theo cách tương tự như 2 giao thức gần nhất với nó là GRE và VLAN. Việc thiết lập VXLAN yêu cầu iproute version 2 - khớp với phiên bản kernel mà VXLAN lần đầu "lên sóng" 
 
* Create VXLAN device 
```
$ ip link add vxlan0 type vxlan id 42 group 239.1.1.1 dev eth1 dstport 4789 
```
Câu lệnh trên sẽ tạo mới một thiết bị có tên là vxlan0. Thiết bị này sử dụng multicast group 239.1.1.1 qua card mạng eth1 để kiểm soát xử lý lưu lượng: nếu không có bản ghi nào trong bảng forwarding table thì nó sẽ gửi multicast request để tìm nạp dữ liệu vào giống như trong LAN 
 
Cổng đích được đặt một giá trị IANA-assigned là 4789. VXLAN được Linux thực thi trước khi IANA chọn ra một cổng đích tiêu chuẩn và sử dụng gía trị Linux đã chọn làm mặc định để đảm bảo tính tương thích ngược sau này. 
 
* Xóa một vxlan device 
```
$ ip link delete vxlan0 
``` 
* Show vxlan info 
```
 $ ip -d link show vxlan0 
``` 
### Có thể tạo, hủy và hiển thị bảng vxlan forwarding table sử dụng một bridge command mới 

* Thêm 1 bản ghi vào forwarding table
```
       $ bridge fdb add to 00:17:42:8a:b4:05 dst 192.19.0.2 dev vxlan0   
 ```
* Xóa một bản ghi trong forwarding table 
```
       $bridge fdb delete 00:17:42:8a:b4:05 dev vxlan0   
 ```
* Hiển thị bảng forwarding table 
```
       $ bridge fdb show dev vxlan0 
 
```
