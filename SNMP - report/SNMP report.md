# Tim-hieu-ve-SNMP
## I. SNMP là gì 
SNMP là 1 tập hợp các giao thức giúp quản lý, trao đổi trạng thái của mạng. Hầu như tất cả các thiết bị có thể chạy phần mềm cho phép lấy được thông tin thì SNMP đều có thể quản lý được, cả phần cứng và phần mềm. 

* SNMP version 1: có 3 chuẩn read-only, read-write và trap. Bất kỳ ứng dụng nào hiểu các chuỗi này đều có thể truy cập vào thiết bị quản lý. 

* SNMP version 2: dựa trên các chuỗi "community", vì thế nó còn được gọi là SNMP v2c 

* SNMP version 3: hỗ trợ các loại truyền thông riêng tư và có xác nhận giữa các thực thể. 

Có 3 vấn đề cần quan tâm trong SNMP là Manager, Agent và MIB. 

* Manager là 1 server có chạy các chức năng NMS (Network manager Stations) có khả năng thăm dò và thu thập các cảnh báo từ các Agent trong mạng. 

* Agent là một phần trong các chương trình chạy trên thiết bị mạng cần quản lý. Đó có thể là chương trình chạy độc lập trong Unix hoặc được nhúng vào trong hệ điều hành Cisco IOS trên router. 

