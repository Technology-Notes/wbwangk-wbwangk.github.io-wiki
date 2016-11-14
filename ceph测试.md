准备3台虚拟机（node1/node2/node3)，在各台虚拟机的/etc/hosts文件中分别加入3台虚拟机的主机名和ip：
```
10.10.56.1      node1
10.10.56.2      node2
10.10.56.3      node3
```
每台虚机上安装ssh：
```
apt-get install openssh-server
```
每台虚机上创建ceph专用的用户ceph2：
```
# useradd -d /home/ceph2 -m ceph2
# passwd ceph2
```
确保各个虚机的ceph用户都有sudo权限：
```
echo "ceph ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph2
sudo chmod 0440 /etc/sudoers.d/ceph2
```
允许无密码SSH登录：
```
$ ssh-keygen

Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
```
复制公钥到其他各个节点：
```
ssh-copy-id ceph2@node2
ssh-copy-id ceph2@node3
```

安装apt-get密钥，apt-get利用该密钥校验安装包的hash sum值。
```
wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
```
将ceph加入APT源，hammer是最新版的ceph版本编号：
```
echo deb http://download.ceph.com/debian-hammer/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
```
lsb_release -sc命令会得到linux的版本类型号。增加了apt源后需要执行apt-get update，否则一些依赖包会缺失。
安装ceph：
```
sudo apt-get update && sudo apt-get install ceph ceph-mds
```
以上操作在每个ceph节点上都要进行。本来ceph提供了安装工具ceph-deploy，希望借助SSH实现网络安装。实测ceph-deploy问题多，所以选择手工安装。

apt-get install ceph碰到大量依赖包缺失，后从网上查到aptitude install ceph命令安装。

为虚机增加一块硬盘，以便ceph使用：http://www.linuxidc.com/Linux/2011-02/31868.htm

# 使用vagrant+centos7的测试
Vagrantfile位于：github.com/ksingh7/ceph-cookbook/

dropbox被墙，修改Vagrantfile：
```
BOX='centos7-standard'
#BOX_URL='https://www.dropbox.com/s/hiarmp3cdzjy94o/centos7-standard.box?dl=1'
BOX_URL='https://github.com/CommanderK5/packer-centos-template/releases/download/0.7.2/vagrant-centos-7.2.box'
```

所有虚拟机防火墙开放端口：
```
firewall-cmd --zone=public --add-port=6789/tcp --permanent
firewall-cmd --zone=public --add-port=6800-7100/tcp --permanent
firewall-cmd --reload
firewall-cmd --zone=public --list-all
```
所有虚拟机禁用SELINUX:
```
setenforce 0
sed -i s'/SELINUX.*=.*enforcing/SELINUX=disabled'/g /etc/selinux/config
cat /etc/selinux/config | grep -i =disabled
```
所有虚拟机安装时间同步服务：
```
yum install ntp ntpdate -y
systemctl stop ntpd.service
ntpdate 2.asia.pool.ntp.org
systemctl restart ntpdate.service
systemctl restart ntpd.service
systemctl enable ntpd.service
systemctl enable ntpdate.service
```
### ubuntu trust 14.04.5 修改更新源
/etc/apt/sources.list文件中，把archive.替换为cn.archive.(:%s/archive./cn.archive./g)  
将security.子域名的4行注释掉，替换为：
```
deb http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ trusty-security main restricted universe multiverse
```
然后apt-get update就可以成功了。如果update有错误，会导致ceph-deploy的失败。
### centos 7修改更新源
fedorapeople.org的openstack源地址发生变化，正确的是:
```
https://repos.fedorapeople.org/openstack/EOL/openstack-juno/epel-7/
```
openstack-node1镜像的默认设置的openstack源是(错误的)：
```
https://repos.fedorapeople.org/repos/openstack/openstack-juno/epel-7/
```
所有虚拟机添加ceph的yum源并更新：
```
rpm -Uhv http://ceph.com/rpm-giant/el7/noarch/ceph-release-1-0.el7.noarch.rpm
(yum --enablerepo=base clean metadata)
yum update -y
```

YUM阿里源：
```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache
```
修改了epel的源：http://blog.csdn.net/user_friendly/article/details/8773417

在各个虚机机上创建OSD:
```
ceph-deploy disk zap ceph-node1:sdb ceph-node1:sdc ceph-node1:sdd
ceph-deploy osd create ceph-node1:sdb ceph-node1:sdc ceph-node1:sdd
```
各虚拟机的OSD添加完后设置rbd存储池的pg_num和pgp_num：
```
ceph osd pool set rbd pg_num 256
ceph osd pool set rbd pgp_num 256
```
pg_num设置后ceph需要一定时间调整然后才能设置pgp_num。都设置好需要一定时间，设置好后，ceph -s命令就显示HEALTH_OK状态了。

OSD的删除：http://www.cnblogs.com/zhangzhengyan/p/5839897.html

centos7缺rbd内核模块的解决办法：http://blog.163.com/digoal@126/blog/static/1638770402014112325944867/

bind nameserver部署碰到的问题: 防火墙忘记关闭，导致名称服务器不管用，关闭防火墙：
```
service firewalld stop
chkconfig firewalld off 
```
chkconfig命令用于永久关闭防火墙。

