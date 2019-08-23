Log Collector Integration
=========================
Integration with Fluentd
************************
`Fluentd quickstart <https://docs.fluentd.org/v1.0/articles/quickstart>`_

* install third party plugin `http <https://github.com/fluent-plugins-nursery/fluent-plugin-out-http>`_ (requires basic knowledge of ruby gems). `Info for fluentd custom plugins <https://docs.fluentd.org/v1.0/articles/plugin-development#installing-custom-plugins>`_ example configuration for the plugin to communicate with logsentinel

.. code:: text

 <source>
   @type tail
   path /opt/log.txt
   refresh_interval 10
   tag logsentinel.file
   <parse>
     @type regexp
     expression /(?<actorId>[^ ]*) (?<action>[^ ]*) (?<entityType>[^ ]*) (?<entityId>[^ ]*) (?<param1>[^ ]*)$/
   </parse>
 </source>
 
 <match logsentinel.**>
   @type http
   endpoint_url     https://api.logsentinel.com/api/log/<actorId>/<action>/<entityType>/<entityId>?param1=<param1>
   serializer    json
   custom_headers {"Application-Id": "b1fgt7a0-5rc5-11e8-8230-0db3d3bfb10d"}
   username <organizationId>
   password <secret>
 </match>
 

* ``<source>`` configuration is only for testing purposes. It shows how to use regex to format data properly  It gets lines from log file with path <path> every <refresh_interval> seconds and parses it with <expression> regex, so data can be extracted easy. This specific regex transforms: ``"actor1 action2 entityType3 entityId4 urlParam"``  ->  ``{"actorId":"actor1","action":"action2","entityType":"entityType3","entityId":"entityId4","param1":"urlParam"}``
* ``<match>`` config is with type http which is the plugin that is already installed.
* ``endpoint_url`` is Logsentinel API url. Path variables and url params can be extracted from input (properly parsed). Params in <> are replaced with their values. Nested params also can be used ( example: ``<data.id>`` extracts ``444`` from ``{"data" :{"id":444}}`` )
* ``custom_headers``, ``usernamd`` and ``password`` contain mandatory headers for authentication and authorization. Values of Application-Id, usernamd and password should be obtained from the API credentials page on your dashboard
Additional configuration params are available - see  `http plugin configuration options <https://github.com/fluent-plugins-nursery/fluent-plugin-out-http>`_ 



Integration with Logstash
*************************

* Logstash http plugin documentation: https://www.elastic.co/guide/en/logstash/current/plugins-outputs-http.html
* sample configuration for integration with logsentinel

logstash.conf

.. code:: text

 input {
     file {
         path => "/opt/log.txt"
         start_position => "beginning"
     }
 }
 filter {
     grok {
         match => { "message" => "actorId=%{WORD:actorId} action=%{WORD:action} entityType=%{WORD:entityType} entityId=%{WORD:entityId}" }
     }
 }
 output{
 
    http {
    format=>"json"
    http_method=>"post"
    url=>"https://api.logsentinel.com/api/log/%{[actorId]}/%{[action]}/%{[entityType]}/%{[entityId]}" 
    headers => ["Application-Id", "b1f8b7a0-5cc6-11e8-8230-0dr3d3brb12d"]
    headers => ["Authorization", "BasicYjFmNjQ2YTAtNWNjNS0xMWU4LTgyMrEtMGRiM1QzYmDiMTBkOmM0YjA4OWViMDg1MmJmNmI0ZGJhNjMwMTJmN2Y2Y2RjMjk3ZWY3ODg4NmRiM2E5YjViODhiNGUxZGZlMzZhOGM="]
 
     }
 }
 

grok filter parses mandatory fields from a sample log file in key=value format. This is just an example, you can use any logstash functionality you wish.

Authorization and Application-Id headers contain mandatory headers for authentication and authorization. Values of Application-Id and Authorization are just an example. Your organization real values must be provided. Authorization header consists of “Basic” string + base64_encode(<your organization id>:<your secret>)

Integration with Nxlog
**********************

* Nxlog http module documentation https://nxlog.co/documentation/nxlog-user-guide#om_http
* sample configuration for integration with logsentinel

nxlog.conf

.. code:: text

 <Input file>
     Module              im_file
     File                '/opt/log.txt' 
 </Input>
 
 <Output http>
     Module              om_http
     URL                 https://api.logsentinel.com
     ContentType application/json
         AddHeader   Authorization : BasicYjFmNjQ2YTAtNWNuNS0xMeU4LTgyMzAtMGRiM1QzYmZiMTBkOmM0YjA4OWViNDg1MmJ
         mNmI0ZGJhNjMwMTJmN2Y2Y2RjMjk3ZWY3ODg4NmRiM2E5YjViODhiNGUxZGZlMzZhOGM=
         AddHeader   Application-Id : b1f8b7a0-5cc5-11e8-8230-0db3d3bfb10d
     <Exec>
         $raw_event =~ /(\S+) (\S+) (\S+) (\S+)/ ;
         $actorId = $1;
         $action = $2;
         $entityType = $3;
         $entityId = $4;
         set_http_request_path('/api/log/'+ $actorId + '/' + $action +'/' + $entityType +'/' +$entityId);
     </Exec>
 </Output>
 

URL is Logsentinel API url (api.logsentinel.com)

\Authorization and Application-Id headers contain mandatory headers for authentication and authorization. Values of Application-Id and Authorization are just an example. Your organization real values must be provided. Authorization header consists of “Basic” string + base64_encode(<your organization id>:<your secret>)

Extracting data from logs here is just simple regex that reads 4 words from log file and fills the mandatory url params (actorId, action , entityType, entityId). You can use all Nxlog functionality to parse and transform your logs as you wish.

Note: Sending custom http headers is only available in Enterprise edition of Nxlog. This feature is mandatory for integration with Logsentinel.
