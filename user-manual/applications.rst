Configuring Applications
========================

An application represents a log source. It can be a CRM, ERP, Active Directory, Exchange, core banking system, or anything else that qualifies as a distinct log source.

Applications are created and configured from the "Applications" menu. Each application has the following properties and features:

* **ID** - this is auto-generated ID that is used to designate logs for a specific application through the ``Application-Id`` header (or the respective configuration in the agent)
* **Name** - a human-readable name for the application, e.g. "ERP", "CRM", "Active Directory"
* **Store hashes in Ethereum** - if the subscription plan allows it (or if an Ethereum private key is configured for on-premise installations), this turns on periodic anchoring of latest hashes and merkle roots to Ethereum. There's an option to manually push the latest ones in addition to the periodic triggers
* **Export chain** - allows exporting the whole chain in JSON or protobuf formats.
* **Storing personal data** - this is an indicator that the application may contain personal data.
* **Verification periods** - how often should full and partial background verifications run. If you have a lot of data it's recommended that full verifications is done rarely, as it puts significant load on the system (in case of on-premise setups).
* **Warn log level** - whether admins should be warned in case a log level above a certain threshold is received
* **JavaScript body transformation** - allows users to specify javascript that prettifies the log details for easier reading. By default JSON-to-table and XML-to-table scripts are supported
* **Displayed details fields** - this allows configuring a comma-separated list of fields that should be displayed on the dashboard for JSON bodies. This is useful in case of noisy log messages that contain less useful information. If you specify a whitelist of fields, all others is stored and searchable, but not displayed by default.
* **List of recipients of verification reports** - a list of emails to receive verification reports for this application
* **List of recipients of log level issues** - a list of emails to receive reports when a log entry above the threshold log level is received
* **IP Whitelist** - a whitelist of IPs that are allowed to send logs for this application
* **Hash recipients** - latest hashes are sent to these emails. Useful to distribute hashes that can later be used for log verifications.
* **Public keys** - a list of public keys used to verify log signatures if such are supplied by the client. Clients can sign each log entry with a private key and provide the public one(s) for verification.
* **Paths** - upon receiving a log enry, the system can extract relevant information from the body. This is useful in cases when the user does not have full control over what gets sent but would like to have actor, action and other fields extracted. Supported extraction schemes are XPath and JSON Path.
* **Log level regexes** - each log entry can be classified as error, critical or fatal in case its body matches the specified regex.
* **Opendata** - on-premise installations have support for opendata. This is usually applicable to public sector customers that have legal transparency obligations. It allows making all data for an application publicly accessible and automatically eported as JSON. The options here include a whitelist of regexes which designate which entries should be displayed unanonymized, particular fields (JSON Path or XPath) to be anonymized in the body and regexes for anonymizing content.