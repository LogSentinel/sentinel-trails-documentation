Flexible Log Collection: Configuring The LogSentinel Agent
==========================================================
Posted on `December 11, 2018February 13, 2019 <https://logsentinel.com/flexible-log-collection-configuring-the-logsentinel-agent/>`_ by `admin <https://logsentinel.com/author/admin/>`_ When audit logs are concerned, there are many ways to generate and collect them. Ideally, audit logs are generated in code, depending on the business logic of each application and sent for secure storage to another service, like `SentinelTrails <https://logsentinel.com/sentinel-trails/>`_ .

However, refactoring a system to include dedicated audit log functionality may not be feasible, as there are a lot of legacy systems out there. This is why we have built the `LogSentinel Agent <https://github.com/LogSentinel/logsentinel-agent>`_ , an open-source tool that can be installed on any machine in order to collect logs that are relevant for forensic, audit and compliance purposes.

Supported log sources
---------------------
The agent supports many types of log sources:



* Log file – reading a simple text log file and sending each line as a separate log event. You can configure a file to be watched and “tailed” and log records will be sent to SentinelTrails.
* Relational database – some applications already have some sort of audit trail stored in the database and only need to forward it to a more secure and tamper-protected service. You can configure multiple queries and map columns to particular audit log entry fields (e.g. actor, action, entityType). Queries rely on a datetime column to only fetch fresh records.
* Database log – a smarter log file extension – it looks for SQL queries in a database log file, parses them and sends each query as a separate log message
* Access log – web servers usually generate access logs and they may be seen as audit logs in some circumstances. The agent supports the standard way (supported by Apache, Nginx, etc.) to provide access log format.
* MS SQL Server audit log – SQL Server has a special audit log functionality. It is enabled
*  `as described here <https://github.com/LogSentinel/logsentinel-agent/blob/master/MS_SQL_README.md>`_ 
* and then it can be queried by the agent in order to obtain and send each audit log entry to the SentienlTrails service
* Linux audit log – Linux distributions have a standard audit log file (/var/log/audit/audit.log). The agent supports the specific format of its contents, extracts the relevant fields and sends the events
* Directory – the agent can listen to directory changes (e.g. adding or removing files)
* Windows event log – windows applications normally use the windows event log and so the agent can tap into that log and forward its contents. the agent is customizable to include or exclude certain sources
* AxonDB – AxonDB is a special event-based database. Since every modification there is an event, the Agent can listen to all these events and convert them to audit log entries to be sent to SentinelTrails

Installing the agent
--------------------
Installing the agent is simple. It depends on whether you install it on a Linux or Windows machine, but it usually involves a short script or a one-click installer.

Installing on Linux
+++++++++++++++++++


1. Get the
2.  `latest release of logsentinel-agent.jar <https://github.com/LogSentinel/logsentinel-agent/releases/download/0.1/logsentinel-agent.jar>`_ 
3. and put in in
4. .. code:: text

 /var/logsentinel/logsentinel-agent.jar


5. Get the
6.  `logsentinel-agent.conf <https://github.com/LogSentinel/logsentinel-agent/blob/master/scripts/logsentinel-agent.conf>`_ 
7. file and put in in
8. .. code:: text

 /var/logsentinel/logsentinel-agent.conf


9. Add a logsentinel-agent.yaml in the same directory. The file should contain the configuration of the agent (see the next section)
10. Get the
11.  `setup-agent.sh <https://github.com/LogSentinel/logsentinel-agent/blob/master/scripts/setup-agent.sh>`_ 
12. script and run it (works on CentOS/RHEL; we’ll soon add a similar script for Debian-based distros)

This should start the agent and configure it to run automatically on startup. You can start and stop it via.. code:: text

 service logsentinel-agent start/stop

Installing on Windows
+++++++++++++++++++++


1. Get the latest
2.  `Windows installer <https://s3-eu-west-1.amazonaws.com/logsentinel-public/logsentinel-agent-install.zip>`_ 
3. Extract it and run the install.bat (you need admin privileges)
4. Customize the logsentinel-agent.yaml file in the installation directory
5. Go to Services and start the LogSentinelAgent service

Configuring the agent
---------------------
Configuring the agent is done via a straightforward YAML file. All properties are `described in the documentation <https://github.com/LogSentinel/logsentinel-agent>`_ . Below is a sample setup that listens to a Windows log as well as a MS SQL Audit trail:

applicationId: ba2f0780-5424-11e8-b88d-6a2c1b6625c8
organizationId: ba2cdc90-5424-11e8-b88d-6a2c1b6625c8
secret: d8b63c3d82a6ded56b015a3b8617bf376b6aa6c181021abd0d37e5c5ac9941a1

# BUSINESS_LOGIC_ENTRY, DATABASE_QUERY, SYSTEM_EVENT
entryType: BUSINESS_LOGIC_ENTRY

# FILE, RELATIONAL_DATABASE, DATABASE_LOG, DIRECTORY, ACCESS_LOG, MSSQL_AUDIT_LOG, LINUX_AUDIT_LOG, AXON_DB, WINDOWS_EVENT_LOG
targetTypes:
  - MSSQL_AUDIT_LOG
  - WINDOWS_EVENT_LOG

logsentinelBaseUrl: https://api.logsentinel.com

includeMacAddress: false
includeLocalIp: false
timestampInitialUseCurrent: true

windowsEventLogAgent:
    sendLogsRate: 30000
    sourceTypes: 
        - Application
        - Security
   
mssqlAuditLogAgent:
    jdbcConnectionString: jdbc:sqlserver://localhost:1434;integratedSecurity=true
    sendLogsRate: 30000
    mssqlLogsPath: c:\logs\mssqltrail\
Conclusion
----------
The logsentinel-agent can be installed on any machine and will forward any of the supported log records to SentinelTrails. This allows for integrating SentinelTrails into any kind of organization, regardless of whether it relies on legacy systems or is building new ones. The agent can also `work alongside existing log collection tools <https://logsentinel.com/log-collectors-logsentinel/>`_ , so that you forward the most business critical events for secure storage and leave the rest of the logs in the existing, less secure solution.

Flexibility and integration-friendliness are key elements of an information security solution and we are happy to offer such a tool, bundled with support for our enterprise customers.

This entry was posted in `log collection <https://logsentinel.com/category/log-collection/>`_ and tagged `log aggregation tools <https://logsentinel.com/tag/log-aggregation-tools/>`_ , `log collectors <https://logsentinel.com/tag/log-collectors/>`_ . Bookmark the `permalink <https://logsentinel.com/flexible-log-collection-configuring-the-logsentinel-agent/>`_ . `Edit <https://logsentinel.com/wp-admin/post.php?post=1835&action=edit>`_ Post navigation
***************
 `←How LogSentinel Complements Log Collectors <https://logsentinel.com/log-collectors-logsentinel/>`_  `Releasing SentinelDB, the Privacy By Design Database→ <https://logsentinel.com/releasing-sentineldb-the-privacy-by-design-database/>`_ 