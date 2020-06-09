Installation via VM image
=========================

SentinelTrails can be provided as a virtual appliance in Open Virtualization Format. The provided appliance is pre-configured and should be deployed with the virtualization platform used by the customer. Then the image needs configuring a few parameters:

* If DCHCP is not available, configure IPs in ``/etc/sysconfig/network-scripts/ifcfg-xxx`` - ``BOOTPROTO=static``, ``IPADDRESS=<ip>``, ``GATEWAY=<gateway>``, ``NETMASK=<netmask>``. Then run ``service network restart``.
* ``root.url`` in ``/var/logsentinel/app.properties`` should be set to the IP address or internal domain name of the machine. 
* ``spring.mail.*`` properties if email notifications are required. The application can work without that for test purposes.
* ``server.ssl.*`` properties (`see reference <https://docs.spring.io/spring-boot/docs/1.2.0.M1/reference/html/howto-embedded-servlet-containers.html#howto-configure-ssl>`_) should be configured in case the service is used 
* ``secure.headers`` should be set to ``true`` if TLS is used (either configured via ``server.ssl`` properties or at a load balancer/reverse proxy put in front of the application
* ``jwt.secret`` and ``hmac.key`` should be set to new values for non-test environments
* After all changes are made, the SentinelTrails service should be restarted by running ``service logsentinel restart``
