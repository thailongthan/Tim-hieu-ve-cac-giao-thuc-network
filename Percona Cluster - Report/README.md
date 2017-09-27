# Cài đặt Percona cluster 
Sử dụng 3 DB server chạy ubuntu 16.04 với IP plan (Card internal, vẫn cần có 1 card Bridge để cài đặt các gói package từ mạng ngoài và từ máy thật ssh đến config cho dễ) như sau: 
* Máy 1:  
  * Hostname: cluster1 
  * IP: 192.168.70.61 
* Máy 2:  
  * Hostname: cluster2 
  * IP: 192.168.70.62 
* Máy 3:  
  * Hostname: cluster3 
  * IP: 192.168.70.63 
## Cài đặt Percona XtraDB Cluster 57 lên cả 3 máy: 
Xóa cài đặt appamor 

    $ apt-get remove apparmor 

Thêm và cài đặt repo của Percona: 

    wget https://repo.percona.com/apt/percona-release_0.1-4.$(lsb_release -sc)_all.deb 
    dpkg -i percona-release_0.1-4.$(lsb_release -sc)_all.deb 
    apt-get update 

Cài đặt Percona XtraDB Cluster 

    apt-get install percona-xtradb-cluster-57 

Trong quá trình cài đặt, hệ thống sẽ yêu cầu đăng ký mật khẩu root user cho mysql 

Tạm dừng dịch vụ mysql để config 

    service mysql stop 

## Thiết lập cho cluster 1: 

 * Backup file config 
```
     cp /etc/mysql/my.cnf /etc/mysql/my.cnf.orgi 
``` 
 * Xóa nội dung file config 
```
      > /etc/mysql/my.cnf 
```
 * Sửa nội dung file config theo mẫu sau: 
```
[mysqld]  
datadir=/var/lib/mysql  
user=mysql  
# Đường dẫn tớiPathera library  
wsrep_provider=/usr/lib/libgalera_smm.so  
# URL kết nối giữa các node 
wsep_clustercluster_address=gcomm://192.168.70.61,192.168.70.62,192.168.70.63  
  
binlog_format=ROW  
default_storage_engine=InnoDB  
innodb_autoinc_lock_mode=2  
# Node #1 address  
wsrep_node_address=192.168.70.61  
# SST method  
wsrep_sst_method=xtrabackup-v2  
# Cluster name  
wsrep_cluster_name=my_ubuntu_cluster  
# Authentication for SST method  
wsrep_sst_auth="sstuser:s3cretPass" 
```
Khởi động dịch vụ trên node1: 

    root@cluster1:~# /etc/init.d/mysql bootstrap-pxc 

Vào mysql kiểm tra trạng thái của cluster, chú ý các dòng sau 
```
mysql> show status like 'wsrep%'; 

| wsrep_local_state                | 4                                                        | 
| wsrep_local_state_comment        | Synced                                                   | 
 
| wsrep_cluster_size               | 1                                                        | 
| wsrep_cluster_status             | Primary                                                  | 
| wsrep_connected                  | ON                                                       | 
 
| wsrep_ready                      | ON                                                       | 
``` 
 Để có thể thực hiện giao thức sst (state snapshot transfer) sử dụng XtraBackup, ta cần tạo mới 1 user trong mysql ở cluster 1 với thiết lập giống như dòng cuối trong file config my.cnf như sau 
``` 
mysql@cluster1> CREATE USER 'sstuser'@'localhost' IDENTIFIED BY 's3cretPass';  
mysql@cluster1> GRANT PROCESS, RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'sstuser'@'localhost';  
mysql@cluster1> FLUSH PRIVILEGES; 
```
root account của mysql có thể thực hiện được SST, nhưng để bảo mật hơn thì người ta sẽ dùng tài khoản khác (đỡ bị nhìn mật khẩu root trong file my.cnf) 
 
