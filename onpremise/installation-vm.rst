On-premise Installation via VM image(s)
======================================

Test / PoC environment
**********************

SentinelTrails can be provided as a virtual appliance in Open Virtualization Format. The provided appliance is pre-configured and should be deployed with the virtualization platform used by the customer. Then the image needs configuring a few parameters:

* If DCHCP is not available, configure IPs in ``/etc/sysconfig/network-scripts/ifcfg-xxx`` - ``BOOTPROTO=static``, ``IPADDRESS=<ip>``, ``GATEWAY=<gateway>``, ``NETMASK=<netmask>``. Then run ``service network restart``.
* ``root.url`` in ``/var/logsentinel/app.properties`` should be set to the IP address or internal domain name of the machine. 
* ``spring.mail.*`` properties if email notifications are required. The application can work without that for test purposes.
* ``server.ssl.*`` properties (`see reference <https://docs.spring.io/spring-boot/docs/1.2.0.M1/reference/html/howto-embedded-servlet-containers.html#howto-configure-ssl>`_) should be configured in case the service is used 
* ``secure.headers`` should be set to ``true`` if TLS is used (either configured via ``server.ssl`` properties or at a load balancer/reverse proxy put in front of the application
* ``jwt.secret`` and ``hmac.key`` should be set to new values for non-test environments
* After all changes are made, the SentinelTrails service should be restarted by running ``service logsentinel restart``


Production environment
**********************

In order to run successfully in production, SentinelTrails needs to be run in a cluster. The typical cluster setup includes the following nodes, all of which are provided in OVF:

* 2 applications nodes - all the configuration options for a Test / PoC environment apply here as well, plus additional configurations below
* 2 database nodes (cassandra) - cassandra nodes require minimal configuration to run in a cluster
* 3 elasticsearch nodes (optionally, 2 ES nodes can be configured, but that risks creating a split-brain scenario in case of failure of connectivity between the nodes)
* load balancer node(s) - running nginx as a reverse proxy that forwards requets to the application nodes


Application node configuration
------------------------------

In addition to the base properties above, the following needs to be set:

* ``hazelcast.nodes`` - comma-separated list of the static IP addresses (or domain names) of the application nodes
* ``cassandra.hosts`` - comma-separated list of the static IP addresses (or domain names) of the database nodes
* ``elasticsearch.url`` - comma-separated list of the URLs to the elasticsearch nodes. If DNS round-robin us used, this can be a single URL
* ``etherscan.key`` - in case hashes are to be pushed to Ethereum, an Etherscan key should be configured
* ``storage.local.root`` - we recommend setting up NFS shared directories and configuring them as a shared storage locations

Cassandra configuration
-----------------------

* Cassandra needs the IP addresses of seed node(s) confiugred as in ``/etc/cassandra/cassandra.yaml``
* JVM options should be configured in ``/etc/cassandra/conf/jvm.options`` depending on the available memory

We only work with Cassandra 3.11 (the latest version)

Elasticsearch configuration
---------------------------

Standard elasticsearch cluster configuration is required `as described here <https://www.elastic.co/guide/en/elasticsearch/reference/current/add-elasticsearch-nodes.html>`_.

Elasticsearch 6.x and 7.x are supported.

Supported Operating Systems
---------------------------

We support virtual machines with the following operating systems:

* CentOS 7
* CentOS 8
* Ubuntu 20.04
* Ubuntu 19.10
* Ubuntu 18.04
* Amazon Linux 2


Backup
------

Backup is ideally performed on the underlying storage. If needed, Cassandra backup tooling (Priam) can be configured by our team.

Monitoring
----------

Different organizations have different tools available for monitoring certain aspects of their infrastructure. We don't impose any tool and will assist the organization in creating the necessary rules with their tooling, including installing an agent, if required. Typical monitoring components are:

* **Free disk space** - while you can configure retention periods for the data and it will automatically get deleted from the database and cassandra (and moved to the configured NFS storage), it's important to be alerted if disk space is running out on either of the nodes.
* **CPU usage** - prologned CPU usage over 70% may indicate that the existing CPUs are not sufficient to handle the load and a new or more preformant application node is needed
* **Free memory** - all nodes are heavily RAM-dependent, so the current free memory should be monitored
* **Healthchecks** - lightweight HTTP healthchecks can be performed against the ``/login`` URL. Additionally, an authenticated `/healthcheck`` is exposed for more detailed healthchecks, including database and elasticserach status