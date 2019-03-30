On-premise Security
===================

In the cloud version of SentinelTrails we take all necessary operational security measures. However, in on-premise deployments this becomes the responsibility of the customer. Here are a few recommendations for keeping the installation secure:

* Install a TLS certificates. The certificate generation process can be seen in `this tutorial <https://docs.oracle.com/cd/E19798-01/821-1841/gjrgy/>`_. The properties that need to be set in ``application.properties`` are:

``
server.ssl.enabled=true
server.ssl.key-store=/path/to/server.jks
server.ssl.key-store-type=JKS
server.ssl.key-store-password=secret
server.ssl.key-alias=server
server.ssl.key-password=secret
secure.headers=true
root.url=https://{your-root-url}
root.url.api=https://{{your-root-url}
``

* Configure network restrictions - the Sentinel Trails machines should only have the required ports opened. That includes: 
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
		
* Configure administrator access restrictions - ideally all nodes should be protected via 2-factor authentication

* Configure custom PAM ----- https://github.com/LogSentinel/logsentinel-PAM This is useful for example when a system administrator logs in an on-premise LogSentinel Trails machine. If they try to manipulate the logs that will be detected, but they may be able to cover who they were. Pushing an event directly, combined with