deis paas的环境部署
==================

在做paas之前你得准备好apache离线下载服务器，本地docker的registry私有仓库,查看etcd是否配置成功
          fleetctl list-machines 
          如果看到
          10.27.36.152
          10.27.36.154
          10.27.36.158
          诸如上述的coreos集群ip地址表示coreos集群没有问题

1.准备apache服务
===============

2.准备registry离线仓库
=====================

  这边可以直接docker pull registry去官网直接pull一个镜像

  1)创建一个存储镜像的目录
  
       mkdir -p docker-image

  2)配置docker的初始化选项,主要配置镜像服务路由地址
  
       --insecure-registry 10.0.0.0/8  --insecure-registry 10.19.95.0/24 --insecure-registry 172.16.0.0/16  
  
       --insecure-registry 10.27.36.0/24
  
  3)运行registry

       docker run -d -p 22000:22  -p 0.0.0.0:5000:5000  -v /opt/docker-image:/opt/docker-image -e    
  
       DOCKER_REGISTRY_CONFIG=/opt/docker-image/registry-config/config.yml -e STORAGE_PATH=/opt/docker-image registry
 
  4)标记一个tag
       docker tag imageID 10.19.95.125/deis/builder:v1.0.2
  
       docker push 10.19.95.125/deis/builder:v1.0.2上传镜像到私有仓库服务器
  
       docker pull 10.19.95.125/deis/builder:v1.0.2下载镜像

3.官网下载deis组件到私有仓库
===============================

    deis/logger      1 docker pull deis/logger:v1.0.2

    deis/logspout   1  docker pull deis/logspout:v1.0.2

    deis/controller  1       docker pull deis/controller:v1.0.2

    deis/builder   docker pull deis/builder:v1.0.2  需要自己编译

    deis/data   1 docker pull deis/data

    deis/base  1 docker pull deis/base

    deis/router 1 docker pull deis/router:v1.0.2

    deis/publisher  1 docker pull deis/publisher:v1.0.2

    deis/cache  1 docker pull deis/cache:v1.0.2

    deis/registry  1 docker pull deis/registry:v1.0.2

    deis/database   1 docker pull deis/database:v1.0.2

    deis/store-metadata 1 docker pull deis/store-metadata:v1.0.2

    deis/store-gateway  1  docker pull deis/store-gateway:v1.0.2  

    deis/store-monitor  1 docker pull deis/store-monitor:v1.0.2  

    deis/store-daemon 1 docker pull deis/store-daemon:v1.0.2  
    
     将下载好的镜像全部打上标记，push到本地仓库
  
4. 配置dns信息，参照dns文档
===========================
     配置完成后用nslookup查看是否配置正确

5. 上传deis,deis.pub到coreos .ssh目录
==========================================
     chmod 0600 deis

6.搭建git服务器
=========================================
   这边可以直接使用gitblit作为git服务
    
    1.下载gitblit.war包并改名ROOT.war包
    
    2.安装tomcat7服务器，将ROOT.war发布，只能tomcat7其他无法编译
  
    3.访问http://ip:8080即可访问git服务器
    
    4.配置des的公钥到gitblit服务器
    
    5.将github上面的heckuo的文件全部git clone下来push到自己的git服务器
    
    git clone https://github.com/ddollar/heroku-buildpack-multi.git         
    git clone https://github.com/heroku/heroku-buildpack-ruby.git           
    git clone https://github.com/heroku/heroku-buildpack-nodejs.git         
    git clone https://github.com/heroku/heroku-buildpack-java.git           
    git clone https://github.com/heroku/heroku-buildpack-gradle.git         
    git clone https://github.com/heroku/heroku-buildpack-grails.git         
    git clone https://github.com/heroku/heroku-buildpack-play.git           
    git clone https://github.com/heroku/heroku-buildpack-python.git         
    git clone https://github.com/heroku/heroku-buildpack-php.git            
    git clone https://github.com/heroku/heroku-buildpack-clojure.git        
    git clone https://github.com/heroku/heroku-buildpack-scala.git          
    git clone https://github.com/heroku/heroku-buildpack-go.git   
    
在gitblit分别将上述工程创建，找某台虚拟机进行并clone到本地，然后将github工程删除掉.git文件以后全部copy到对应的本地git工程，并push到服务器如下
    
    git clone https://github.com/heroku/heroku-buildpack-java.git  
    cd heroku-buildpack-java
    rm .
    
    git clone http://10.27.36.105:8080/r/heroku-buildpack-java.git
    mv * ../heroku-buildpack-java
    git add .
    git commit -m "update codes"
    git push origin master
    
7.修改编译buildpack源代码
===============================
1). 官网镜像到私有仓库
     docker pull progrium/cedarish

     导出镜像文件为progrium_cedarish.tar放入deis-1.0.2/builder目录下面

