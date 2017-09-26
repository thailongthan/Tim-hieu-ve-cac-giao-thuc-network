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

## Setup Lab Environment
### Topo
![topo](https://lh3.googleusercontent.com/SfnfvqHvEsmS3z47-Zg7c_heK9kiSaAXqvtOzmsGUzi4LS1vrGBOxycslGbtn6h6flaV_U_NUnKCE-VsN5HsESi-lsbDfOdjq9LkMSt8m5a6kcsbnpKViuQYKKIbGToMyGd6GVgJRgMeibCySOGSSZx47FByiRmhH5FoMQBpPG_y4n8wQhYGxMbqwzRn8wAELff-owCs93xgDt18ngaLKLnUQC3nUZSG1OK2yRQdjxz39jnU2C9LU5G3gSK1Qcx-dNGDK7jDajs51LaLYNJS8czfIzkGisRU_RCe3_drkj_TXWb6NTF5geujdIrBwLsRR2weCQsZlqlLjXtmlNEX07t07R829M1sL4VDrowlGkA7xfWQx313T7rmu2Bsq52VbPTz4xtJY_D4ZXenOsm8cpvrqo6wOG0vfD3KbP19TfC8wfmMekxfjXKlta7HZCIGYxfoAVz89f2n_cYqLhb1KgBG9xnpCzEmGYm3VtSXjzufrcDFDhVg6c3zzbd8g0ZfUOfn8DfguNlp28KAFv2gs7TK6E2sn01dbLC81LYgfZlpTHDgtZctEPRXLo8UNexNmNlj_Y8SayZeSJUJkFYJkrL_cGNmr-aLsMfI51u2eOw=w905-h621-no)
 
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
 
#### Cấu hình gre trên nodeA:  
```
$ ip link add gre1 type gre dev enp0s8 remote 172.16.0.3 local 172.16.0.2 encap-dport 4789 
$ ip link set gre1 up 
$ ifconfig vxlan1 10.0.0.2/16 
``` 
Định tuyến tĩnh cho gói tin đi từ cent1 sang mạng 192.168.2.0/24 

    $ ip r add 192.168.2.0/24 via 10.0.0.2 

Đặt cho phép forward gói tin: 

    $ sysctl net.ipv4.ip_forward=1 
 
#### Cấu hình vxlan trên nodeB:  
```
$ ip link add gre1 type gre dev enp0s8 remote 172.16.0.2 local 172.16.0.3 encap-dport 4789 
$ ip link set gre1 up 
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
