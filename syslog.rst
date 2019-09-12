Syslog Integration
==================
In order to integrate with syslog, there are several options. 

* We support syslog over TCP (plaintext and over TLS) as well as over UDP. 
* We support both RFC 3164 and RFC 5424. In addition to that, we support SonicWall extended syslog messages

Below is a list of endpoints and ports for each supported variant. We recommend TCP over TLS for most installations. However, some setups lack the needed flexibility, so fallback to plaintext TCP or UDP may be needed. In such cases there's an option for a VPN tunnel (for enterprise customers), or a more complicated internal setup with a intermediate syslog forwarder.

* syslog.logsentinel.com:514 - plaintext TCP
* syslog.logsentinel.com:515 - TCP over TLS
* syslogudp.logsentinel.com:1516 - plaintext UDP

We have a syslog configuration script which you can download and run `configure-syslog.sh <https://d381qa7mgybj77.cloudfront.net/wp-content/uploads/2018/12/configure-syslog.sh>`_. It configures a syslog template that allows authenticating against LogSentinel Trails. The important line for authentication is this:

.. code:: text

    \$template LogSentinelFormat,\"<%pri%>%protocol-version% %timestamp:::date-rfc3339% %HOSTNAME% %app-name% %procid% %msgid% [logsentinel@$LOGSENTINEL_DISTRIBUTION_ID organizationId=\\\"$LOGSENTINEL_ORG_ID\\\" secret=\\\"$LOGSENTINEL_ORG_SECRET\\\" applicationId=\\\"$LOGSENTINEL_APP_ID\\\" tag=\\\"RsyslogTLS\\\"] %msg%\n\"


SonicWall and other devices can be authenticated using a syslogId that is configured per device. You can obtain the syslogId (which effectively comprises of an ID and secret) from the API credentials page.
