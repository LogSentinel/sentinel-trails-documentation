Load Balancing and High Availability
====================================

On-premise installations would often require multiple application nodes, database nodes and search nodes to be run alongside for high availability. And therefore load balancing is required.

Application node load balancing
*******************************

One option of application node load balacing is to use `round-robin DNS <https://en.wikipedia.org/wiki/Round-robin_DNS>`_. That way a client will randomly connect to one of the listed nodes. Healtchecks will have to be implemented manually, however, because in case of one node going down, the DNS A-record will still contain its IP and send traffic to it.

Another opton is to configure an Nginx load-balancer and setup a Let's encrypt certificate using the following script:

.. code-block: bash

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

	sudo firewall-cmd --permanent --add-service=http -add-service=https
	sudo firewall-cmd --reload

	sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

	sudo yum -y install certbot-nginx

	sudo cat <<EOT >>  /etc/nginx/conf.d/load-balancer.conf

	upstream backend {
	   # IPs of the application nodes
	   server 10.0.0.101:8080 max_fails=3 fail_timeout=30s;
	   server 10.0.0.102:8080 max_fails=3 fail_timeout=30s;
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
		   # IPs of the application nodes
		   server 10.0.0.101:1514 max_fails=3 fail_timeout=30s;
		   server 10.0.0.102:1514 max_fails=3 fail_timeout=30s;
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

Database load balancing
***********************

There are multiple ways to do load balancing with Cassandra in case you are running more than one node (which is highly recommended):

* Configuring a comma-separated list of cassandra nodes with the ``cassandra.hosts`` property
* TCP Nginx load balancing as shown above for the syslog backend
* Round-robin DNS


Elasticsearch load balancing
****************************

When running multiple Elasticsearch nodes (which recommended), you have similar options:

* Configuring a comma-separated list of elasticsearch nodes with the ``elasticsearch.url`` property
* Use HTTP Nginx load balancing for port 9200 (as shown above)
* Round-robin DNS

Other configurations
********************

You have to set the ``hazelcast.nodes`` property to contain a comma-separated list of the IPs of application nodes