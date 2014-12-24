deis使用的操作系统为coreos系统，并且需要集群环境这里我们谈一下coreos的集群安装

这里我们使用coreos的信息：
  Containers: 12
  Images: 82
  Storage Driver: btrfs
  Execution Driver: native-0.2
  Kernel Version: 3.17.2+
  Operating System: CoreOS 509.1.0

1.首先我们需要配置coreos网络如下：
==============================================================================
  vi /etc/systemd/network/static.network

  [Match]
  Name=ens32(网卡的名称可以用ifconfig查看)

  [Network]

  Address=10.27.36.152/24

  Gateway=10.27.36.254

  DNS=10.27.36.202
 
  systemctl restart systemd-networkd重启网络即可


2.搭建自己的apache服务器 10.27.36.102
==============================================================================

  将resource目录的文件都放如/var/www/html目录下面


3.下载配置文件到coreos本地
==============================================================================
  wget 10.27.36.102/deis_config.yaml

  wget 10.27.36.102/coreos-install

deis_config.yaml修改：

  根据你需要设置的ip进行替换即可，命令%s/ip1/ip2/g
  如果需要配置peers也就是从etcd配置需加入：

  coreos:

    etcd:
  
       peers: ip:7001
     
      ....
 
 根据需要你需要修改你的镜像pull，push的路由地址
  Environment="DOCKER_OPTS=--insecure-registry 10.19.95.0/24 --insecure-registry 172.16.0.0/24  --insecure-registry 
  10.27.36.0/24"

coreos-install修改:

  需要修改BASE_URL="http://10.27.36.102"（apache的服务器地址）


4.安装coreos
==============================================================================
./coreos-install -d /dev/sda -C stable -c ./deis_config.yaml 


    
    
