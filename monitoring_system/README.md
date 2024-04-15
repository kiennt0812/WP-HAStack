# Monitoring system: Node_exporter, Prometheus & Grafana
![](https://github.com/kiennt0812/WP-HAStack/blob/main/images/Monitoring.png?raw=true)
- **Node Exporter** is a tool that helps collect system information and display them as metrics, helping you monitor and manage nodes in your infrastructure.
- **Prometheus** will perform the process of pulling parameters/metrics from jobs (exporter) and storing the collected data. Prometheus will run rules to process data as needed and test the alerts you desire.
- **Grafana** helps build data into graphs and charts for visualization. Helps us easily monitor and closely monitor the system.

## Preparation:

| Hostname | OS | IP address |
|--------------|-------|------|
| manager | ubuntu 20.04 | 192.168.64.21 |

## Node Exporter install & configure 
**Executed on all servers that need monitoring of the system**

- Step1 : Create a user that will be used by Node Exporter

    ```bash
    useradd -M -r -s /bin/false node_exporter
    ```

- Step2 : Download and unzip

    ```bash
    wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
    ```
    ```bash
    tar -xvf node_exporter-1.3.1.linux-amd64.tar.gz
    ```

- Step3: Copy binary to `/usr/local/bin` and change owner.

    ```bash
    cp node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin/
    ```
    ```bash
    chown node_exporter:node_exporter  /usr/local/bin/node_exporter
    ```

- Step4 : Create an init at `/etc/systemd/system/node_exporter.service` so systemd can manage this service. 

    ```bash
    [Unit]
    Description=Node Exporter
    Wants=network-online.target
    After=network-online.target

    StartLimitIntervalSec=500
    StartLimitBurst=5

    [Service]
    User=node_exporter
    Group=node_exporter
    Type=simple
    Restart=on-failure
    RestartSec=5s
    ExecStart=/usr/local/bin/node_exporter \
        --collector.logind

    [Install]
    WantedBy=multi-user.target
    ```

- Reload systemctl and start service

    ```bash
    systemctl daemon-reload
    systemctl start node_exporter
    systemctl status node_exporter
    systemctl enable node_exporter
    ```
## Prometheus MySQL Exporter install & configure 
**Executed on Database servers**
- Step1 : Create Prometheus exporter database user 
    ```bash
    mysql> CREATE USER 'mysqld_exporter'@'%' IDENTIFIED BY 'Trungkien1998@';
    mysql> GRANT ALL PRIVILEGES ON *.* TO 'mysqld_exporter'@'%';
    mysql> FLUSH PRIVILEGES;
    mysql> EXIT
    ```
- Step2 : Configure MySQL login information at `/etc/.mysqld_exporter.cnf` :
    ```bash
    [client]
    user=mysqld_exporter
    password=Trungkien1998@
    ```
    ```bash
    chown root:prometheus /etc/.mysqld_exporter.cnf
    ```

- Step3 : Create a user and group that will be used by  Prometheus MySQL Exporter

    ```bash
    groupadd --system prometheus
    useradd -s /sbin/nologin --system -g prometheus prometheus
    ```

- Step4 : Download and unzip Prometheus MySQL Exporter 

    ```bash
    curl -s https://api.github.com/repos/prometheus/mysqld_exporter/releases/latest   | grep browser_download_url   | grep linux-amd64 | cut -d '"' -f 4   | wget -qi - 
    ```
    ```bash
    tar xvf mysqld_exporter*.tar.gz
    ```

- Step5: Copy binary to `/usr/local/bin` and chmod.

    ```bash
    cp  mysqld_exporter-*.linux-amd64/mysqld_exporter /usr/local/bin/
    ```
    ```bash
    chmod +x /usr/local/bin/mysqld_exporter
    ```

- Step6 : Create an init at `/etc/systemd/system/mysql_exporter.service` so systemd can manage this service. 

    ```bash
    [Unit]
    Description=Prometheus MySQL Exporter
    After=network.target
    User=prometheus
    Group=prometheus

    [Service]
    Type=simple
    Restart=always
    ExecStart=/usr/local/bin/mysqld_exporter \
    --config.my-cnf /etc/.mysqld_exporter.cnf \
    --collect.global_status \
    --collect.info_schema.innodb_metrics \
    --collect.auto_increment.columns \
    --collect.info_schema.processlist \
    --collect.binlog_size \
    --collect.info_schema.tablestats \
    --collect.global_variables \
    --collect.info_schema.query_response_time \
    --collect.info_schema.userstats \
    --collect.info_schema.tables \
    --collect.perf_schema.tablelocks \
    --collect.perf_schema.file_events \
    --collect.perf_schema.eventswaits \
    --collect.perf_schema.indexiowaits \
    --collect.perf_schema.tableiowaits \
    --collect.slave_status \
    --web.listen-address=0.0.0.0:9104       #Port exporter service

    [Install]
    WantedBy=multi-user.target
    ```

- Reload systemctl and start service

    ```bash
    systemctl daemon-reload
    systemctl start mysql_exporter
    systemctl enable mysql_exporter
    systemctl status mysql_exporter
    ```
## Prometheus install & configure
**Executed on monitor server**
- Step1 : Create a user that will be used by Prometheus
    ```
    useradd -M -r -s /bin/false prometheus
    ```
- Step2 : Create folders containing configuration and data for Prometheus 
    ```bash
    mkdir /etc/prometheus /var/lib/prometheus
    ```
- Step3 : Download and unzip

    ```bash
    wget https://github.com/prometheus/prometheus/releases/download/v2.32.1/prometheus-2.32.1.linux-amd64.tar.gz
    ```
    ```bash
    tar -xvf prometheus-2.32.1.linux-amd64.tar.gz
    ```
- Step4: Copy and change owner.:

    ```bash
    cp prometheus-2.32.1.linux-amd64/{prometheus,promtool} /usr/local/bin
    chown prometheus:prometheus /usr/local/bin/{prometheus,promtool}
    cp -r prometheus-2.32.1.linux-amd64/{consoles,console_libraries} /etc/prometheus/
    cp prometheus-2.32.1.linux-amd64/prometheus.yml /etc/prometheus/prometheus.yml 
    chown prometheus:prometheus -R /etc/prometheus/
    chown prometheus:prometheus -R /var/lib/prometheus/

    ```

- Step5 : Create an init at `/etc/systemd/system/prometheus.service` so systemd can manage this service. 

    ```bash
    [unit]
    Description=Prometheus
    Wants=network-online.target
    After=network-online.target

    StartLimitIntervalSec=500
    StartLimitBurst=5

    [Service]
    User=prometheus
    Group=prometheus
    Type=simple
    Restart=on-failure
    RestartSec=5s
    ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090 \
    --web.enable-lifecycle

    [Install]
    WantedBy=multi-user.target
    ```

- Reload systemctl and start service
    ```bash
    systemctl daemon-reload
    systemctl start prometheus
    systemctl status prometheus
    systemctl enable prometheus
    ```
- Step6: configure prometheus `/etc/promtail/config-promtail.yml`
    ```bash
    # my global config
    global:
    scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
    evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
    # scrape_timeout is set to the global default (10s).

    # Alertmanager configuration
    alerting:
    alertmanagers:
        - static_configs:
            - targets:
            # - alertmanager:9093

    # Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
    rule_files:
    # - "first_rules.yml"
    # - "second_rules.yml"

    # A scrape configuration containing exactly one endpoint to scrape:
    # Here it's Prometheus itself.
    scrape_configs:
    # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
    - job_name: "prometheus"

        # metrics_path defaults to '/metrics'
        # scheme defaults to 'http'.

        static_configs:
        - targets: ["192.168.64.10:9100","192.168.64.11:9100","192.168.64.12:9100","192.168.64.13:9100","192.168.64.14:9100","192.168.64.16:9100","192.168.64.17:9100","192.168.64.18:9100","192.168.64.19:9100"]
    - job_name: "Database"

        # metrics_path defaults to '/metrics'
        # scheme defaults to 'http'.

        static_configs:
        - targets: ["192.168.64.10:9104","192.168.64.11:9104"]
        ```
- Reload new config and Restart 
    ```bash
    systemctl daemon-reload
    systemctl restart prometheus
    ```
## Grafana install & configure
**Executed on monitor server**
- Step1 : Make sure dependencies are installed. Then create 1 CPG key
    ```bash
    apt-get install -y apt-transport-https software-properties-common
    wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
    ```

- Step2 : Add repository, `apt update` and install grafana
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
### Visualizing Prometheus Metrics with Grafana:
- Step1: Add Prometheus as a Data Source in Grafana:
    - Access Grafana at **192.168.64.21:3000**, log in (admin/  admin), and go to Configuration > Data Sources.
![](https://github.com/kiennt0812/WP-HAStack/blob/main/images/data_sources1.png?raw=true)
    - Add Prometheus as the data source, set the Prometheus server URL, and save the configuration.
- Step 2: Create a Dashboard:
    - In Grafana, create a new dashboard by clicking on the plus icon and selecting "Dashboard."
    - Select **Import dashboard** to use available templates.
![](https://github.com/kiennt0812/WP-HAStack/blob/main/images/import_dashboard.png?raw=true)
- Step 3 : Next, you can select and copy the ID of the dashboard you want from the grafana homepage https://grafana.com/grafana/dashboards/ . Fill them in the box and select Load :
![](https://github.com/kiennt0812/WP-HAStack/blob/main/images/load_dashboard.png?raw=true)

After uploading, you will have a dashboard like this. The information about **CPU, Mem, Disk**... is shown specifically and in detail, you click to see details of each option. The status of each server in the system will also be displayed:
![](https://github.com/kiennt0812/WP-HAStack/blob/main/images/dashboard.png?raw=true)