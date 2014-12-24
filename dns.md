采用bind9作为dns服务器
====
vi /etc/bind/named.conf.local 

    zone "deis.suning" {
       type master;
        file "/etc/bind/db.deis.suning";
    };

deis deploy

正向解析：

vi /etc/bind/db.deis.suning

    $ORIGIN .
    
    $TTL 7D
    
    deis.suning    IN SOA  ns.deis.suning. admin.deis.suning. (
    
            2014112701      ; Serial
            
            8H        ; Refresh
            
            2H        ; Retry
            
            4W        ; Expire
            
            1D)       ; Minimum TTL
    
            NS  ns.deis.suning.
    
    $ORIGIN deis.suning.
    
    deis-01           A   10.27.36.152
    
    deis-02           A   10.27.36.154
    
    deis-03           A   10.27.36.158
    
反向解析：
    
    ;
        
    ; BIND reverse data file for dev domains
    
    ;
    
    $TTL    604800
    
    @       IN      SOA     dev. root.dev. (
    
                                  1         ; Serial
                                  
                             604800         ; Refresh
                             
                              86400         ; Retry
                              
                            2419200         ; Expire
                            
                             604800 )       ; Negative Cache TTL
                             
    ;
    
    @        IN      NS      cf.suning.
    
    3        IN      PTR     cf.suning.
    
    @        IN      NS      deis.suning.
    
    3        IN      PTR     deis.suning.
    
    
    
    
