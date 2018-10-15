yum install keepalived haproxy psmisc -y

vi /etc/keepalived/keepalived.conf

``` sh
vrrp_script chk_haproxy {           
        script "killall -0 haproxy"     
        interval 2                      
        weight 2                        
}

vrrp_instance VI_1 {
        interface ens34
        state MASTER
        virtual_router_id 51
        priority 101                    
        virtual_ipaddress {
            10.10.10.180       
        }
        track_script {
            chk_haproxy
        }
}
```
``` sh
vrrp_script chk_haproxy {       
        script "killall -0 haproxy"     
        interval 2                      
        weight 2                        
}

vrrp_instance VI_1 {
        interface ens34
        state BACKUP
        virtual_router_id 51
        priority 100                    
        virtual_ipaddress {
            10.10.10.180             
        }
        track_script {
            chk_haproxy
        }
}
```

Note: Thử chạy killall -0 haproxy nếu server báo command not found, bạn cần cài gói yum -y install psmisc.


Sau khi cấu hình xong, start dịch vụ và cho phép khởi động cùng hệ thống

systemctl start keepalived
systemctl enable keepalived


yum install haproxy -y

vi /etc/haproxy/haproxy.cfg

global
        daemon
        maxconn 256

defaults
        mode http
        timeout connect 5000ms
        timeout client 50000ms
        timeout server 50000ms
        stats enable
        stats uri /monitor
        stats auth root:123456

listen httpd
    bind 10.10.10.180:80
    balance  roundrobin
    mode  http
    server web1 10.10.10.170:80 check
    server web2 10.10.10.171:80 check

systemctl start haproxy
systemctl enable haproxy	


firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --reload



yum -y install policycoreutils-python
semanage port -m -t http_port_t -p tcp 8080
systemctl restart haproxy
systemctl status haproxy
setsebool -P haproxy_connect_any=1


modprobe ip_conntrack







