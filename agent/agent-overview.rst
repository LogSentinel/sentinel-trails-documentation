LogSentinel Agent Overview
==========================
The `Sentinel Trails listening agent is an open source component <https://github.com/LogSentinel/logsentinel-agent>`_ that gets installed on target machines to listen to a configured set of log sources. It can be installed on Linux and Windows and supports the following types of sources:


*  **Log files**  an arbitrary text file can be collected and sent, line by line, to the Sentinel Trails service. These typically application logs and logs by any other non-standard software
*  **Database logs files**  - if database query logs are enabled, the agent listens to newly issued queries and sends them to the Sentinel Trails service
*  **Database tables**  - if you store audit trail inside relational database tables, you can configure queries that periodically fetch new entries and send them to the Sentinel Trails service
*  **MS SQL audit trail**  – if MS SQL audit trail is enabled, the agent can be configured to listen to it and forward the events
*  **MS SQL change tracking**  – if MS SQL change tracking is enabled, the agent can be configured to listen to it and forward the change events
*  **Access logs**  – the standard web server access log files can be parsed and sent to Sentinel Trails
*  **Linux audit log**  – the native linux audit log file can be tailed and forwarded to Sentinel Trails
*  **Windows event logs** – the Windows event logs (including all categories – Application, Security, System) can be read continuously as sent to Sentinel Trails
*  **Directory changes** - any changes in a directory (new files, removal of files, modification of files) can be tracked using this agent configuration.
*  **AxonDB logs**  – AxonDB is a special type of non-relational database. We support its custom log format.

Any combination of the above can be configured. Note that for some target types the agent can be installed on a different machine than the actual log source. For example, in case of database tables or database audit trail, the agent can be installed on another machine that connects to the database server via a database connection string and credentials.

`Full configuration details can be seen here <https://github.com/LogSentinel/logsentinel-agent/blob/master/configuration.md>`_ .

All communication between the agent and the Sentinel Trails service is **encrypted** .

Configuration and installation is done through scripts provided by us and you :doc:`can follow the steps here </agent/configuring-agent>`:


Below is an overview of how the agent fits into the architecture:

.. image:: https://d381qa7mgybj77.cloudfront.net/wp-content/uploads/2018/11/Sentinel_trails_overview-708x1024.png
   :height: 926
   :width: 640
