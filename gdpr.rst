GDPR functionality
==================

The General Data Protection Regulation requires systems to be upgraded to follow certain rules. Some of the requirements can’t be handled by a drop-in solution, but some can. That’s why LogSentinel supports a number of features – a GDPR register and GDPR-specific logging endpoints.
 
The GDPR endpoints are under ``/api/log-gdpr/``. There you can log consent and all requests by data subjects in a way that you can prove to regulators the events that happened. More details can be found in the `API console <https://app.logsentinel.com/api>`_ .
 
We support a GDPR Article 30 register, where a company should enlist all its processing activities (what types of data about what types of data subjects it processes). What does this have to do with audit logs? Since the regulation requires processing of data to be authorized, and the integrity of the data to be guaranteed, audit logs can be mapped to a particular processing activity in the register. That for each processing activity you’ll be able to track the relevant actions. This is done simply by providing an extra GET parameter to the logging call – ``gdprCorrelationKey``. Each processing activity can be assigned a unique correlation key to make it match the audit log records.
 
Additionally, you can use the GDPR register (via the ``/gdpr`` API endpoints) to fetch information about processing activities in order to display it to users for the purpose of collecting their consent. It’s best to have the register and your website in sync, so that all consent-dependent activities are covered and the user explicitly agrees to each of them.
 
