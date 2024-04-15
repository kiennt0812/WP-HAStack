
## Install and configure database cluster

![](https://github.com/kiennt0812/WP-HAStack/blob/main/images/galera.png?raw=true)
### - Preparation :

| Hostname | OS | MySQL| Role | IP address |
|--------------|-------|------|-------|-------|
| db1 | ubuntu 20.04  | ver 8.0 | Master | 192.168.64.10 |
| db2 | ubuntu 20.04  | ver 8.0 | Master | 192.168.64.11 |
| db3 | ubuntu 20.04  | ver 8.0 | Master | 192.168.64.12 | 


### 1. Add repositorys  
- You can now edit the `/etc/apt/sources.list.d/galera.list` to include the following lines
    ```sh
    deb https://releases.galeracluster.com/galera-4/ubuntu focal main
    deb https://releases.galeracluster.com/mysql-wsrep-8.0/ubuntu focal main
    ```
- You should also pin the repository by editing `/etc/apt/preferences.d/galera.pref`  
    ```sh
    # Prefer the Codership repository
    Package: *
    Pin: origin releases.galeracluster.com
    Pin-Priority: 1001
    ```       
- Don't forget to make ensure that the Galera Cluster GPG key is installed:
     ```sh
    apt-key adv --keyserver keyserver.ubuntu.com --recv 8DA84635
    ```     

### 2.  Install Galera 4 with MySQL 8:
- You should now run an `apt update` and then install Galera 4 with MySQL 8:

    ```diff
    apt install galera-4 mysql-wsrep-8.0
    ```
- Select `Use Strong Password Encryption` then `OK` :

    ![](https://galeracluster.com/wp-content/uploads/2024/02/ubu2204-04-604x270.png)
### 3. Configure Galera 4
- You can edit `/etc/mysql/mysql.conf.d/mysqld.cnf` and add a basic configuration: 
    ```sh
    [mysqld]
    pid-file        = /var/run/mysqld/mysqld.pid
    socket          = /var/run/mysqld/mysqld.sock
    datadir         = /var/lib/mysql
    log-error       = /var/log/mysql/error.log

    binlog_format=ROW
    default-storage-engine=innodb
    innodb_autoinc_lock_mode=2
    bind-address=0.0.0.0

    # Galera Provider Configuration
    wsrep_on=ON
    wsrep_provider=/usr/lib/galera/libgalera_smm.so

    # Galera Cluster Configuration
    wsrep_cluster_name="galera"
    wsrep_cluster_address="gcomm://192.168.64.10,192.168.64.11,192.168.64.12"  #All IP of cluster

    # Galera Synchronization Configuration
    wsrep_sst_method=rsync

    # Galera Node Configuration
    wsrep_node_address="192.168.64.10"  # IP Node1
    ```
- Execute `systemctl stop mysql`. Run `mysqld_bootstrap` only on the first node.


- **Note**: An error may occur during synchronization of databases due to AppArmor blocking access rights.
    To temporarily disable AppArmor for the MySQL service:

    ```diff
    sudo ln -s /etc/apparmor.d/usr.sbin.mysqld /etc/apparmor.d/disable/usr.sbin.mysqld
    ls -l /etc/apparmor.d/disable/usr.sbin.mysqld  #make sure link is established
    sudo /etc/init.d/apparmor restart
    apparmor_parser -R /etc/apparmor.d/usr.sbin.mysqld
    ```
- *Continue repeating the above steps with the remaining nodes.*
### 4. Check the result:
- Now, when you bring up all 3 nodes as simply as `systemctl start mysql`, you can execute command below and will see that the wsrep_cluster_size has increased to 3. 
    ```diff
    mysql -u root -p -e "show status like 'wsrep_cluster_size'"
    Enter password: 
    +--------------------+-------+
    | Variable_name      | Value |
    +--------------------+-------+
    | wsrep_cluster_size | 3     |
    +--------------------+-------+
    ```
- You can also choose to test replication by creating a database and table on one node, and see that the replication is happening in real time.

### 5. CREAT USER and Database
- Once the Galera cluster is installed and configured in your server, create a user and a database.  
```diff
$ mysql -u root -p
Enter password:

GRANT ALL ON wordpress_db.* TO 'haproxy_root'@'%' IDENTIFIED BY 'Trungkie1998@' WITH GRANT OPTION;
FLUSH PRIVILEGES;

CREATE DATABASE wordpress_db;
```