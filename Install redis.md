## Step 1 download file install All node
```
wget https://github.com/redis/redis/archive/7.0.5.tar.gz
```

## Step 2 Tar file and make redis All node
```
tar xvf 7.0.5.tar.gz
ls -ltr
cd redis-7.0.5/src
yum install make
yum install gcc
yum install -y oracle-epel-release-el7
yum install -y python36
make 
```
```
mkdir -p /var/redis-log
```

## Step 3 Install package for redis all node
```
yum-config-manager –add-repo= http://yum.oracle.com/repo/OracleLinux/OL7/SoftwareCollections/x86_64
yum install oraclelinux-release-el7
/usr/bin/ol_yum_configure.sh
yum-config-manager --enable software_collections
yum-config-manager --enable ol7_latest ol7_optional_latest
yum-config-manager --enable ol6_latest
yum install scl-utils
yum -y install devtoolset-9
```

## Step 4 Test redis all node
```
scl enable devtoolset-9 bash
cd /root/redis-7.0.5/src/
ls -ltr test
make test
```

## Step 5 Create file config master and slave all node
```
mkdir -p /etc/redis
cp /root/redis-7.0.5/redis.conf /etc/redis-master/redis.conf
cp /root/redis-7.0.5/redis.conf /etc/redis-slave/redis.conf
```

## Step 6 Create service for redis-master and redis-slave all node
>redis-master
```
cat<<EOF>/etc/systemd/system/redis-master.service
[Unit]
 Description=Redis In-Memory Data Store
 After=network.target
[Service]
 User=root
 Group=root
 ExecStart=/root/redis-7.0.5/src/redis-server /etc/redis-master/redis.conf
 ExecStop=/root/redis-7.0.5/src/redis-cli shutdown
 Restart=always
[Install]
 WantedBy=multi-user.target
EOF
```
>redis-slave
```
cat<<EOF>/etc/systemd/system/redis-slave.service
[Unit]
 Description=Redis In-Memory Data Store
 After=network.target
[Service]
 User=root
 Group=root
 ExecStart=/root/redis-7.0.5/src/redis-server /etc/redis-slave/redis.conf
 ExecStop=/root/redis-7.0.5/src/redis-cli shutdown
 Restart=always
[Install]
 WantedBy=multi-user.target
EOF
```

## Step 7 Edit file config redis-master and redis-slave all node
>redis-master
```
vi /etc/redis-master/redis.conf
```
```
bind $IPNODE
port 6479
protected-mode no
cluster-enabled yes
cluster-node-timeout 15000
logfile "/var/log/redis-log/redis-master.log"
dbfilename dump-NU850RED01-redis-master.rdb
cluster-config-file redis_master_NU850RED01.conf
```
>redis-slave
```
vi /etc/redis-slave/redis.conf
```
```
bind $IPNODE
port 6480
protected-mode no
cluster-enabled yes
cluster-node-timeout 15000
logfile "/var/log/redis-log/redis-slave.log"
dbfilename dump-NU850RED01-redis-slave.rdb
cluster-config-file redis_slave_NU850RED01.conf
```
## Step 8 Start and enable redis-master, redis-slave all node
```
systemctl start redis-master && systemctl start redis-slave
systemctl enable redis-master && systemctl enable redis-slave
systemctl status redis-master && systemctl status redis-slave
```

## Step 9 Create cluster redis
```
redis-cli -h 172.20.96.10 -p 6479 --cluster create 172.20.96.10:6479 172.20.96.11:6479 172.20.96.12:6479 172.20.96.10:6480 172.20.96.11:6480 172.20.96.12:6480 --cluster-replicas 1
```

## Step 10 Verify cluster redis
```
redis-cli -h 172.20.96.10 -p 6479 cluster nodes
```
