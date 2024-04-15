
## Load Balancer for Galera cluster

![](https://github.com/kiennt0812/WP-HAStack/blob/main/images/lb_34.png?raw=true)
- HAProxy : is a versatile and powerful open-source load balancer and proxy server software. It is designed to efficiently distribute incoming network traffic across multiple servers to ensure high availability, reliability, and optimal performance for web applications and services.
- Keepalived : is an open-source software package that provides high availability (HA) and failover capabilities for Linux-based systems. It is primarily used to manage Virtual IP (VIP) addresses and ensure uninterrupted service availability in scenarios where system or application failures can occur.

### - Preparation :

| Hostname | OS | Keepalived Role | IP address |
|--------|------|------|-------|
| lb3 | ubuntu 20.04  |  Master | 192.168.64.13 |
| lb4 | ubuntu 20.04  |  Backup | 192.168.64.14 |
 


### 1. Install & Configure HAProxy   
- On the HAProxy server (lb3) install the package.
    ```sh
    apt-get install haproxy
    ```
- You can edit `/etc/haproxy/haproxy.cfg` and add a basic configuration:  
    ```sh
    global
        log 127.0.0.1 local0 notice
        user haproxy
        group haproxy

    defaults
        log global
        retries 2
        timeout connect 3000
        timeout server 5000
        timeout client 5000

    listen mysql-cluster
        bind 0.0.0.0:3306
        mode tcp
        balance roundrobin
        server db1 192.168.64.10:3306 check
        server db2 192.168.64.11:3306 check
        server db3 192.168.64.12:3306 check

    ```
- Once you’re done configuring start the HAProxy service.
    ```diff
    systemctl start haproxy 
    systemctl status haproxy 
    systemctl enable haproxy 
    ```
- Continue repeating the above steps with the lb4 server.
- After doing this, the server will be running HAProxy. When new connections are made to this server, it routes them through to nodes in the Galera cluster.
### 2. Install & Configure Keepalived  
- On the HAProxy server (lb3) install the package.
    ```sh
    apt-get install keepalived
    ```
- You can edit `/etc/keepalived/keepalived.conf` and add a basic configuration:  
    ```sh
    global_defs {
        router_id test1
    }

    vrrp_script chk_haproxy {
        script "killall -0 haproxy"
        interval 2
        weight 2
    }
   
    vrrp_instance VI_1 {
        virtual_router_id 51
        advert_int 1
        priority 101
        state MASTER
        interface ens32
        virtual_ipaddress {
            192.168.64.15 dev ens32
        }
        authentication {
            auth_type PASS
            auth_pass 123456
        }
        track_script {
            chk_haproxy
        }
    }

    ```
- Continue configure on lb4 server
     ```diff
        global_defs {
        router_id test1
    }
    vrrp_script chk_haproxy {
        script "killall -0 haproxy"
        interval 2
        weight 2
    }
    
    vrrp_instance VI_1 {
        virtual_router_id 51                       
        advert_int 1
        priority 100                               
        state BACKUP
        interface ens32
        virtual_ipaddress {
            192.168.64.15 dev ens32
        }
        authentication {
            auth_type PASS
            auth_pass 123456
            }
        track_script {
            chk_haproxy
        }
    } 
    ```

- Once you’re done configuring start the HAProxy service.
    ```diff
    systemctl restart haproxy 
    ```

- After completed, we can use VIP ( Virtual IP : 192.168.64.15) to access the 2 HAProxy servers above. 
