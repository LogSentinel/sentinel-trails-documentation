Configuring The LogSentinel Agent
=================================
When audit logs are concerned, there are many ways to generate and collect them. Ideally, audit logs are generated in code, depending on the business logic of each application and sent for secure storage to `SentinelTrails <https://logsentinel.com/sentinel-trails/>`_ .

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


1. Get the `latest release of logsentinel-agent.jar <https://github.com/LogSentinel/logsentinel-agent/releases/download/0.1/logsentinel-agent.jar>`_ and copy it to ``/var/logsentinel/logsentinel-agent.jar``
2. Get the `logsentinel-agent.conf <https://github.com/LogSentinel/logsentinel-agent/blob/master/scripts/logsentinel-agent.conf>`_ file and copy it to ``/var/logsentinel/logsentinel-agent.conf``
3. Add a ``logsentinel-agent.yaml`` in the same directory. The file should contain the configuration of the agent (see the next section)
4. Get the `setup-agent.sh <https://github.com/LogSentinel/logsentinel-agent/blob/master/scripts/setup-agent.sh>`_ script and run it (works on CentOS/RHEL; we’ll soon add a similar script for Debian-based distros)

This should start the agent and configure it to run automatically on startup. You can start and stop it via ``service logsentinel-agent start/stop``

Installing on Windows
+++++++++++++++++++++


1. Get the latest `Windows installer <https://s3-eu-west-1.amazonaws.com/logsentinel-public/logsentinel-agent-install.zip>`_ 
2. Extract it and run the ``install.bat`` (you need admin privileges)
3. Customize the ``logsentinel-agent.yaml`` file in the installation directory
4. Go to Services and start the LogSentinelAgent service

Note: if you are going to collect Windows event logs from other machines, you need a series of permissions configurations `described in detail here <https://techblog.bozho.net/remote-log-collection-on-windows/>`_

Configuring the agent
---------------------
Configuring the agent is done via a straightforward YAML file. All properties are `described in the documentation <https://github.com/LogSentinel/logsentinel-agent/blob/master/configuration.md>`_ . Below is a sample setup that listens to a Windows log as well as a MS SQL Audit trail:

.. code:: text

	applicationId: ba2f0780-5424-11e8-b88d-6a2c1b6625c8
	organizationId: ba2cdc90-5424-11e8-b88d-6a2c1b6625c8
	secret: d8b63c3d82a6ded56b015a3b8617bf376b6aa6c181021abd0d37e5c5ac9941a1

	# BUSINESS_LOGIC_ENTRY, DATABASE_QUERY, SYSTEM_EVENT
	entryType: BUSINESS_LOGIC_ENTRY

	logsentinelBaseUrl: https://api.logsentinel.com

	includeMacAddress: false
	includeLocalIp: false
	timestampInitialUseCurrent: true

	windowsEventLogAgent:
		- sendLogsRate: 30000
		  sourceTypes: 
			- Application
			- Security
	   
	mssqlAuditLogAgent:
		- dbcConnectionString: jdbc:sqlserver://localhost:1434;integratedSecurity=true
		  sendLogsRate: 30000
		  mssqlLogsPath: c:\logs\mssqltrail\
	

Conclusion
----------
The logsentinel-agent can be installed on any machine and will forward any of the supported log records to SentinelTrails. This allows for integrating SentinelTrails into any kind of organization, regardless of whether it relies on legacy systems or is building new ones. The agent can also `work alongside existing log collection tools <https://logsentinel.com/log-collectors-logsentinel/>`_ , so that you forward the most business critical events for secure storage and leave the rest of the logs in the existing, less secure solution.

Flexibility and integration-friendliness are key elements of an information security solution and we are happy to offer such a tool, bundled with support for our enterprise customers.
