Sentinel Trails Dashboard
=========================

The dashboard is the main place for day-to-day usage of the product. It has several components:

*  **Statistics** - general statistics about your organization as well as charts of certain attributes
*  **Main timeline chart**  - The main timeline of log entries
*  **Reports**  - Reporting functionality in various formats
*  **Custom charts**  - Additional custom, user-defined charts
*  **Exports**  - Exporting functionality in various formats
*  **Query bar**  - A way to query the logs with detailed search queries
*  **Entries list**  - A list of all entries that satisfy the current search criteria and period
*  **Time aggregations**  - Aggregated statistics over smaller time periods
*  **Numeric aggregations**  - Perform aggregation functions on numeric parameters
*  **Relationship graph**  - Visual representation of the relationship between the entries in the current result


Statistics
----------

The total number of log entries, distinct actors, actions and entities as well as the alerts recently triggered. Note that statistics is cached, so it's not real-time.

The statistics row also contains information about background log verification - when was the last time log verification was performed and what's he status. The green icon indicates that the log is intact. In rare cases of someone trying to manipulate the log, the icon turns red.

A little further down are the statistics charts which display the top actors, actions, entities and applications by number of log entries. The charts can be expanded to show the actual values. There's also a link to the rankings pages to display the top actors/actions/entries for a selected period.

Both the statistics row and the charts are updated when the period of the main timeline is changed from the timepicker on the right.

Main timeline chart
-------------------

The main timeline charts show the log activity over a selected period of time. The default period is three days. 

The period can be changed in two ways: via the timepicker on the right or via slicing the chart with the mouse. Changing the period not only changes the visualized timeline but also the search period in the entries list below. That way you can visually inspect a certain period of time for the current search query. The currently sliced period can be reset via a button that appears ontop of the chart.

Additionally, you an click on a each point on the chart (points are placed at hourly intervals) which will select a 1-hour period starting from that point and apply it to the current search.


Custom charts
-------------

Custom charts that are defined in the "Charts" menu appears below the default statistics charts. Charts can be defined as aggregations on a specific field with or without an additional query. For example a chart can display the top actors that have performed one of particular set of actions.

Custom charts can be bar, pie, line, doughnut and can have custom or random colors defined. Custom charts are useful for displaying business-specific statistics.

Reports
-------

Above the main timeline chart are reporting buttons. A high-level report for the currently selected period can be exported in either PDF or XLSX formats. Reports contain an overview of the log activity over the selected period of time. 

Additionally, reports are generated periodically and sent via email. The reporting periods are defined from the "Charts" page and can be "daily", "weekly", "monthly" and "yearly". Multiple report recipients can also be configured.

Exports
-------

The currently displayed log entries can be exported to various formats - CSV, XLSX, TXT. The export is done from the buttons right above the entries list.

Query bar
---------

The query bar is the way to define detailed queries to search through the logs. The queries are in the format ``field:value``, where field is either one of the predefined fields or ``additionalParams`` specified in the log requests. The ``details`` field allows full-text search using ``details:*value*``. Multiple query parts can be used with ``AND`` or ``OR``

There are buttons to help with query generation - each button appends additional part of the query. The additional params can be found in the "params" dropdown.

Queries are run over a search period. The period is defined from the timepicker on the right. This timepicker is usually in sync with the one for the main timeline chart, but does not have to be, i.e. you can display entries over 3 days and search only within the past 10 hours.

Queries can be performed over all applications (i.e. sources of logs) or to one or more of them. You can select them via a dropdown which appends the ``applicationId`` parameter to the query.

Common queries can be saved from the dropdown next to the search button and then executed without the need to retype them.

Finally, if log details have been encrypted before sending them, you can perform search in the encryped data by specifying the symmetric (AES, base64-encoded) key in the encryption key field on the right. The key is never transmitted to the server so the decryption are  performed entirely client-side on the results that are fetched based on the the encrypted keywords.

Entries list
------------

The entries list contains all entries that satisfy the current search query. If there is no search query, the latest processed entries are displayed. Note that there might be a slight delay between sending the logs and seeing them in the list, as forming the cryptographic blocks does not happen immediately. This does not mean that the logs are not securely stored in the meantime.

The list has the following columns:

* **Actor** - the column contains the actor for the current entry. If there is an ``actorDisplayName`` specified, it is shown instead of the actorId, which is assumed to not be human-readable. If there is no ``actorDisplayName``, the ``actorId`` is displayed. Additionally, if any roles have been specifed for the actor, they are displayed in parentheses
* **Action** - the action that has been performed. In case the actor is a non-person (e.g. a system actor), the action is the event that occurred.
* **Entity** - the entity about which the log is. The semantics depends on the type of logs, but usually an entity has an ID as well. Both are displayed in this column as "<entityType> #<entityId>". In case a database is monitored, for example, this would be "<table>#<primary key>".
* **Details** - this is the body of the log and can contain anything. It is trimmed in the table so clicking each cell is required to show the entire content. You can define a formatting function for standard formats (JSON and XML) that would allow displaying the values as a table rather than as a raw message
* **Level** - the log level, if one has been specified. Log levels can be: TRACE, DEBUG, INFO, WARN, ERROR, CRITICAL, FATAL. 
* **Timestamp** - when did the even occur. The time is shown in the current user timezone. The clock icon opens a dialog that displays details about the cryptographic timestamp over a block of events. The base64-encoded token is in a RFC3161-specified format.
* **Params** - any ``additionalParams`` that were specified as part of the requests
* **Hash** - opens a dialog where a number of details are shown: the individual hash of the entry, the hash as part of the hash chain, the entry id as well as the previous entry id (previous in the hash chain). These values can be used for log verification, e.g. for obtaining merkle proofs.

From the list you can generate simple queries by clicking on one of the fields (actorId, action, entity).

Time aggregations
-----------------

Time aggregations show charts spit by hour, day, week, month or year, as well as the number of entries for the last periods of the selected type. Charts can be show for all applications or for a paricular application.

Numeric aggregations
--------------------

Certain parameters (``additionalParams``) can be numeric. If there are such parameters, users can perform aggregations on them. The supported aggregations are: average, sum, min and max.

Relationship graph
------------------

The relationship graph is a visualization aid for the current search results. The graph displays the relationships between actors, actions and entities. It can be useful for analyzing actor behavior.