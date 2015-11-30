# Install-and-design-Ceph-on-Ubuntu1404

### A. Mô hình LAB

### B. Cài đặt Ceph
Chuẩn bị môi trường:
- 4 máy ảo chạy Ubuntu1404
- Disable iptables
- Các máy ảo có 3 card mạng tương ứng:với các dải mạng Local, External và Replicate

#### B.1. Truy cập bằng tài khoản root vào máy các máy chủ và tải các gói, script chuẩn bị cho quá trình cài đặt
```sh
apt-get update
```

#### B.2. Sửa file /etc/hosts
    127.0.0.1       localhost
    10.10.10.154     cephmon
    10.10.10.155     ceph1
    10.10.10.157     ceph2
    10.10.10.158     ceph3

##### B.3. Tạo cặp khóa SSH trên node MON
    ssh-keygen
    Enter file in which to save the key (/home/user_name/.ssh/id_rsa): [press enter]
    Enter passphrase (empty for no passphrase): [press enter]
    Enter same passphrase again: [press enter]
	
#### B.4. Chuyển khóa public sang các node CEPH khác
    ssh-copy-id root@ceph2
    ssh-copy-id root@ceph3
    ssh-copy-id root@ceph1
	
#### B.5. Tải trusted key và add repo, cài đặt trên các node	
    wget -q -O- 'https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc' | sudo apt-key add -
    echo deb http://ceph.com/debian-hammer/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
	apt-get update
	apt-get install ceph -y
	
### B.6. Kiểm tra gói cài đặt
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

### C. Thiết kế
