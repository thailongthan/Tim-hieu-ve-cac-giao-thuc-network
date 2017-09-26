
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
## II. Cài đặt lab 
### Topo
![topo](https://lh3.googleusercontent.com/H42lUfm8x5NoSJHsgPFT0NUD2LGxHnRouMuqhZ2dYgqWUE1CasBxZbuEjqGI9wiK8n_pKYI8hce5xfeyJo62IB25ZHj68485lqniySCvST3wgIuvl0WHgrr2U5496yf-kCWJajwO08S_4i08kl7P78O3ZvYxUef8VjZvIRhUXiDZmci9NdPZITGTNaTWh4FGXivnP5MbqJvajoLzxiTTRMXXf6xjpEUYU6WwrE7FEorFOrSTLznD1lR4p2F6z9UCCR3tgwC2PkL7mJL5qLC9ZHhX2qtGdZNjjP9xgBQh1o_VzXDL8ve9XlpAryGgI3mi1a0Y0lyKNfnFejZ3tQhOmNKPFhpeWu6OSJCWtUw3ey13vKBzM2AgTYlt2FeKoo2_zpaeAhZziwtnbSEWrdf_BE0dG8id90OD7rZiU8fQgPGHuyOxbGBLr6e-Jmbz66MerkJMXLhsvLgLwdDr5fxqrHZusoB5DN-NSiGdR-ifMMtMA8OeldRsHJUofNXg0KfwX7vyt8oBpcGh16GKnfmJJAYqHKDfUcTKOuyGSoqrlIr4jHustd3DRmiKDHi5dvY6O1Grj7RjvWi2QKSUEVwWMVTFgf2kp7yWwa-SYD-K534=w1341-h916-no)
 
### IP Plan 
* nodeA - ubuntu 16.04: 
  * LAN: 192.168.1.100/24 
  * WAN: 172.16.0.2/24 
* nodeB - ubuntu 16.04 
  *  LAN: 192.168.2.100/24 
  *  WAN: 172.16.0.3/24 
* cent1 - centos7: 
  *  IP: 192.168.1.50/24 
  *  Gateway: 192.168.1.100 
* cent2 - centos7: 
  *  IP: 192.168.2.50/24 
  *  GATEWAY: 192.168.2.100 
 
#### Cấu hình vxlan trên nodeA:  
```
$ ip link add vxlan1 type vxlan id 42 dev enp0s8 remote 172.16.0.3 local 172.16.0.2 dstport 4789 
$ ip link set vxlan1 up 
$ ifconfig vxlan1 10.0.0.2/16 
``` 
Định tuyến tĩnh cho gói tin đi từ cent1 sang mạng 192.168.2.0/24 

    $ ip r add 192.168.2.0/24 via 10.0.0.2 

Đặt cho phép forward gói tin: 

    $ sysctl net.ipv4.ip_forward=1 
 
#### Cấu hình vxlan trên nodeB:  
```
$ ip link add vxlan1 type vxlan id 42 dev enp0s8 remote 172.16.0.2 local 172.16.0.3 dstport 4789 
$ ip link set vxlan1 up 
$ ifconfig vxlan1 10.0.0.3/16 
``` 

Định tuyến tĩnh cho gói tin đi từ cent1 sang mạng 192.168.1.0/24 

    $ ip r add 192.168.1.0/24 via 10.0.0.2 

Đặt cho phép forward gói tin: 
    
    $ sysctl net.ipv4.ip_forward=1 

### Kiểm tra: 
Ping từ cent1 đến cent2 

![ping 1 2](https://lh3.googleusercontent.com/xeOr-WKcmcZAbvuzVxEEBuhH3lK3AvVHPJkY-aOxV4RAMAps8aVUH4Q4L628CW9RMl4t17a4I_dIRAIt2WG-Q5b4GfC0pLTue2gJVOhVEhe-tlWOtVbu_bE6IIZrLt3Bc33MxlEJ43JAGetKS_xI7dMTsQJS--w2Ni5sjZqsuu86NkvSa64Dg4Z24X4WoMVydi28ORQWstpkvTscK5ci4ssZWpVk1Zts70HjEio-TtKNarK1MkEkj5HPB3z8IR1rlLXneZI2sO4Sk7MCW_B-LomVXPhiEfxoE4tvM2PwZxSWF238O0hRbSa7its_96NVO3y8f7wzr18f-_0-zr5YXZWN_KyxsQiW9iTchp6BQvwOxXj9maxzh56He2QwfYhh0W8m1OSkldI3ZnUFhUad006wGkmUe8Vemxc0jw30YrEZQ3eLSApdcidozJp5A40L0Ixso2Yb9_uXw0NbALE8WarxdgZNJnvvrD6eO506tHMF73Oy0tlAAn8vqrnThnGpF9OQmQ8JKuvQ8Kl4f6uZJnrNRTyHbkGpEiPRx8adwcAFqvukjzCI-mgc1biBzTtvhbwXwYhjCMOs1IRw5kguDW1PytoKeJYcmW9zQ6q9uVc=w553-h115-no)


Ping từ cent2 đến cent1 

![ping 2 1](https://lh3.googleusercontent.com/xYqszNYPS58yrvriGp5hOURePIaendO6oRjPncfrLekbpuuECzWH6AHPX-HBIETFGwdBAhu71Hz2w6CzN5gZU7berfo4t4NJGvbAlNc6pmo4sPeRGqUBMrLBXg_lD6VCN5kv6pY0brQfi3Z1wC9f2VTlsq5GxLGGZXIU4OvXI-fIajQFkRFaE2ajft_sS3MA4vlUUXljplPXZqWONdSMQX3R_qCVTCx13xZIwasGgEHCUfAfPyxYj08LHBvhrZ5rYUvaAxTYXk6P25ZdT0wSxf6DORH9_IJvjWiLpDeOKdcza2vz_anx_DugihgaMA-vNaV8Xgia1w9z1_pFKaVPzex2c5sMj2-GiZZ6te7aj_Ug_3x8Pr738ivAnupHCpcCKfhz-Iz0-ROS-86XeEtCq2rpqqii4sFLyZfSdYQ68Wm0mbfNGYGz-RCygA5KmFhd7PkDljgGd3XIKVljz8tsQ_Xqu4tWWwkHUMLN6stE4UeSeV1y0GGTNQLDDrmiChpVQpiuu6QV4gmwaKnzEgcOOJGcX1ov6BuKgRzn2shX55swxg1kGaEiMXbIBlEBbHQ8x9WQ7l5i_BZDTp6kiIekqQqFY2YEJgeqDJZo87jAThA=w674-h138-no)


SSH từ cent1 đến cent2 

![ssh 1 2](https://lh3.googleusercontent.com/AQfBzHbiP7d0NJZtDzzhnqWX20RovgCvpbHzpFGPMZhxKvEwr00KXu1au_YJ72UVcQ8f2HYbAdRbH_pt9813QN__Guo-I7IrwahKlUJ8QV5ozN69mJTL3F02dGBja4cMLzGvSM5Aw-ByKbJUEU52QZ1AtvqfyZVTA9AWhDJ5HjRiCc-MA0qSNxz_g5HXNIkUVRLbmfgFejA2CS4vFjxXEa_mWyWt6Erp7wkDVeprORklAO8kt8NOE6Q3RimaKWD0uSV7njpn1V6SlBhwNR9KYHEb9rb-ebTt4TJJYqCNAaA1rbAdjGpaFLnd8C5LQK3rrpOjpDvRx1sxAKlafwMz1LewEXTakWyNoQw8CZ5q9dXeNEt43AlihYy4p4NCihotBxY_ZDjvCQws40Qs7Mdu-ogMwK51kn_DFEw79HtTCBsx01NK51kFovPALJDQ3ttkukQeSEbhpsqk-YbwlSr9dE57tOb7VWnrQQ4vSyVRyC0kraTFI8PbbSkzAKcxt_Hve6TjlA52OXlAu3EEs4rkdlAmHuLknO1wUNnr9Y0UHGa6-qcTiUeyLmNvLjTjhnfTQVgHINw1xnJi3AJtiBnlyE04lC_NS7EpHQdd4KerPIA=w720-h146-no)


SSH từ cent2 đến cent1 

![ssh 2 1](https://lh3.googleusercontent.com/cOrRO6xVvZPKEmhUuJ1yRNZJqQ-tVl01WRh9gfISvq2oIx2OCwQ5RMeD7FSTdEb_jJ1MT85s3IOvdyFxQoQ_p0WwjNClafNIh-kHx4meVmN_5mhfRygJbwNoJQU1F4hebjVkiWIMfKXpvqiXVIHAeBOark8aqMbAeKLL_y9eVUmUuxUUxb38P31SL-FWtUGDuSKi2-v49iUWw0WCmCAP2UUqhY8_mHW4m81uGJtgdtymq0LAJ2Tjqu-wTNeya3phllavYZ6sEwwAvQtDh1-Bj03SFYlKxFy3k8ixubNqfhC198d2ExCNGCjJmHeIL_XqUm98VeqLoYuWmkcsBDw-3n9v1xDsvZpAk5qGLqMQwF-1IWLfHmSdo1szj5mhi8y81PIyQKxKaEtBLuytVxaalyejGtRag9u1Om3cM6fFxzPGOR_RkGP5Nq-woMPQ6-Dkat8HQWuN5AobDMSLL8Ztm9cl8zwNe9T3c0Sn5iI2ROIxqa4rDNeR2-K-9X--5UMi3hMoESb4QMl8N82kt26uMhc-Bj1xz1rQ9RmhG1tswQPwqb_osvSR953CcySa4jAE7aotqT_GpwYsBUy9qJdP1rZaNdV3R0q5YJPBwdc_NP8=w721-h131-no)
