# Logging manager: Promtail, Loki & Grafana
![](https://github.com/kiennt0812/WP-HAStack/blob/main/images/logging_manager.png?raw=true)
- **Promtail**: an agent installed on servers that need to monitor logs, it will be responsible for collecting logs and sending them to Loki(PUSH).
- **Loki** is a distributed logs storage system that helps store and query logs at scale. It provides the ability to search and query logs based on labels.
- **Grafana** is a data visualization and monitoring tool.

## Preparation:

| Hostname | OS | IP address |
|--------------|-------|------|
| manager | ubuntu 20.04 | 192.168.64.21 |

## Promtail install & configure 
**Executed on all servers that need monitoring of the system**

- Step1 : Create a user and group that will be used by Promtail
    ```bash
    useradd --system promtail
    ```
    ```bash
    usermod -a -G adm promtail
    ```
- Step2 : Download and unzip

    ```bash
    cd /usr/local/bin
    curl -O -L "https://github.com/grafana/loki/releases/download/v2.4.1/promtail-linux-amd64.zip"
    ```
    ```bash
    unzip "promtail-linux-amd64.zip"
    ```

- Step3: Change permission

    ```bash
    chmod a+x "promtail-linux-amd64"
    ```

- Step4 : Create an init at `/etc/systemd/system/promtail.service` so systemd can manage this service. 

    ```bash
    [Unit]
    Description=Promtail service
    After=network.target

    [Service]
    Type=simple
    User=promtail
    ExecStart=/usr/local/bin/promtail-linux-amd64 -config.file /etc/promtail/config-promtail.yml

    [Install]
    WantedBy=multi-user.target
    ```
- Step5 : Create a directory containing Promtail configuration, then configure it
    ```
    mkdir /etc/promtail
    ```
    ```
    vi /etc/promtail/config-promtail.yml
    ```
    ```
    server:
    http_listen_port: 9080
    grpc_listen_port: 0

    positions:
    filename: /tmp/positions.yaml  

    clients:
            - url: 'http://192.168.64.21:3100/loki/api/v1/push' 

    scrape_configs:
    - job_name: system
        static_configs:
        - targets:
            - localhost
            labels:
            job: varlogs
            __path__: /var/log/*log
            host: 192.168.64.17

    - job_name: nginx
        static_configs:
        - targets:
            - localhost
            labels:
            job: nginx
            __path__: /var/log/nginx/*log
            host: 192.168.64.17
        pipeline_stages:
        - match:
            selector: '{job="nginx"}'
            stages:
            - regex:
                expression: '^(?P<remote_addr>[\w\.]+) - (?P<remote_user>[^ ]*) \[(?P<time_local>.*)\] "(?P<method>[^ ]*) (?P<request>[^ ]*) (?P<protocol>[^ ]*)" (?P<status>[\d]+) (?P<body_bytes_sent>[\d]+) "(?P<http_referer>[^"]*)" "(?P<http_user_agent>[^"]*)"?'
            - labels:
                remote_addr:
                remote_user:
                time_local:
                method:
                request:
                protocol:
                status:
                body_bytes_sent:
                http_referer:
                http_user_agent:
    ```
    Reload systemctl and start service
    ```bash
    systemctl daemon-reload
    systemctl start promtail
    systemctl enable promtail
    systemctl status promtail
    ```
## Loki install & configure 
**Executed on monitor server**
- Step1 : Create a user and group that will be used by Promtail
    ```bash
    useradd --system loki
    ```
- Step2 : Download and unzip

    ```bash
    cd /usr/local/bin
    curl -O -L "https://github.com/grafana/loki/releases/download/v2.9.7/loki-linux-amd64.zip"
    ```
    ```bash
    unzip "loki-linux-amd64.zip"
    ```
- Step3 : Create an init at `/etc/systemd/system/loki.service` so systemd can manage this service. 

    ```bash
    [Unit]
    Description=Loki service
    After=network.target

    [Service]
    Type=simple
    User=loki
    ExecStart=/usr/local/bin/loki-linux-amd64 -config.file /etc/loki/config-loki.yml

    [Install]
    WantedBy=multi-user.target
    ```
- Step4 : Create a directory containing Loki configuration, then configure it
    ```
    mkdir /etc/loki
    ```
    ```
    vi /etc/loki/config-loki.yml
    ```
    ```
    auth_enabled: false

    server:
    http_listen_port: 3100                 
    grpc_listen_port: 9096

    common:
    path_prefix: /tmp/loki
    storage:                              
        filesystem:
        chunks_directory: /tmp/loki/chunks  
        rules_directory: /tmp/loki/rules
    replication_factor: 1
    ring:
        kvstore:
        store: inmemory

    schema_config:                      
    configs:      
        - from: 2024-03-24                
        store: boltdb-shipper         
        object_store: filesystem               
        schema: v11
        index:                           
            prefix: index_                 
            period: 24h                      

    ruler:
    alertmanager_url: http://localhost:9093

    ```
    Reload systemctl and start service

    ```bash
    systemctl daemon-reload
    systemctl start loki
    systemctl enable loki
    systemctl status loki
    ```

## Grafana install & configure
**Executed on monitor server**
- Step1 : Make sure dependencies are installed. Then create 1 CPG key
    ```bash
    apt-get install -y apt-transport-https software-properties-common
    wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
    ```

-  Step2 : Add repository, `apt update` and install grafana
    ```bash
    echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
    apt update
    apt-get -y install grafana
    ```
- Step3 : Start service

    ```bash
    systemctl start grafana-server
    systemctl status grafana-server
    systemctl enable grafana-server
    ```
### Visualizing Loki with Grafana:

- Step1: Add Loki as a Data Source in Grafana:
    - Access Grafana at **192.168.64.21:3000**, log in (admin/  admin), and go to Configuration > Data Sources.
    ![](https://github.com/kiennt0812/WP-HAStack/blob/main/images/data_sources1.png?raw=true)
    - Add Loki as the data source, set the Loki server URL, and save the configuration.
- Step 2: Check log Loki
    - Click on explore in the below screenshot after adding the data source.
    ![](https://github.com/kiennt0812/WP-HAStack/blob/main/images/explore_log.jpg?raw=true)
    - In the label filters, we can choose job and varlogs which is generally the path /var/log/*log in the backend to show all the system logs.
    ![](https://github.com/kiennt0812/WP-HAStack/blob/main/images/query_log.jpg?raw=true)
    - Click on the run query in the above screenshot to execute and show all the system logs as below.
    ![](https://github.com/kiennt0812/WP-HAStack/blob/main/images/query_log_result.jpg?raw=true)