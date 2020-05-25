On-premise Installation on Linux
==================================
    
Installing Java
***************

    # install Java 9
    sudo yum -y update
    sudo yum -y remove java

    # Change the open socket limits
    sudo echo "*     soft   nofile  16384" >> /etc/security/limits.conf
    sudo echo "*     hard   nofile  20000" >> /etc/security/limits.conf
    sudo echo "net.ipv4.tcp_max_syn_backlog = 2048" >> /etc/sysctl.conf

    #sudo rpm -ivh /tmp/install/jdk-9_linux-x64_bin.rpm
    sudo yum install -y java-9-openjdk


Setting up the database node
****************************
    1. Obtain the following files (from a dedicated S3 bucket)
        a. cassandra-topology.properties
        b. root.crt and root.key (or generate them)
        c. cassandra-truststore.jks
        d. setup-cassandra.sh
        e. update-cassandra-cluster-config.py
        f. cassandra.yaml.template
    2. Place all of the files in /tmp/install/
    3. Fill in the IP addresses of database nodes in cassandra-topology.properties
    4. Change ${seeds} to a comma-separated list of IPs of Cassandra nodes in cassandra.yaml.template

    Installing Cassandra (setup-cassandra.sh):
    
	.. code-block:: bash
	
		sudo cat <<EOT >>  /etc/yum.repos.d/cassandra.repo
			[cassandra]
			name=Apache Cassandra
			baseurl=https://www.apache.org/dist/cassandra/redhat/311x/
			gpgcheck=1
			repo_gpgcheck=1
			gpgkey=https://www.apache.org/dist/cassandra/KEYS
			EOT

		sudo yum -y install cassandra

		
Setting up the search node
**************************
    1. Obtain the following files (from a dedicated S3 bucket)
        a. setup-elasticsearch.sh
    2. Place it in /tmp/install/
    3. Run setup-elasticsearch.sh

    Installing ElasticSearch (setup-elasticsearch.sh)
    
	.. code-block:: bash
	
        BIND_IP=`ifconfig | sed -En 's/127.0.0.1//;s/.*inet (addr:)?(([0-9]*\.){3}[0-9]*).*/\2/p'`

        rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

        sudo yum -y update

        sudo cat <<EOT >>  /etc/yum.repos.d/elasticsearch.repo
        [elasticsearch-6.x]
        name=Elasticsearch repository for 6.x packages
        baseurl=https://artifacts.elastic.co/packages/6.x/yum
        gpgcheck=1
        gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
        enabled=1
        autorefresh=1
        type=rpm-md
        EOT

        sudo yum -y install elasticsearch

        sudo sed -i -- 's/#cluster.name: my-application/cluster.name: logsentinel/g' /etc/elasticsearch/elasticsearch.yml
        sudo sed -i -- 's/#node.name: node-1/node.name: ls-node-1/g' /etc/elasticsearch/elasticsearch.yml
        sudo sed -i -- "s/#network.host: 192.168.0.1/network.host: $BIND_IP/g" /etc/elasticsearch/elasticsearch.yml
        sudo sed -i -- 's/#discovery.zen.ping.unicast.hosts: \["host1", "host2"\]/discovery.zen.ping.unicast.hosts: \["", ""\]/g' /etc/elasticsearch/elasticsearch.yml
        sudo sed -i -- 's/#discovery.zen.minimum_master_nodes:/discovery.zen.minimum_master_nodes: 2/g' /etc/elasticsearch/elasticsearch.yml

        sudo sed -i -- 's/-Xms1g/-Xms1500m/g' /etc/elasticsearch/jvm.options
        sudo sed -i -- 's/-Xmx1g/-Xmx1500m/g' /etc/elasticsearch/jvm.options

        sudo chkconfig --add elasticsearch
        sudo chkconfig elasticsearch on

        sudo service elasticsearch start

		

Setting up the load balancer node (optional)
********************************************

