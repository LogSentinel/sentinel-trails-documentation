Configuring an Organization
===========================

The organization is the main entity for each account. It has global configurations, some of which can be overridden per application. The configuration options include:

* **General and billing information** - Name, address, representative, VAT number
* **GDPR information** - GDPR-required information about a company (applicable to the cloud setup and in some cases to the on-premise setup)
* **TSA details** - in case the organization chooses to timestamp groups of log entries with a qualified timestamp, the TSA (Time Stamping Authority) can be configured here. In addition to the credentials, the inerval at which the timestamping is performed is configured here. It can be incrased or decreased to adjust the cost of the external service.
* **IP whitelist** - a whitelist of IPs that are allowed to send logs. Can be overridden by each application
* **Retention period** - how long should logs be kept operational (they are then backed up on disk or S3 after being cleaned up from the operational data stores)
* **Password validation regex** - allows imposing per-organization password rules for users. A message to be displayed to users can also be configured
* **Report periods** - specifies when should automated reports be sent to the report recipients
* **Actor ids correlation** - allows specifying a comma-separated table of user IDs across multiple systems. Each line holds all the IDs of a single user across multiple systems. This information is used when searching via the dashboard, as additional user IDs are automatically suggested and appended to the search query.

Actor ids correlation
=====================
An example actorId correlation looks like this:

.. code:: text

      user1IdInSystem1,user1IdInSystem2,user1IdInSystem3
      user2IdInSystem1,user2IdInSystem2,user2IdInSystem3
      user3IdInSystem1,user3IdInSystem2,user3IdInSystem3
      user4IdInSystem1,user4IdInSystem2,user4IdInSystem3