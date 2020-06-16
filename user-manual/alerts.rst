Alerts
======

Alerts are an important part of audit log aggregation as they allow defining anomalous scenarios to be alerted for. Alert configuration is available from the "Alerts" menu. We support three types of alerts:

* **Behavior rules** - specify rules for anomalous behavior over a period of time
* **Correlation rules** - specify specific sequences of events
* **Machine learning (experimental)** - use unsupervised machine learning to detect anomalous behavior

There are predefind sample rules on each of the rule-configuration screens. They set some reasonable defaults for testing purposes.

Each rule can be enabled or disabled. Only enabled rules are executed.

Alert destinations
------------------

Alert destinations have to be specified first in order to be able to receive alerts. Alerts can be received via email, Telegram or URL. For setting up telegram, :doc:`check this page </user-manual/telegram>`. 

URLs can be used to call the notification API of a centralized notiifcation system, or to trigger or block certain processes in other applications.

Working hours
-------------

An important configuration for alert rules are the business hours for the organization. They can be configured from the dedicated menu "Alerts/Working Hours" menu. 

Working hours can be set globally for the organization, or individually per application (data source). This is needed as some applications may be used by employees working in multiple shifts or with clients in different timezones and have shifted working hours.

Public holidays can be manually added. That's optional, but some alerts may be triggered on holidays due to an unexpected reduction of activity.

Rules can be defined to work either inside or outside working hours, as behavior differs significantly. Rules can be configured to work throughout the whole day, regardless of business hours as well.

Behavior rules
--------------

These rules are defined via a wizard and can be expressed as follows: trigger the alert if the number of actions in the last 5 minutes is bigger than 2 standard deviations compared to the last 2 hours; apply this only within working hours.

Supported comparison methods include standard deviation, mean value of a fixed constant. Alerts can be triggered if the observed value is eiher above, or below, or both above or below the expected normal value.

By default the number of log entries is taken, but for numeric fields a sum or average can also be calculated. Numeric fields can be specified as GET parameters to log queries and are stored under ``additionalParams.*`. They do not have to be configured, as they are automatically extracted if a given additionl param is a number.

Typical examples include:
* Missing logs - look for at least 1 log entry over a configured period of time. If there's less than one (using FIXED comparison), trigger an alert
* Anomalous log count - send an alert if there's a difference of more than 3 standard deviations from the observed base. The observed base is the "search period" in the wizard, whereas the period to compare is called "aggregation period"
* Anomalous log count by a particular actor - same as above, but select ``actorId`` as a "Group by field". That way entries are grouped by actorId in order to compare the activity.

Correlation rules
-----------------

These rules allow for specifying a chain of events which trigger the alert. The rule is executed every X minutes (configurable) and peforms its analysis over the previous Y minute or hours (configurable).

A rule starts with an initial action and/or entity. If those are met, a list of subsequent events can be specified. If all criteria are met, the alert is triggered. Each subsequent event has the following properties:

* Action - the action of the particular event
* Count - "less than" or "more than" the specified amount of events
* Time frame - the timeframe in which the specified number of evnets (matching the specified action) should have occurred

An example correlation alert can be: "If you encounter a DELETE action outside working hours, trigger an alert". Or "After an UPDATE action, trigger an alert if there are more than 10 other update actions on the same entity within the next 2 minutes by the same actor".

For numeric parameters sums can be used as well. A sum action is exected at the end to addditionally confirm whether the given alert should be fired or not. A sum action is a specific type of event that contains the numeric param in sum path (JSON Path or XPath). We calculate the sum of some numeric values on all previous entries of the sum type and compare it with the current entry. If current entry value is larger than percentage of the sum then alert is triggered. This is done for every entry in the time frame.

Machine learning
----------------

Machine learning anomaly detection is configured per-organization and runs in the background. If anomalous behaviour is detected, users are notified. 

In order to enable machine learning anomaly detection, go to the "Alerts/Anomaly detection" page and select for which applications it should be enabled. There are a few parameters to configure:

* Enable anomaly detection - by default ML anomaly detection is disabled for all applications. Once enabled, it needs a period of at least 10 days of steady flow of data to get the model trained. Make sure you enable it after data ingestion has started.
* Use original timestamp - this indicates which timestamp should be used by the algorithm. By default it uses the timestamp when the event was received by SentinelTrails; however in some cases an ``originalEventTimestamp`` can be specified (depending on the sending/collection logic that may be more accurate)
* Use entry fields entityId and entityType for anomaly detection - some applications are able to send ``entityType`` and ``entityId`` for each log entry. This is an important feature for the machine learning model, if available. For example, it's  when the ``entityType`` and ``entityId`` are related to database records, where ``entityType`` is the table name and ``entityId`` is the primary key of the table.

The algorithm used in Isolation forest, which is perfectly suited for datasets that are expected to have very few anomalies

API management
--------------

Alerts can be managed through API calls in addition to the UI wizard. `The API reference is available here <https://api.logsentinel.com/api#/Alerts>`_