### 为S3创建ceph用户
为s3访问创建RADOS网关用户：
```
radosgw-admin user create --uid=mona --display-name="javacup00" --email=javacup00@163.com -k /etc/ceph/ceph.client.radosgw.keyring --name client.radosgw.gateway
{
    "user_id": "mona",
    "display_name": "javacup00",
    "email": "javacup00@163.com",
    "suspended": 0,
    "max_buckets": 1000,
    "auid": 0,
    "subusers": [],
    "keys": [
        {
            "user": "mona",
            "access_key": "8KF9PZE24O7A43PZ4DM9",
            "secret_key": "JXXQv5Tj6yELsIqhIhFMNr2CzNBcXm2YwrtDw5nr"
        }
    ],
    "swift_keys": [],
    "caps": [],
    "op_mask": "read, write, delete",
    "default_placement": "",
    "placement_tags": [],
    "bucket_quota": {
        "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1
    },
    "user_quota": {
        "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1
    },
    "temp_url_keys": []
}
```

### 创建swift用户
```
radosgw-admin subuser create  --uid=mona --subuser=mona:swfit --access=full -k /etc/ceph/ceph.client.radosgw.keyring --name client.radosgw.gateway
{
    "user_id": "mona",
    "display_name": "javacup00",
    "email": "javacup00@163.com",
    "suspended": 0,
    "max_buckets": 1000,
    "auid": 0,
    "subusers": [
        {
            "id": "mona:swfit",
            "permissions": "full-control"
        }
    ],
    "keys": [
        {
            "user": "mona",
            "access_key": "8KF9PZE24O7A43PZ4DM9",
            "secret_key": "JXXQv5Tj6yELsIqhIhFMNr2CzNBcXm2YwrtDw5nr"
        }
    ],
    "swift_keys": [
        {
            "user": "mona:swfit",
            "secret_key": "ZykqoYycub1hpuHH7NCzIRquNqARsTegyhTDwo9w"
        }
    ],
    "caps": [],
    "op_mask": "read, write, delete",
    "default_placement": "",
    "placement_tags": [],
    "bucket_quota": {
        "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1
    },
    "user_quota": {
        "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1
    },
    "temp_url_keys": []
}
```
为mona:swift子用户创建密钥：
```
radosgw-admin key create --subuser=mona:swift --key-type=swift --gen-secret -k /etc/ceph/ceph.client.radosgw.keyring --name client.radosgw.gateway
{
    "user_id": "mona",
    "display_name": "javacup00",
    "email": "javacup00@163.com",
    "suspended": 0,
    "max_buckets": 1000,
    "auid": 0,
    "subusers": [
        {
            "id": "mona:swfit",
            "permissions": "full-control"
        }
    ],
    "keys": [
        {
            "user": "mona",
            "access_key": "8KF9PZE24O7A43PZ4DM9",
            "secret_key": "JXXQv5Tj6yELsIqhIhFMNr2CzNBcXm2YwrtDw5nr"
        }
    ],
    "swift_keys": [
        {
            "user": "mona:swfit",
            "secret_key": "ZykqoYycub1hpuHH7NCzIRquNqARsTegyhTDwo9w"
        },
        {
            "user": "mona:swift",
            "secret_key": "ZXZOIPUyrBYQgliTVEnJiY9B65Gi4edazMAUJ2QG"
        }
    ],
    "caps": [],
    "op_mask": "read, write, delete",
    "default_placement": "",
    "placement_tags": [],
    "bucket_quota": {
        "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1
    },
    "user_quota": {
        "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1
    },
    "temp_url_keys": []
}
```
列出默认bucket：
```
# swift -A http://192.168.1.106:7480/auth/1.0 -U mona:swfit \
-K ZykqoYycub1hpuHH7NCzIRquNqARsTegyhTDwo9w list
my-new-bucket
```
添加一个新bucket：
```
# swift -A http://192.168.1.106:7480/auth/1.0 -U mona:swfit \
-K ZykqoYycub1hpuHH7NCzIRquNqARsTegyhTDwo9w post second-bucket
```
上传文件：
```
# swift -A http://192.168.1.106:7480/auth/1.0 -U mona:swfit \
-K ZykqoYycub1hpuHH7NCzIRquNqARsTegyhTDwo9w upload second-bucket /etc/hosts
```
列出某容器（桶）下的对象：
```
# swift -A http://192.168.1.106:7480/auth/1.0 -U mona:swfit \
-K ZykqoYycub1hpuHH7NCzIRquNqARsTegyhTDwo9w list second-bucket
```
下载文件：
```
# swift -A http://192.168.1.106:7480/auth/1.0 -U mona:swfit \
-K ZykqoYycub1hpuHH7NCzIRquNqARsTegyhTDwo9w download second-bucket etc/hosts
etc/hosts [auth 0.010s, headers 0.015s, total 0.015s, 0.143 MB/s]
```
### 用python测试对象网关
yum install python-boto
vi s3test.py
下面是s3test.py的内容：
```
import boto
import boto.s3.connection

access_key = '8KF9PZE24O7A43PZ4DM9'
secret_key = 'JXXQv5Tj6yELsIqhIhFMNr2CzNBcXm2YwrtDw5nr'
conn = boto.connect_s3(
        aws_access_key_id = access_key,
        aws_secret_access_key = secret_key,
        host = 'rgw-node1.cephcookbook.com', port = 7480,
        is_secure=False, calling_format = boto.s3.connection.OrdinaryCallingFormat(),
        )

bucket = conn.create_bucket('my-new-bucket')
for bucket in conn.get_all_buckets():
                print "{name}".format(
                name = bucket.name,
                created = bucket.creation_date,
                )
```
执行：
```
python s3test.py
```
