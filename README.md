
# Build a High Availability 
## Introduction
- High availability is a critical success factor for any given enterprise application.
- High availability can be defined as running a system 24*7 without a downtime even if  there are hardware and software failures.
- Logging and monitoring are both valuable components to maintaining optimal application performance. Using a combination of logging tools and real-time monitoring systems helps improve observability and reduces the time spent sifting through log files to determine the root cause of performance problems.
- Following is an architecture that supports high availability. It’s the minimal requirement to implement high availability in your application.
![](https://github.com/kiennt0812/WP-HAStack/blob/main/images/mohinh.png?raw=true)
## Install & Configure
### [1. Galera cluster](https://github.com/kiennt0812/WP-HAStack/tree/main/galera_cluster)
- Galera cluster is a solution that uses the Master-Master model for database management systems. Using galera cluster, application can read/write on any node. 
### [2. Load Balancer for Galera cluster](https://github.com/kiennt0812/WP-HAStack/tree/main/Loadbalancer_galeracluster)
- A database Load Balancer is a middleware service that stands between applications and databases. It distributes the workload across multiple database servers running behind it.
- The goals of having database load balancing are to provide a single database endpoint to applications to connect to, increase queries throughput, minimize latency and maximize resource utilization of the database servers. 
### [3. Application(Wordpress)](https://github.com/kiennt0812/WP-HAStack/tree/main/wordpress_application)
- Server Cluster is understood as a system consisting of many servers operating together and sharing resources to increase system performance and reliability.
- In a Server Cluster, servers connect together and operate as a single system to serve users or applications without interruption.
### [4. Load Balancer for Application(Wordpress)](https://github.com/kiennt0812/WP-HAStack/tree/main/Loadbalancer_wordpress)
- Load balancing refers to efficiently distributing incoming network traffic across a group of backend servers, also known as a server farm or server pool.
- Modern high‑traffic websites must serve hundreds of thousands, if not millions, of concurrent requests from users or clients and return the correct text, images, video, or application data, all in a fast and reliable manner. To cost‑effectively scale to meet these high volumes, modern computing best practice generally requires adding more servers.
### [5. Monitoring system: Node_exporter, Prometheus & Grafana ](https://github.com/kiennt0812/WP-HAStack/tree/main/monitoring_system)
- Monitoring system is a software or hardware capable of continuously monitoring and recording the status and activities of a system.
- In the past, one administrator/developer only intervened when a system error occurred, which came with a lot of risk in fixing the problems, or only used a few small tools, cannot provide a comprehensive view, which costs time and money.
- Monitoring system is a suitable solution to help administrators/developers easily monitor and evaluate the system comprehensively (analyzing cpu, ram, disk...) thereby building a stable and stable system. such as quick response to incidents that occur. In addition, the characteristic of the Monitoring system is centralized management, so when you encounter a problem, you will receive an immediate notification to help us resolve it quickly, save time and create peace of mind about the system.
### [6. Logging manager: Promtail, Loki & Grafana ](https://github.com/kiennt0812/WP-HAStack/tree/main/logging_manager)
- Logging is the process of recording operational information of a system or application while it is running. The use of logs is an important part of system management, helping to track and analyze errors, performance, application activity and behavior.
- Logging tools play an important role in helping to manage and maintain applications and systems effectively, providing detailed and useful information to solve problems and improve operations.Logging tools play an important role in helping to manage and maintain applications and systems effectively, providing detailed and useful information to solve problems and improve operations.