Setting up a load balancer is optional. You can use a single application node or you can use round-robin DNS to balance between multiple nodes.  The round-robin DNS doesn’t ensure perfect high availability, as a dead node will still be referenced. If you want to setup a load balancer, we’d recommend issuing a Let’s encrypt certificate for it.
    1. Obtain the following files (from a dedicated S3 bucket)
        a. setup-loadbalancer.sh
    2. Place it in /tmp/install/
    3. Edit the IP addresses in the script (currently 10.1.0.101)
    4. Run setup-loadbalancer.sh <domain> <contact-email>

    (setup-loadbalancer.sh)
	
	.. code-block:: bash
	
        #!/bin/sh

        DOMAIN=$1
        EMAIL=$2

        sudo yum -y install epel-release

        sudo yum -y update
        sudo yum -y install nginx
        sudo yum -y install firewalld

        sudo systemctl start firewalld
        sudo systemctl enable firewalld
        sudo systemctl status firewalld

        sudo firewall-cmd --zone=public --permanent --add-service=https
        sudo firewall-cmd --zone=public --permanent --add-service=http
        sudo firewall-cmd --reload

        sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

        sudo yum -y install certbot-nginx

        sudo cat <<EOT >>  /etc/nginx/conf.d/load-balancer.conf

        upstream backend {
        server 10.1.0.101:8080 max_fails=3 fail_timeout=30s;
        server 10.1.0.102:8080 max_fails=3 fail_timeout=30s;
        }

        server {
        listen 80;
        server_name $DOMAIN;

        location / {
            proxy_pass http://backend
            proxy_set_header Host            $host;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
        location /.well-known/acme-challenge/ {
            root /usr/share/nginx/html;
            default_type text/plain;
        }
        }

        stream {
            upstream syslog_backend {
            server 10.1.0.101:1514 max_fails=3 fail_timeout=30s;
            server 10.1.0.102:1514 max_fails=3 fail_timeout=30s;
            }

            server {
                listen     127.0.0.1:514;
                proxy_pass syslog_backend;
            }
        }

        EOT

        # Install certificate (it automatically updates the load-balancer.conf)
        # we use different installer and authenticator plugins because we don't want to restart nginx on renewal
        sudo certbot -a webroot -i nginx -w /usr/share/nginx/html -d $DOMAIN --noninteractive --agree-tos -m $EMAIL

        # only needed in case SELinux is present - allowing connecting to the app nodes
        setsebool -P httpd_can_network_connect 1

        sudo service nginx start
        sudo chkconfig nginx on

        # Auto-renewal of Letsencrypt ceretificates
        (crontab -l 2>/dev/null; echo "15 3 * * * /usr/bin/certbot renew --quiet --renew-hook \"service nginx reload\"") | crontab -




Setting up the application node
*******************************
    1. Obtain the following files (from a dedicated S3 bucket)
        a. logsentinel-x.x.x.jar – the application node binary
        b. sample app.properties – contains all configuration options
        c. sample tsa-store.jks – used for local trusted timestamping
        d. sample cassandra-truststore.jks – used for communication with Cassandra nodes
        e. setup.sh – used to setup and run the application
        f. logsentinel.conf – Java configuration options for the application
        g. setup-nfs-client.sh and setup-nfs-server.sh – used for sharing files in case of multiple application nodes
        h. jdk-9_linux-x64_bin.rpm
    2. Place all .sh files and the rpm file in /tmp/install
    3. Place all other files in /var/logsentinel
    4. Configure app.properties by changing the needed properties (see below)
    5. Run setup.sh with: > setup.sh
    6. In case of multiple application nodes, you’d also run setup-nfs-server.sh <CLIENT_IP> and setup-nfs-client.sh <SERVER_IP> on different machine so that /var/nfs becomes a shared file system


    Setting up the logsentinel service(setup.sh)
    
	.. code-block:: bash
	
        # install Java 9
            sudo yum -y update
            sudo yum -y remove java

        # Change the open socket limits
            sudo echo "*     soft   nofile  16384" >> /etc/security/limits.conf
            sudo echo "*     hard   nofile  20000" >> /etc/security/limits.conf
            sudo echo "net.ipv4.tcp_max_syn_backlog = 2048" >> /etc/sysctl.conf

        #sudo rpm -ivh /tmp/install/jdk-9_linux-x64_bin.rpm
        sudo yum install -y java-9-openjdk

        # setup the logsentinel service
		adduser logsentinel
		chown logsentinel:logsentinel /var/logsentinel/logsentinel.jar
		ln -s /var/logsentinel/logsentinel.jar /etc/init.d/logsentinel
		chmod +x /etc/init.d/logsentinel

		mkdir -p /var/log/logsentinel/access
		chmod 777 /var/log/logsentinel/access

		service logsentinel start
		chkconfig logsentinel on


	(setup-nfs-client.sh)
		
	.. code-block:: bash
	
        SERVER_IP=$1
        sudo yum -y install nfs-utils
        sudo mkdir -p /mnt/nfs/var/nfs
        sudo mount $SERVER_IP:/var/nfs /mnt/nfs/var/nfs
        sudo echo "$SERVER_IP:/var/nfs  /mnt/nfs/var/nfs   nfs      rw,sync,hard,intr  0     0" >> /etc/fstab
        ln -s /mnt/nfs/var/nfs /var/nfs

		
    (setup-nfs-server.sh)
	
	.. code-block:: bash
	
        CLIENT_IP=$1

        sudo yum -y install nfs-utils
        sudo systemctl enable nfs-server.service
        sudo systemctl start nfs-server.service

        sudo mkdir /var/nfs
        sudo chown nfsnobody:nfsnobody /var/nfs
        sudo chmod 755 /var/nfs

        sudo echo "/var/nfs        $CLIENT_IP(rw,sync,no_subtree_check)" >> /etc/exports
        sudo exportfs -a



Properties to be configured:
***************************
    • spring.mail.* - configure an outgoing mail server
    • registration.email.from, generic.email.from – outgoing mails would be sent from these addresses
    • admin.username, admin.password – used to access the admin panel of the system (note: the username is <username>@logsentinel.com, i.e. if you configure admin.username=test, you’d be able to login with test@logsentinel.com)
    • spring.security.user.password – password used to access an application management dashboard
    • hmac.key  - an alphanumeric key used for calculating HMACs
    • jwt.secret – a secret alphanumeric key used for JWT session tokens
    • etherscan.key, ethereum.private.key, ethereum.chain.id, ethereum.store.days.interval – configuring the push of hashes to Ethereum. We utilize the Etherscan API for which a key should be obtained. By default chain.id is 3 which is a test net. In the test net ether can be obtained for free from faucets.
    • cassandra.hosts – comma-separated list of the IPs of the database nodes
    • elasticsearch.url – a comma-separated list of elasticsearch URLs, e.g. http://172.10.12.15:9200,http://172.10.12.16:9200
    • hazelcast.nodes – comma-separated list of IP addresses of application nodes for the purposes of distributed locking.
    • root.url – the URL under which the web UI will be accessed. Can be an IP address, e.g. https://172.10.12.17. 