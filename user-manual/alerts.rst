Alerts
======

Alerts are an important part of audit log aggregation as they allow defining anomalous scenarios to be alerted for. We support three types of alerts:

* **Behavior rules** - specify rules for anomalous behavior over a period of time
* **Correlation rules** - specify specific sequences of events
* **Machine learning (experimental)** - use unsupervised machine learning to detect anomalous behavior


Alert destinations
------------------

Alert destinations have to be specified first in order to be able to receive alerts. Alerts can be received via email, Telegram or URL. For setting up telegram, :doc:`check this page </user-manual/telegram>`. URLs can be used to trigger or block certain processes in other applications.

Working hours
-------------

An important configuration for alert rules are the business hours for the organization. They can be configured from the dedicated menu. Many rules will be defined to work either inside or outside working hours, as behavior differs significantly. 

Behavior rules
--------------

These rules are defined via a wizard and can be expressed as follows: trigger the alert if the number of actions in the last 5 minutes is bigger than 2 standard deviations compared to the last 2 hours; apply this only within working hours.

Supported comparison methods include standard deviation, mean value of a fixed constant. Alerts can be triggered if the observed value is eiher above, or below, or both above or below the expected normal value.

By default the number of log entries is taken, but for numeric fields a sum or average can also be calculated.

Correlation rules
-----------------

These rules allow for specifying a chain of events which trigger the alert. The rule is executed every X minutes (configurable) and peforms its analysis over the previous X minute or hours.

A rule starts with an initial action and/or entity. If those are met, a list of subsequent events can be specified. If all criteria are met, the alert is triggered.

An example correlation alert can be: "If you encounter a DELETE action outside working hours, trigger an alert". Or "After an UPDATE action, trigger an alert if there are more than 10 other update actions on the same entity within the next 2 minutes".

Machine learning
----------------

Machine learning anomaly detection is configured per-organization and runs in the background. If anomalous behaviour is detected, users are notified.