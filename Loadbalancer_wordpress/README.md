
## Load Balancer for Wordpress Appication

![]()
- HAProxy : is a versatile and powerful open-source load balancer and proxy server software. It is designed to efficiently distribute incoming network traffic across multiple servers to ensure high availability, reliability, and optimal performance for web applications and services.
- Keepalived : is an open-source software package that provides high availability (HA) and failover capabilities for Linux-based systems. It is primarily used to manage Virtual IP (VIP) addresses and ensure uninterrupted service availability in scenarios where system or application failures can occur.

### - Preparation :

| Hostname | OS | Keepalived Role | IP address |
|--------|------|------|-------|
| lb1 | ubuntu 20.04  |  Master | 192.168.64.18 |
| lb2 | ubuntu 20.04  |  Backup | 192.168.64.19 |
 
### 1. Create self-signed SSL 
In the Load Balancing model, HAProxy stands between the client and the backend servers. Therefore, we will request an SSL encrypted connection between the Client and HAProxy
- Step1 : Create your own self-signed certificate :
    - Create a directory containing the certificate:
    ```
    mkdir -p /etc/ssl/haproxyssl.local
    cd /etc/ssl/haproxyssl.local 
    ```
    - Create key `ssl.key` with RSA encryption:
    ```
    openssl genrsa -des3 -out ssl.key 2048
    ```
    Enter your optional passphrase
- Step2: Register a certificate (Certificate Signing Request -CSR)
    - Register Certificate:
    ```
    openssl req -new -key ssl.key -out ssl.csr
    ```
    Fill in some Organizational information.
    - Create certificate – CRT
    ```
    openssl x509 -req -days 365 -in ssl.csr -signkey ssl.key -out ssl.crt
    ```
- Step3: Create a file `.pem` from `.crt` and `.key`
    ```
    cat ssl.crt ssl.key > ssl.pem
    ```
- Finally, we will configure HAproxy to listen on port 443 (https) and transfer traffic from port 80 to 443 as in part 1.
- *Continue repeating the above steps with the lb2 server.*

### 2. Install & Configure HAProxy   
- On the HAProxy server lb1 install the package.
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

    frontend haproxy-main
        bind *:80
        bind *:443 ssl crt /etc/ssl/haproxyssl.local/ssl.pem ssl-min-ver TLSv1.2
        redirect scheme https code 301 if !{ ssl_fc }
        mode http
        option forwardfor
        default_backend nginx_webservers

    backend nginx_webservers
        balance roundrobin
        server app1      192.168.64.16:80 check
        server app2      192.168.64.17:80 check
    ```
- Once you’re done configuring start the HAProxy service.
    ```diff
    systemctl restart haproxy
    systemctl enable haproxy
    systemctl status haproxy
    ```
- Continue repeating the above steps with the lb2 server.
- After doing this, the server will be running HAProxy. When new connections are made to this server, it routes them through to nodes in the Wordpress Application.
### 3. Install & Configure Keepalived  
- On the HAProxy server lb1 install the package.
    ```sh
    apt-get install keepalived
    ```
- You can edit `/etc/keepalived/keepalived.conf` and add a basic configuration:  
    ```sh
    global_defs {
        router_id test2
    }

    vrrp_script chk_haproxy {
        script "killall -0 haproxy"
        interval 2
        weight 2
    }
   
    vrrp_instance VI_1 {
        virtual_router_id 52
        advert_int 1
        priority 101
        state MASTER
        interface ens32
        virtual_ipaddress {
            192.168.64.20 dev ens32
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
        router_id test2
    }
    vrrp_script chk_haproxy {
        script "killall -0 haproxy"
        interval 2
        weight 2
    }
    
    vrrp_instance VI_1 {
        virtual_router_id 52                       
        advert_int 1
        priority 100                               
        state BACKUP
        interface ens32
        virtual_ipaddress {
            192.168.64.20 dev ens32
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
    systemctl enable haproxy
    systemctl status haproxy
    ```

- After completed, we can use VIP ( Virtual IP : 192.168.64.20) to access the 2 HAProxy servers above. 
