# Install-and-design-Ceph-on-Ubuntu1404

### A. Mô hình LAB
![Alt text](http://i.imgur.com/Cu50qBU.png)
### B. Thiết lập trên node MON
Chuẩn bị môi trường:
- máy chạy Ubuntu1404
- Các máy ảo có 3 card mạng tương ứng:với các dải mạng Local, External và Replicate
#### B.1. Sửa file /etc/hosts
    127.0.0.1       localhost
    10.10.10.154     cephmon
    10.10.10.155     ceph1
    10.10.10.157     ceph2
    10.10.10.158     ceph3
#### B.2. Tải trusted key và add repo, cài đặt
    wget -q -O- 'https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc' | sudo apt-key add -
    echo deb http://ceph.com/debian-hammer/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
	apt-get update
	apt-get install ceph -y
	
#### B.3. Kiểm tra gói cài đặt
```sh
dpkg -l | egrep -i "ceph|rados|rbd"
```
```sh
ceph                                 0.94.5-1trusty                   amd64        distributed storage and file system
ii  ceph-common                          0.94.5-1trusty                   amd64        common utilities to mount and interact with a ceph storage cluster
ii  ceph-fs-common                       0.94.5-1trusty                   amd64        common utilities to mount and interact with a ceph file system
ii  ceph-fuse                            0.94.5-1trusty                   amd64        FUSE-based client for the Ceph distributed file system
ii  ceph-mds                             0.94.5-1trusty                   amd64        metadata server for the ceph distributed file system
ii  libcephfs1                           0.94.5-1trusty                   amd64        Ceph distributed file system client library
ii  librados2                            0.94.5-1trusty                   amd64        RADOS distributed object store client library
ii  libradosstriper1                     0.94.5-1trusty                   amd64        RADOS striping interface
ii  librbd1                              0.94.5-1trusty                   amd64        RADOS block device client library
ii  python-cephfs                        0.94.5-1trusty                   amd64        Python libraries for the Ceph libcephfs library
ii  python-rados                         0.94.5-1trusty                   amd64        Python libraries for the Ceph librados library
ii  python-rbd                           0.94.5-1trusty                   amd64        Python libraries for the Ceph librbd library
```
#### B.4. Tạo FSID cho Ceph Cluster
```sh
uuidgen
df1bc49a-36e8-4669-a5ce-03f2ec635102
```
#### B.5. Tạo thư mục ceph và file ceph.conf 
```sh
mkdir /etc/ceph
vim /etc/ceph/ceph.conf
```
```sh
[global]
public network = 10.10.10.0/24
cluster network = 10.10.20.0/24
fsid = df1bc49a-36e8-4669-a5ce-03f2ec635102

osd pool default min size = 1
osd pool default pg num = 128
osd pool default pgp num = 128
osd journal size = 5000

[mon]
mon host = cephmon
mon addr = 10.10.10.154
mon initial members = cephmon

[mon.cephmon]
host = cephmon
mon addr = 10.10.10.154

[osd]
osd crush update on start = false
```
#### B.6.Tạo keyring và monitor secret key
```sh
ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'
```
#### B.7.Tạo client.admin user và thêm user vào keyring
```sh
ceph-authtool --create-keyring /etc/ceph/ceph.client.admin.keyring --gen-key -n client.admin --set-uid=0 --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow'
```
#### B.8.Thêm client.admin key vào ceph.mon.keyring:
```sh
ceph-authtool /tmp/ceph.mon.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring
```
#### B.9. Sinh monitor map 
```sh
monmaptool --create --add cephmon 10.10.10.154 --fsid df1bc49a-36e8-4669-a5ce-03f2ec635102 /tmp/monmap
```
#### B.10. Tạo thư mục cho monitor như là /path/cluster_name-monitor_node:
```sh
mkdir /var/lib/ceph/mon/ceph-cephmon
```
#### B.11. Tạo thư mục cho monitor như là /path/cluster_name-monitor_node:
```sh
mkdir /var/lib/ceph/mon/ceph-cephmon
```
#### B.12. Tạo thư mục cho monitor deemon
```sh
ceph-mon --mkfs -i cephmon --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring
```
### B. Thiết lập trên các node OSD




### C. Thiết kế