## Thiết lập cho cluster 2: 
Tương tự như cluster 1, ta config file my.cnf như sau: 
```
[mysqld] 

datadir=/var/lib/mysql 
user=mysql 
 
# Đường dẫn tớiPathera library  
wsrep_provider=/usr/lib/libgalera_smm.so 
 
# URL kết nối giữa các node 
wsep_clustercluster_address=gcomm://192.168.70.61,192.168.70.62,192.168.70.63 
 
binlog_format=ROW 
default_storage_engine=InnoDB 
innodb_autoinc_lock_mode=2 
 
# Node 2 address 
wsrep_node_address=192.168.70.62 
 
# SST method 
wsrep_sst_method=xtrabackup-v2 
 
# Cluster name 
wsrep_cluster_name=my_ubuntu_cluster 
 
# Authentication for SST method 
wsrep_sst_auth="sstuser:s3cretPass" 
``` 
Và khởi động mysql trên cluster 2: 

    root@cluster2:~# /etc/init.d/mysql bootstrap-pxc 

Sau khi khởi động xong, cluster 2 sẽ tự động nhận SST, trạng thái cluster có thể được kiểm tra ở bất cứ node nào. Ta vào mysql trên cluster 2 và kiểm tra 

    mysql> show status like 'wsrep%'; 
    
Chú ý những dòng sau 
```
| wsrep_local_state                | 4                                                        | 
| wsrep_local_state_comment        | Synced                                                   | 
 
| wsrep_cluster_size               | 2                                                        | 
| wsrep_cluster_status             | Primary                                                  | 
| wsrep_connected                  | ON                                                       | 
 
| wsrep_ready                      | ON                                                       | 
```
## Thiết lập cho cluster 3: 
Thiết lập file config my.cnf tương tự như cluster 1 và cluster 2, chỉ thay phần IP của cluster 3 vào. 
```
# Node 3 address 
wsrep_node_address=192.168.70.63 
```
Và khởi động mysql trên cluster 3: 

    root@cluster3:~# /etc/init.d/mysql bootstrap-pxc 

Sau khi khởi động xong, cluster3 sẽ tự động nhận SST, trạng thái cluster có thể được kiểm tra ở cả 3 node. Ta vào mysql trên cluster3 và kiểm tra 

    mysql> show status like 'wsrep%'; 

