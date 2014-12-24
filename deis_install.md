deis paas的环境部署
==================

在做paas之前你得准备好apache离线下载服务器，本地docker的registry私有仓库

1.准备apache服务
===============

2.准备registry离线仓库
=====================

  这边可以直接docker pull registry去官网直接pull一个镜像

  创建一个存储镜像的目录
  
    mkdir -p docker-image

  配置docker的初始化选项,主要配置镜像服务路由地址
  
    --insecure-registry 10.0.0.0/8  --insecure-registry 10.19.95.0/24 --insecure-registry 172.16.0.0/16  
  
    --insecure-registry 10.27.36.0/24
  
  运行registry

    docker run -d -p 22000:22  -p 0.0.0.0:5000:5000  -v /opt/docker-image:/opt/docker-image -e    
  
    DOCKER_REGISTRY_CONFIG=/opt/docker-image/registry-config/config.yml -e STORAGE_PATH=/opt/docker-image registry
 
  标记一个tag
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
   chmod 0600 deis

6.搭建git服务器
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
   
    
    
    
    
  
  
  