* MIB là cơ sở dữ liệu định nghĩa các thông tin NMS có thể truy cập được tới agents. MIB II: có thể đưa ra thông tin tình trạng interfaces (tốc độ, MTU, các octet,...) hoặc đưa ra thông tin hệ thống. 
![Manager Agent MIB topo](http://www.testingtech.com/images/graphics/snpm.png)

### RMON
RMON (Remote Monitoring) cung cấp cho NMS các thông tin dạng packet về các thực thể trong mạng LAN hay WAN 

RMON v2 đặt 1 bộ phận thăm dò của RMON trên các phân đoạn mạng cần theo dõi. 

RMON MIB được thiết kế để các RMONs có thể chạy mà không cần kết nối logic giữa NMS và agents, không cần chờ request từ NMS. Khi NMS muốn có truy vấn thì RMON sẽ trả lời. 

RMON có thể đặt ngưỡng, cảnh báo cho NMS khi vượt ngưỡng (Threshold) 

### SMI (The structure of Management Information) 
SMI cung cấp cách định nghĩa, lưu trữ các đối tượng quản lý, các thuộc tính của chúng. 

SMI có 3 đặc tính: 

* NAME or OID (object identifier) : định nghĩa tên của đổi tượng, được tổ chức theo dạng cây 
* Kiểu và cú pháp 
* Mã hóa 

## II. Cơ chết hoạt động của SNMP 
![Co che SNMP](https://i2.wp.com/i202.photobucket.com/albums/aa89/thuthuattinhoc/4.jpg)

Các lệnh: 
```
Get 
get-next 
get-bulk (SNMP v2 & v3) 
Set 
get-response 
trap (cảnh báo) 
notification  (SNMP v2 & v3) 
inform (SNMP v2 & v3) 
report (SNMP v2 & v3) 
```
## III. Xây dựng mô hình LAB – Cacti 
### 1. Mô hình
* Monitoring server - ubuntu 16.04 (cài đặt và cấu hình cacti): IP 192.168.25.30 
* Agent Linux server - ubuntu 16.04: IP 45.124.95.209, enable SNMP v2, community string: vtp
### 2. Cài đặt Cacti trên monitoring server 
Cài đặt web server (apache, PHP, mysql) 

Chú ý: Cacti không hỗ trợ mysql 5.7 nên ta phải add repo để cài đặt Mysql-server-5.6 

```
$apt-get update 
$apt-get upgrade 
$add-apt-repository 'deb http://archive.ubuntu.com/ubuntu trusty universe' 
$apt-get update 
$apt-get install mysql-server-5.6 
$apt-get install apache2 
$apt-get install php 
$apt-get install libapache2-mod-php 
```
Cài đặt SNMP để giám sát và vẽ đồ thị 
    
    $apt-get install snmp snmpd rrdtool 
Cài đặt Cacti 

    $apt-get install cacti cacti-spine 
Khởi động các dịch vụ 
```
$service httpd start 
$service mysql start 
$service snmpd start 
```
Cấu hình mysql: tạo mysql account và database cho quá trình cài đặt cacti 

    $/usr/bin/mysql_secure_installation 
(Mặc định chưa set pass nên nhấn enter) 
```
Set root password? Y 
Remove anonymous users? Y 
Disallow root login remotely? N 
Remove test database and access to it? Y 
Reload privilege tables now? Y 
```
Tạo account và database cacti sau đó cấp quyền cho user cacti toàn quyền trên database cacti 
``` 
[root@cactiSVR ~]# mysql -uroot -p  
Enter password: 
mysql> CREATE DATABASE cacti;  
mysql> CREATE USER 'cacti'@'localhost' IDENTIFIED BY '1';  
Query OK, 0 rows affected (0.00 sec)  
mysql> GRANT ALL ON cacti.* TO 'cacti'@'localhost';  
Query OK, 0 rows affected (0.00 sec)  
mysql> FLUSH PRIVILEGES;  
Query OK, 0 rows affected (0.00 sec)  
mysql> exit 
```
Download cacti 
```
cd /var/www/html 
Wget http://www.cacti.net/downloads/cacti-0.8.8h.tar.gz 
Tar –xzvf cacti-0.8.8h.tar.gz 
Mv cacti-0.8.8h cacti 
```
Import database mẫu cho 'cacti' database: 

    mysql –u root –p cacti < /var/www/html/cacti/cacti.sql (Nhập password root mysql) 
Chỉnh sửa username và password trong file cấu hình 
```
vi /var/www/html/cacti/include/config.php 
$database_type = "mysql"; 
$database_default = "cacti"; 
$database_hostname = "localhost"; 
$database_username = "cacti"; 
$database_password = "1"; 
$database_port = "3306"; 
$database_ssl = false; 
```
Truy cập địa chỉ 45.124.94.198/cacti để tiến hành cài đặt, chọn next 
![1](https://lh3.googleusercontent.com/XMJDSCFq7QTQoYZqOXA0rT2gzrOk1DXPQ8dShxQPr0E1N8C7bINyB6GkdQMaxwkr4-L2-lVxpnL8_thRzczIEWdCOmXRXybEk8MicmRG3KhndEYbjdkEtOrpZI0R4SywbW0QsOSQMkQez5kI4k-8eR4nty0ac0aiYmB4E6wyiYrVlAmXB-yneMqA_zAG-jex9JUv-7Wy-ADIlstE_HRZa2Poz4h4gx5wdEZF5iJRRhc0vXRybpeNU8DPZoBgIekG6Df9V0s3HTVHWKXhTWuzwEoOPBo2pXCBybp6d9dBcZBlCpj34NVXE7Njfyqjgo2-WuQnK1R3w-tU6DgsYyFH_wdlaVS3AsyL7_I6xo9OBEaJjGlclUIt6Hxk5TR41k32DfUcFTA8RQ0o5K3v5BRGm_SoNHj0vUXk9esGoUDpfkvvAJ1cBBDP1S4OV-pSjHGVYrwiIA43Y1QtQSMAy-Efrrcl8IryvaiodQCA60DgTj6UCzztoOCWvBKxM32C7ASxy7gyg3vzVnEdUEbJrRCsAmMUt5DsmVNiTsGGAC5U7StzsZsB9ZCikX7MVV6th6td4CTJaCvqNbw5mBWqMmssYFamczSvFLzq45NUtl3cYHY=w723-h409-no)

![2](https://lh3.googleusercontent.com/ozNbYP6-o-QLcL4YVrI6MRjpVQ__I52tZx_vvxtnQfOjPKjQg6Kffbw-LKFXgP5N0K0coQqIcom7KL0o9Rp9uxgV5ODkkIp7sK4PujsNYkrbnEuH2t3TlQTkK0sq5zJRV2XCno4TkSN1IiQXTJki94pVYD0xIVGqV6p3o31km52CezpjLgfqw5cQUccNNXTzor22wBYE0LhnId2pPXAZYL0HVp9NTbXv-sprja_89jpNOPBBPB3476PyLOOQtyOxHAdcXgV8NNgu9zy3fLlmcIqTCkAYxpFkyQh0Olz6sT5sJlOESwzbUzCjjAzaFgOOyZ4FuTEEV00NMAVWbcu00XqdvzoWuwn_DBjrc6wK8mmxSmmCOCFND4mP-mVHJlxobLfZ5Bnj6u-AjcFmiUk9uSAOGqZMiuIY0fXnC50Bw_oaiNC9Le-a0VuFdt9Fu29dKs0KD_q0Syoik9WaSpplKuPg6CdQL-JgJ1uR5q4jwgxsnyqwZYm6w9mKZB8-xic_y0iA-bbf_seAvFJDkiPhchkrPGVYPYTqyJRj21mVvMHMZYIwr8hqdh8Qt6uZD5hCPeqzZ7dciEaMN8-HsCae94x_q6CLDOtzJfP3loq-oEU=w758-h328-no)

Chọn finish nếu không thay đổi gì 

![3](https://lh3.googleusercontent.com/g12eppvat1GJkD4qPdfN6ajBX9tv18fTAhFoQehCTES00uPX3D5-GMG9kot8SHOFs3fLGsC_xCdPqSyji82QB5vJD1L2QYg_5xLkKgCiuCifjd6-XY4IPEHmrjL1QlfXVA4P2zf1tmvUSWtqjjAPoshyd8aEROlmSCipDj0YCQZKFnV53aD_EE0WzC_mvqvJdNsGiVkqbN9CtAhuqHEAzJfPaFY2Ai50ISsjLlSn2iOHLxEft71IgEcwBqH0WYeOs5mTr7ZGeJBrGk1OWksOcqfTt0K-IqQWKQg0i0SKAU_DNwNrduSC6PjyNIUOHtrWfVy_KKibg7vWtfEzK-jesxxb-2TPcy2KReyyMvi3ruJmw-hbtzBX5WVSakE_aWWLvX2f6mtBXMMzVcqkFhWhI7giOjYffp_hwcmMEhjOjXmc9niABsvVX53hAR-KAG5pr2N5ZsLfmhl0A1w8vcxVQifWfely8Muzuta7EqHMjIvR1XIFGlMBts2hJ4wwqyGFdydALvPORb2WV8W1yxrlqS-0VG32Nqz-thbjanqwbuPyLBSlWAPw9C6StpuYFuksyK8PAFaZUY0B5qQqO5jDQZQLtYhA3rxh42efU76KeCI=w734-h801-no)


Đăng nhập bằng tài khoản mặc định: admin/admin


![4](https://lh3.googleusercontent.com/zyF6HcakYIX8qoVENYqrOAuQJm-hpdNoTIhFwzoNRVSVoF6pQbgUMYQwPosBggP7ZjFDC8kZ2wdP2STO_MbWUK1vI6Fqy1PjuWd9d795Ce8WkyoBEmMSXTL8T-22TifwyOvlb6mjStLw5cKwmrVWcdqYKftbqo7ejCPbxS8QNT5jeSY2Le_1VryzjH2J2CSA3aNjrv7GqcoykwP6bIj7-xMShht7DY_r3Asp9RrzGEmr4i6t92XDq5SNerRhR_FoD1lQeJT4dbHdE3btOR1kqc1yrhQvm8Lge4s8KhnFMEz5_N0NX4oKqAbnA41nmFZtiYiZC2N5SQwBavshR33F0MjcWP6v2Q6n1Rjo48TFK4JAT8no5Us1SY5OyVVvaMCrWzf4QovoGsd30ZyrB79EefSjUT7FJ2D3MgxSxUKxdo3vKJUbJ8UuWJLsdSa1Z3vmC5uXFOVUfU-R2NiUtaKIsIGOWKRHBHQSsHxg7DatxewX51xwKtqu1pmZgGs8K171FIGIKV43GTxbdYVSHuUFyLAoooE9hG-Nii2SRQ2V4EsmUpvpuHUBqv5Fqu3azpwCDkWuDHYGN398pRPLRL5gh4bmep-1T6YIdBt5fx7D368=w478-h289-no)


Lần đăng nhập đầu tiên yêu cầu đổi mật khẩu


![5](https://lh3.googleusercontent.com/s4OK8Q8okMpeWQ2gP30tMvrZqnCH_CAKePoKwk6rPPWu6OrMh-cTaHzPbyHPcDKuM7sD_7egPXwrBwm7_LgsJT2WRVxy-86vDvSv09vnq8c821Svxj2r-gK3B-4ZcvJdeFS26Aay30P_XCJNiymlk-LaTt5ZBW6YNGPZynxIOxzgYAQBdObISZsB2giuFoR-ayNsleltcrDFll9bVXMg2o36mEQuQblpz-iEPur9VsNQE_pvMGIaFsbRlrcqAr-rQhhXQ8FkilXQ572dZe_o9mh9MN3DR3fr9iVlz3JvrgtMJFPW9RNNBtjpCC83EmaQUcDN6A6S-7PbX6L2X0t3jF-pCYByoMlLFnNtWOTq1a0DxV-4_t2WX8zupCRif0BXsJvLZq2f7bcY5e_pN1bu6z3sXeAw4Lm4gjuqta10MtUuVFChYh1v5-PEk_Q1v3wbBLfEHoDpR5HBB4os3qY0YeA800aMDjxUY8u-bZSnuuFDENToQ3dOO_ewZsO-RedBRe4kg5zVik67flWs-rLDqNqNJgqDUzVR3_Ot8IFSCFzzKl5LafhXdY8aIMD-t41jaaIsSD7VerYkFUKQNdy5Lotrb0SyhSPLx4hboFolxc4=w484-h323-no)


Cấu hình cho cacti cứ 5 phút sẽ thu thập thông tin và lưu trong rrd files, graph tab trên cacti web sẽ hiển thị thông tin dựa trên rrd file trong thư mục /var/www/html/cacti/rra 

Tạo scrip poller.php (/var/www/html/cacti/poller.php) thu thập thông tin sau mỗi 5 phút với account mới trên ubuntur cactiSVR là cacti 
```
[root@cactiSVR:~ ]# useradd cacti 
[root@cactiSVR:~ ]# passwd cacti 
```
![6](https://lh3.googleusercontent.com/t-OKoWDUGt9XLSoXjYd1JoBb2tlOrrdWvMb1VC7Jp1PRzEmEI6vszlu4cjBd71X5Vp0TZaNzBrJO-cv-__7l3hocS2F7Ug_AuGEKtutxkzgNuU9ow-fOlI6v8u81YeDJ9daNbegtxH3Xs9sMX2GH7M2Jdj-frPgXBSxPXD9dqdrWAiUHmQvhaPzY3uU_5X_S_BVGCdzRyBetRrIyHzcf5WMVkMNQ46tgJWXoaGdBmopcoSiMVw3_1LySUSrzm1n78_jC--qwQ-XR9ISagqmVp1mYX-kSISmMaxU1CcBtSMhg1un1FICS400NPwUTOGo7ilHW5cACMElHW127PmXKu9FXnWPrhck__qBFWt-AGMjiBdXN9FQnis_9C2VkyqrOKEAIR1lBHz6FhXLHhCtu6TBnr98vOm2IdIJzo1VsHK4awyDc1xQ4XW8Fl_p8RiCbnZBh-pYzpTjCVk5p3EiKX6WxF2cf3Z3_8NQW53SvtaR-a_HCwU-gBXf9OOwrKLu_Slc2r-ZU7sz1wSJiVZyIkHjwg4YreMkFgm9cfK3DAnT7tmjp9OSmTA8qQcQzXMoKPeAPZd2Klqb6_0L-cfVwVvxTdvAL9KYM91QJEsOZIJw=w363-h125-no)

Tạo crontab cho account cacti sẽ thu thập dữ liệu 5 phút 1 lần và đẩy về /var/www/html/cacti/rra 

    crontab -u cacti –e
![7](https://lh3.googleusercontent.com/eMKTJU2Kmpie2y1UXJg1XLySD8j7cx4oR8ZJPE2RJBf-LvtxKexV0pKLCtpknt89gYEnCV3KvAfHKVtHNOiyKzJmKVvMX6YTNcU0-XN0faH-DZqtf0xpRUrSF3no0h6jEMi53ET-cgr9SdotKc-DAMN6JqnGqDDfyuZ3mzxJ1fC3ItZDrfDDevMsON0H9I25Rh6ouXrFb_vHdRHneefCkXfjaVuWIX-c2wCibvMhJuW-lzfZ8ndopOmyjXac-UL35hXJHZc-sBnSfdd7BXTnBetrmJi0PHVxbatzCD3ev17qvJUYi4czm7LmQwue-aPeu3qfzEWEpu1f6wWu_GV-9Bpwc9g1Xp5dmGETrsDdXX2Ncl_zvnFPyL0Ml_57WEHV1P1UFUy07z35Gyl3ZLQ1fcT3RYEa-Kyr93zyJLt3Yv3v_1snvEm45RG0n8OE2kKCMTIAryYxfI5BhcJIFM6zq4WI7uH_XJPYS6_DvaOPkPxgMr1ca9i6BaIduzj6HpsDVvhOQG24MzhPULSc5q4xUkdT1Nlz8JWU9su_eIBALGf0Oxud8EWmOHiHfS8S4yvb_PYSuIB3huBkKVS9FXcg6OkYc92K144EKqaFUZ-CH0g=w534-h180-no)

Thêm dòng: 

    */5 * * * * /usr/bin/php /var/www/html/cacti/poller.php 
Set quyền cho thư mục rra và log file: 
```
chown cacti /var/www/html/cacti/rra 
chrown cacti /var/www/html/cacti/log/cacti.log 
```
Chỉnh lại timezone trong file php.ini 
    
    root@cactiSVR:~# nano /etc/php/7.0/cli/php.ini  
Thêm dòng: 
    
    date.timezone = "Asia/Ho_Chi_Minh" 
Khởi động lại service apache2 

    root@cactiSVR:~# service apache2 restart 
Truy cập lại địa chỉ http://45.124.94.198/cacti/graph_view.php 

![8](https://lh3.googleusercontent.com/JUYvLAl5WpzstPxexeFzg-a0cEHBX5vDWSE1GgOVVaBJYsqfaGiWN3pov-zblJrZHmglWECHJVqKHilJU7U0rHaTiPSrPS7XUAkLmkKbWHf944FwE5IMZW2NQClwRkyrE8UI1w1_lU591NMrDuBBkQdGUgsfy0yWXbpQWs6WtG1stucq9OVwLdlLs0qlFSfrheKpohbMSqxAnqCoMXFrRZQCxZ9k5kTAcZOyhHVZKV2u9amyj9lmUIC9vdt4C6ZBmHOBYRM9MVtBWLpZ9yx_O7g0ymp8-9Ho6WtTR4p-QfsHxKJGM-xwez8ozp85ntwboej-NE5RyEHvVPds7ByX61OJyD2ho7x3ITko7tV7TlJOann_83oOGTiGcEXsCAAloUIp0HOFW5OrbxRBOG2vAZi17oJgH489UMgQBL41hvHFS5c8CZU_8-hCjJ7MT1fv5JP8u7146M4tDA3LJvzyrooKVZg0sv_O82Xrsri7FEOxcRSOpMLC8EkzAH9WZfDw0vCO-XQc24Obehll2TGMze-OOqACeRiavL5hXMGXwt3tIDtOiSjVeIj7abwGFsmOap9KQTrEanSzya9pWnpVHBAB3kWJeMw0O_9oEZzZ8FI=w950-h535-no)

Như vậy ta đã cài đặt xong cacti server trên ubuntu 16.04. 
### 3. Giám sát Linux server ubuntu (IP 45.124.95.209) 
Cài đặt snmp

    root@agent:~# apt-get install snmpd 
Backup file config snmpd.conf 

    root@agent:~# mv /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf-orgi 
Xóa toàn bộ thông tin trên file snmpd.conf 
    
    root@agent:~# touch ' ' >  /etc/snmp/snmpd.conf 
Thêm thông tin rocommunity string, syslocation and syscontact: 
```
Rocommunity vtp 
Syslocation "VCCloud – Viet Nam" 
Syscontact svtn-tuanphamvan@vccorp.vn 
```
Thay đổi file /etc/defaul/snmpd 

    root@agent:~# nano /etc/defaul/snmpd 
Sửa dòng  

    SNMPDOPTS='-Lsd -Lf /dev/null -u snmp -I -smux -p /var/run/snmpd.pid 127.0.0.1' 
thành: 
    
    SNMPDOPTS='-Lsd -Lf /dev/null -u snmp -I -smux -p /var/run/snmpd.pid -c  
Restart dịch vụ: 

    root@agent:~# /etc/init.d/snmpd restart 
 
Truy cập lại địa chỉ http://45.124.94.198/cacti/ Vào Devices --> add 

Thêm thông tin name, IP, rocommunity string và cac thông tin khác sau đó ấn save 
![9](https://photos.google.com/album/AF1QipNHR517vstZftp9a4xermNuSdQsxWh44uKrQ7_u/photo/AF1QipPJ6Jnhg7GrX3tbDoMBQAABwhW1O-A1eMSymLAn)

Hiện thông báo successfully: 
![10](https://lh3.googleusercontent.com/eWitiDCwq4BZKtbgy3YT5LTgm3awW6IzQ81G-leNLkfaMCOGCp4XXdRpNE8vgHjvekTZAbgE9_gNJ4vGu-yJeO6QRuyyUGpfoY-Am8y-kvFuVvkYg32JtTRRnyjKTkdZKNlqIksigVc31J98RQ_fTgvOAQICF6gyFGBiY4zV_HmwfBePXlSGDLqWGG5S1PV1YMb7UkZ-vZDc8cO5pujSjpVe4B5F7_EFOcBUZtErYSiwwfB3TregFRIaqDxP4YeTZZFW8NP53FiqU72J-tySQBiIeUeG-qZenWLbhZEbXcXheflVmXyjiffXuLETS50LOlwA3Fzp6UOwSn8cO4tHxeNM4ODx5fionw3DPuoPAelPhg16IlgunTlKmPXUmbMW5wx8YRSFQXtIdgA0mRTBdhoXgQHOBJhRRuNKiHRfp6wYbfOXPwZynxnsGYJ3_plNFUMOe6dwIImTB1MWk3hKx5H0RjdtEJsL10rRgmQVwuEiovwOehmVU4TbjjhKbKViTYsZO2htIzCTUFP9H0jVpTfx4VMV5n-D-PTExKwRWy7vDR4WC5D5fdJYuB1DUrMvTE749Br9bwYqdUMl8NKCgOkJyYj1WbO7KbRKzyIaL30=w1760-h990-no)

Như vậy chúng ta đã dùng cacti để giám sát linux server thành công! 