2).修改Dockerfile
   # HACK: import progrium/cedarish as a tarball
   # see https://github.com/deis/deis/issues/1027
   # RUN curl -#SL -o /progrium_cedarish.tar \
   #    https://s3-us-west-2.amazonaws.com/opdemand/progrium_cedarish_2014_10_01.tar
   
    RUN curl -sSL -o /usr/local/bin/etcdctl https://s3-us-west-2.amazonaws.com/opdemand/etcdctl-v0.4.6 \
    && chmod +x /usr/local/bin/etcdctl ==>> 
   
     RUN curl -sSL -o /usr/local/bin/etcdctl http://192.168.1.103/opdemand/etcdctl-v0.4.6 \
     && chmod +x /usr/local/bin/etcdctl
    
3).修改slugbuilder,slugrunner
     FROM progrium/cedarish ===> FROM 10.19.95.125:5000/progrium/cedarish:latest
  
4). 修改boot文件,根据需要修改镜像设置
    spawn a docker daemon to run builds
    docker -d --storage-driver=$STORAGE_DRIVER --bip=172.19.42.1/16 --insecure-registry 10.0.0.0/8 --insecure-registry
    172.16.0.0/12 --insecure-registry 10.27.36.0/24 --insecure-registry 10.19.95.0/24 & 
    
    /progrium_cedarish.tar ==> /app/progrium_cedarish.tar

5). make build执行编译，如果没有make命令直接将编译好的make copy到本地编译即可，并push本地仓库

8.同步时间服务器
====================
    sudo systemctl stop ntpd; sudo ntpdate -s 192.168.118.201;sudo systemctl start ntpd
    ntpq -p如果出现同步信息表示正确

9.部署deis
===================
    deisctl install platform
    如果无法安装，需要先deisctl refresh-units下载service，需要网络
    
    deisctl start platform 
    如果启动成功则会如下：
    deis-builder.service		87a01b13.../10.27.36.154	loaded	active	running
    deis-cache.service		d6d6e4cc.../10.27.36.158	loaded	active	running
    deis-controller.service		87a01b13.../10.27.36.154	loaded	active	running
    deis-database.service		d04f30aa.../10.27.36.152	loaded	active	running
    deis-logger.service		d04f30aa.../10.27.36.152	loaded	active	running
    deis-logspout.service		87a01b13.../10.27.36.154	loaded	active	running
    deis-logspout.service		d04f30aa.../10.27.36.152	loaded	active	running
    deis-logspout.service		d6d6e4cc.../10.27.36.158	loaded	active	running
    deis-publisher.service		87a01b13.../10.27.36.154	loaded	active	running
    deis-publisher.service		d04f30aa.../10.27.36.152	loaded	active	running
    deis-publisher.service		d6d6e4cc.../10.27.36.158	loaded	active	running
    deis-registry.service		d6d6e4cc.../10.27.36.158	loaded	active	running
    deis-router@1.service		d04f30aa.../10.27.36.152	loaded	active	running
    deis-router@2.service		d6d6e4cc.../10.27.36.158	loaded	active	running
    deis-router@3.service		87a01b13.../10.27.36.154	loaded	active	running
    deis-store-daemon.service	87a01b13.../10.27.36.154	loaded	active	running
    deis-store-daemon.service	d04f30aa.../10.27.36.152	loaded	active	running
    deis-store-daemon.service	d6d6e4cc.../10.27.36.158	loaded	active	running
    deis-store-gateway.service	87a01b13.../10.27.36.154	loaded	active	running
    deis-store-metadata.service	87a01b13.../10.27.36.154	loaded	active	running
    deis-store-metadata.service	d04f30aa.../10.27.36.152	loaded	active	running
    deis-store-metadata.service	d6d6e4cc.../10.27.36.158	loaded	active	running
    deis-store-monitor.service	87a01b13.../10.27.36.154	loaded	active	running
    deis-store-monitor.service	d04f30aa.../10.27.36.152	loaded	active	running
    deis-store-monitor.service	d6d6e4cc.../10.27.36.158	loaded	active	running
    deis-store-volume.service	87a01b13.../10.27.36.154	loaded	active	running
    deis-store-volume.service	d04f30aa.../10.27.36.152	loaded	active	running
    deis-store-volume.service	d6d6e4cc.../10.27.36.158	loaded	active	running

9. 常用命令举例
       deisctl list列出组件
       deisctl restart builder重启builder组件
       deisctl start builder启动builder组件
      
       journalctl -u deis-store-monitor -f
       cat /var/lib/log/deis/...
       fleetctl list-units此处科员查看service情况和对应的
      
       查看ceph信息
       nse deis-store-monitor
       ceph -s


  
    
    

   
   
    

   
    
    
    
    
  
  
  