Chú ý những dòng sau 
```
| wsrep_local_state                | 4                                                        | 
| wsrep_local_state_comment        | Synced                                                   | 
 
| wsrep_cluster_size               | 3                                                       | 
| wsrep_cluster_status             | Primary                                                  | 
| wsrep_connected                  | ON                                                       | 
 
| wsrep_ready                      | ON                                                       | 
``` 
![anh1](https://lh3.googleusercontent.com/AqiJVb4gt1xCJGWvEQfsMAVbaj4rtQ0kG6S_3pD6QQM7NY8GIISXuOkeZso2eiBN_q6z1EN1SlixYxmfBgVanOSCH7VzC7iXb0cN4Q0z5ANokWz3WXwZ8_EKtC1icLE6ZV62snJn86Mbu5WoOJNIqbNasAfTOG7uK1BLYeoMAfaGS7XXzbvwRGbpHUmfRIT544--FlPKqcRLVG3WPvC0ErdEjg7KxLvEymQ46EkEqXdZcGvLyCi3kOO85nKbyAfbga4IBVFXF7cCWTH-LtAhb1L04-e9U110T1gFLY9l1RuWkARhAbwvOzVLaDOi9dfnxVQaz_1HBtCT9WZuXJFoNaOYFcldLZC0IybRbNVfxBrox4ChlLoxNCVXZ80-_yi9GkP1qPVWFPw2geZ3tjbQoKSE69LnE11lcweU5S0Mmzj23NY99nY_scb71KiKGtxdHIogm8VduEzEHZY4iXPZZLwwLbZ3bBn_jHwQUmPTuUx8DSQ9743U11kGuKZMOlgSkckHFg2fd0L1c2ZLSahqB_yAOa-cn4ZkLgmksRsmYS_IENLAQCkbxSfT95wJZzbmDYRqN9lTGDcEPH2H-42xLhjvejU5rDf3H7DKaRY0p-4=w882-h569-no)

## Kiểm tra khả năng sao chép dữ liệu giữa các node 
Tạo 1 database mới trên cluster2 
```
mysql> CREATE DATABASE vccloud; 
Query OK, 1 row affected (0.09 sec) 
```
Tạo 1 bảng mới trên cluster 3 
```
mysql> use vccloud; 
Database changed 
mysql> CREATE TABLE internship (node_id INT PRIMARY KEY, node_name VARCHAR(30)); 
Query OK, 0 rows affected (0.08 sec) 
``` 
![anh2](https://lh3.googleusercontent.com/gk3Wy-KNteE-clv2GJ6nVzeQ4hDYx47fsORcYpdI-uAWgrGXpmlGx7SmR-hHyZFOOnRFm8ucBmLItHfhlkF47Q7_-hIyCPMQ3_hpr7fsgAmbcC_MqF4gBl3qbXz5_T47xWhXSmE663i21oUzzmEqdMDL-oWYTU5wBZzyTD93Jn0i4pGQB10MlifeFf-8yes2dwvwZXEaoOXHUwDM7_VMHTOFYMXLigIW0GG1DAgz4TPL2kwW1RK0vl_DV7aopZGuk6iQ1Qlcm-1Jk3A85xO8bDxELDfb_77YMKcRqI4hkPE1xqD4Jo4UFbjZgsGS23ODigyTpw_BrJhXz5AshtuUs91oTI0ucyNQs7oZrbOnV5bO3g4IfYfhm2MzQby7PIM_tQ5XysV07pZo0enKcVDjN3eA16bHaaYjQdCXXYfUdVLOv4vqbPnVt0UoOvhpW1P9pVHR9vzyi4aA4tf9IRy55LuKVHmbMq94hk9HG0IVolfcN14-QrorNWxhRLPYz-UQs5QZngKC_R5PUtN7SpiFNmLnY0iuGucht151nLaV-YJLVk1Lt9CVQ1kdrXL1hEhSMDEcCv8r5RVy3JSP9q0arYnfsdGFwoT_0AoFZ4UxtCw=w760-h80-no)

Thêm một giá trị mới vào bảng intership trên cluster 1 
```
mysql> INSERT INTO vccloud.internship VALUES (1, 'Van Tuan Pham'); 
Query OK, 1 row affected (0.04 sec) 
```
Kiểm tra lại bảng internship trên cluster 2 
```
mysql> select * from vccloud.internship; 
+---------+---------------+ 
| node_id | node_name     | 
+---------+---------------+ 
|       1 | Van Tuan Pham | 
+---------+---------------+ 
1 row in set (0.00 sec) 
``` 
![anh3](https://lh3.googleusercontent.com/-RZdeWjzy1GuHdYvSpgqIFCyn6pjnuUZKBe2YzzsYaRhAi1XhmqoL_eBZBUAsrSNBgMu8K53uiOtkqN0bm3em3g3AKYmYZwT-AN0o9TGLBQEBJMiZT7WbNwBjbaTy_UqMGQHo6uruB4TOftuqbr-2cPN0ujAlG9WKApwkAqm2fn1NB6wAZPWHdMLJV09O2ZVXi_pQNiTDDPnASLbysrfw2_FvIvkhuBCTJO6jJ5wY59AU8EV6wa-x5sOIbSrJeytelkrEB-Yb11bTwahJe5lg8VDnwaagjU5qVk6fgWNaS_vkDSnObrpXg_s8tTIl6GkTAWH1ubnnSPaooApNJyHkNl5q3PFErk5njSL_7hQa4Gjy8GCwc7CEaeMYbyRYD1ankGKIwIP_mcdTmsbEdUvgcjZwPVzdf3BaFOxccK7nIlWeDBbrjIf5kK0P4qydYgZeUZWpdeiFRKlI6BD5fRsfWv3_7pmmW_eDQIfXG92BZ4YRJQkTcMDcFCpyAHOXj_y_WvqcgzMMTzKKgl44PqNhM9jBt92-N04veBKHVVeDUWu134vLXeavjeNNW4pveCK3ZUt2nDt0gW8hPQ71hn2x2CRwmQsZmDftOiZeprszs4=w398-h137-no)
Như vậy ta đã cài đặt xong Percona Cluster gồm 3 node trên ubuntu và tất cả các node đều đã đồng bộ với nhau cũng như làm việc tốt. 
