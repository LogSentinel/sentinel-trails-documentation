On-premise Security
===================

In the cloud version of SentinelTrails we take all necessary operational security measures. However, in on-premise deployments this becomes the responsibility of the customer. Here are a few recommendations for keeping the installation secure.

Install TLS certificate
***********************

The certificate generation process can be seen in `this tutorial <https://docs.oracle.com/cd/E19798-01/821-1841/gjrgy/>`_. The properties that need to be set in ``application.properties`` are:

.. code:: text

	server.ssl.enabled=true
	server.ssl.key-store=/path/to/server.jks
	server.ssl.key-store-type=JKS
	server.ssl.key-store-password=secret
	server.ssl.key-alias=server
	server.ssl.key-password=secret
	secure.headers=true
	root.url=https://{your-root-url}
	root.url.api=https://{{your-root-url}


Configure network restrictions 
******************************

The Sentinel Trails (virtual) machines should only have the required ports opened. That includes: 

    * application nodes: 
        * incoming: 8080 (for web requests), 1514, 1515, 1516 (for syslog), 5701 (for hazelcast) and 22 for SSH.
        * outgoing: 80, 443, 9200 (for elastic search), 9042 (for cassandra), 5701, 465 (for smtp)
    * database nodes:
        * incoming:  7000, 7001, 7199, 9042, 9160 and 22
        * outgoing: 80, 443, 7000, 7001
    * search nodes:
        * incoming: 9200, 9300, 22
        * outgoing: 80, 443, 9300
    * load balancer node:
        * incoming: 80, 443, 22
        * outgoing: 80, 443, 8080
        
Configure administrator access restrictions 
*******************************************

Ideally all nodes should be protected via 2-factor authentication. You can do that by executing the following setup-2fa.sh script:

.. code:: text

    #!/bin/sh
    # Execute manually AFTER the host has been setup
    # Based on https://aws.amazon.com/blogs/startups/securing-ssh-to-amazon-ec2-linux-hosts/
    sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    sudo yum -y install google-authenticator
    # Execute this line for each user manually, before they can login. Use > sudo su <user> before that
    google-authenticator --time-based --disallow-reuse --force --rate-time=30 --rate-limit=3 --window-size=8
    sudo echo "auth required pam_google_authenticator.so" >> /etc/pam.d/sshd
    sudo sed -i -- 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
    sudo sed -i -- 's/auth       substack     password-auth/#auth       substack     password-auth/g' /etc/pam.d/sshd
    sudo echo "\nAuthenticationMethods publickey,keyboard-interactive" >> /etc/ssh/sshd_config

    sudo service sshd restart


Track administrative access through a custom PAM 
************************************************

We provide a custom PAM which you can get by `following the instructions here <https://github.com/LogSentinel/logsentinel-PAM>`_. The PAM makes sure that each administrative access is pushed directly to either Ethereum or a qualified trust service provider. That way, in case an administrator tries to manipulate the logs that will not only be detected, but they won't be able to cover who they were. The PAM does some additional checks, e.g. if the network is not blocked, and does not let the administrator login if the log event can't be pushed properly.