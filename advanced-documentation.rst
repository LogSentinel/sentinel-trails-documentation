Advanced Documentation
======================
Documentation
*************
First make sure you’ve read the `getting started guide <https://logsentinel.com/getting-started/>`_ and the `white paper <https://logsentinel.com/white-paper/>`_

In this section you’ll find a more detailed description of each of the endpoints and the related use-cases

General use cases
*****************
You would normally use LogSentinel as a way to securely store your business-level audit log entries. “Business level” means events that are important to the business logic – all CRUD operations, as well as operations like checking out a basket, completing a transaction, etc. Additionally, you can log authentication events, like registration, login, logout, auto-login and failed login.

That way you’ll get a searchable list of business logic events. Regular program logging, where information about the program flow is logged at different levels (error, info, debug), is a separate concern, as it is more useful for identifying problems with the program flow, rather than creating a log of what the users of the system did. The latter needs to have legal strength and protect again malicious internal actors, while the former is useful for engineers. In that sense, LogSentinel is more than a log aggregation tool.

General client setup
********************
Applications send events to LogSentinel using a RESTful API. That can be done using a native RESTful call mechanism (in dynamic languages it’s pretty straightforward) or via one of the `client libraries <https://logsentinel.com/sentinel-trails/documentation/libraries-plugins/>`_

When configuring the client, a few mandatory parameters must be specified (explained in the `Getting started guide <https://logsentinel.com/getting-started/>`_ ):



* ``ApplicationId`` – obtained from the “API credentials” section in the admin panel and passed with a ``Application-Id`` header
* ``OrganizationId`` – obtained from the “API credentials” section and passed, alongside with the Secret parameter, in ``Authorization`` header
* ``Secret`` – obtained again from the “API credentials”

You’d also have to make a choice whether you want to send the full details of events, their hashes, or an encrypted version of the details. The latter two would practically disable parts of the search functionality, but will mean that the log server does not have any privacy related details

Additional headers
******************


* ``Signature`` - the ``Signature`` contains a digital signature of the whole request body. The signature is an RSA “signature” using a locally (on your application end) generated key that the LogSentinel server does not have. The premise for LogSentinel is that even if a log server is compromised, modification of the records will be detectable. In the unlikely event of a LogSentinel server being compromised, an attacker could insert fake records, but if they don’t possess the private key to sign the records, it will be detectable upon inspection. The algorithm used for signing should be ``SHA256withRSA`` and the result should be Base64 encoded and set in the ``Signature`` header. You can configure your public key in the application configuration so that verification is performed automatically.
* ``Audit-Log-Entry-Type`` – the type of the audit log entry. Allowed values are ``BUSINESS_LOGIC_ENTRY``, ``DATABASE_QUERY``, ``SYSTEM_EVENT``, ``NETWORK_EVENT``, ``DOCUMENT``. The header is optional and the default value is ``BUSINESS_LOGIC_ENTRY`` as this is the default use case for LogSentinel. However, database monitor agents and other system or network logging solutions can be attached to LogSentinel as well, and this header allows for that.
* ``


Additional parameters
*********************
You can specify any number of additional query parameters (after ? in the URL) for all endpoints below. There are are a few standard ones


* ``actorRoles``, ``actorDisplayName`` – with them you can assign a search-friendly and dashboard-friendly roles of the actor as well as a human-readable alias (in addition to the ID passed in the path)
* ``actorDepartment`` - the department where the actor belongs
* ``gdprCorrelationKey`` - a key to correlate the entry with a certain GDPR process from the Article 30 register we provide
* ``process`` - the name of the business process from which the event originates
* ``directExternalPush`` - you can designate certain events to be directly pushed to either Ethereum, a qualified electronic timeestamp provider, email or twitter (respective values being ``ETHEREUM``, ``QTSA``, ``EMAIL``, ``TWITTER``). You can find more details in the our :doc:`On-premise security <onpremise/security>_ page.
* ``encryptedKeywords`` – with it you can enable search in encrypted payload. See more details in the next section
You can use your custom parameters to perform structured searches by prefixing ``additionalParams.``. For example you can pass 

Response
********
All of the methods below return a JSON or XML response (depending on the supplied Accept header) which contains two fields:


* ``logEntryId`` – the internal ID of the inserted log record
* ``lastKnownHash`` - the last known computed hash in the hash chain. Note that this is not the hash corresponding to the current item, as the hashes are computed asynchronously. You can store this value and call the ``/log/verify`` endpoint, but this is not necessary, as automatic verification is performed by the LogSentinel service on regular intervals 

``/api/log/simple``
*******************
The simple logging endpoint requires no specific parameters to be passed (apart from the authorization headers). You are free to pass anything in the body of the POST request, including encrypted or hashed versions of the event details.

``/api/log/{actorId}/{action}``
*******************************
The simple logging endpoint requires no specific parameters to be passed (apart from the authorization headers). You are free to pass anything in the body of the POST request, including encrypted or hashed versions of the event details.

``/api/log/{actorId}/{action}/{entityType}/{entityId}``
*******************************************************
The full logging endpoint passing the following parameters as part of the path:

* ``actorId`` – the ID of the actor who performed the action leading to this event. Normally this is a userId, but in some cases it can be a system process name or plugin name (e.g. in the case of WordPress), in case the action is performed in the background
* ``action`` - you can use any action name that makes sense for your application. The regular can be INSERT/UPDATE/DELETE/GET, but there is no limitation.
* ``entityType`` - the type of the entity that is modified. If there is no entity, use the endpoint below. Usually that would correspond to a database table name or an ORM-mapped class name
* ``entityId`` - the ID of the entity that this event is about

``/api/log/{actorId}/{action}``
**********************************************
Same as the above endpoint, but used in case there is no particular entity (for example a user kicks-off a background process, or performs a search)


``/api/log/document/{actorId}/{documentAction}/{documentId}``
****************************************************************************
Useful when working with documents, rather than audit log events. Each time a document is created, modified or deleted, this can be logged

* ``documentAction`` - ``CREATE_DOCUMENT``, ``UPDATE_DOCUMENT``, ``DELETE_DOCUMENT``, ``RETRIEVE_DOCUMENT``, 
* ``documentId`` - can be the document name or another identifier. 
* Document type can be specified via a query param (e.g. ``?documentType=PDF``)

When documents are logged, you can perform regular verifications on the integrity of your documents – do a search for particular document names and check if the hashes that you’ve originally passed match the ones stored at LogSentinel.

``/api/log/{actorId}/auth/{action}``
*************************************
This endpoint is about authentication-related actions. The allowed values for the ``action`` parameter are: ``LOGIN, LOGOUT, SIGNUP, AUTO_LOGIN, LOGIN_AS, LOGIN_FAILED``

``LOGIN_AS`` is used when a staff member logs in on behalf of a user and the ``AUTO_LOGIN`` can be used to distinguish regular login from remember-me functionality.

This endpoint allows two additional optional headers – ``Signed-Login-Challenge`` and.. ``User-Public-Key``. In case your users are authenticating using a private key (or a password-derived private key, `e.g. using WebCrypto API <https://techblog.bozho.net/electronic-signature-using-webcrypto-api/>`_ ), you can have them sign a login challenge with their private key and provide the signature and the public key. The login challenge can be the login event details, or a custom challenge that you can pass as an additional parameter. That way their authentication bears more legal strength, as they cannot deny having logged in (the signature has the non-repudiation property).

/api/log/batch
**************
This method is used for batch inserts. It is generally recommended to insert events as soon as they occur, to avoid any intermediate tampering on the client side. But in some cases it makes sense to group requests (e.g. an agent that listens to the database audit log / query log – making a request for each query might mean excessive number of requests) The method accepts only a request body in the following format (all the fields are optional, but you should specify at least one for the entry to make sense):

      [
        {
          "actionData": {
            "action": "ACTION",
            "details": "",
            "entityId": "123",
            "entityType": "BASKET",
            "entryType": "BUSINESS_LOGIC_ENTRY"
          },
          "actorData": {
            "actorDisplayName": "John Doe",
            "actorId": "123",
            "actorRoles": [
              "manager"
            ]
          },
          "additionalParams": {}
        }
      ]
	  
Note that if you want to provide a signature, you have to provide it in ``additionalParams`` with a field ``signature`` per entry, rather than one signature for the whole request.

/api/search
***********

With that endpoint you can perform programmatic search on your stored events. The parameters are:



* ``query`` – the Lucene query to perform against the search engine. You can read more about the query syntax here.
* ``startTime`` - epoch millis of the start of the period you want to limit your search to
* ``endTime`` - epoch millis of the end of the period you want to limit your search to
* ``page`` - the page number for the search results
* ``pageSize`` - the page size for the search results

The response is a list of audit log entry details:

      [
         {
            "id":"89b71f20-512c-11e7-815e-c3cda8182be4",
            "timestamp" :1497463738898,
            "actorId":"1",
            "actorRoles":[
               "administrator",
            ],
            "action":"UPDATE",
            "entityId":"195",
            "entityType":"Post",
            "details":{
               "PostID":195,
               "PostType":"post",
               "PostTitle":"...",
               "CurrentUserID": 1
            },
            "applicationId":"07d7ed50-5040-11e7-863a-6bd5280da4f2",
            "ipAddress":"172.31.15.212",
            "actorDisplayName":"admin",
            "previousEntryId":"3f36b0e0-5128-11e7-815e-c3cda8182be4",
            "hash":"cvpyp98p7pjg8GZjXpQ-kpFH8hqnUq9IGArzrUBhk_KsgOy2-9ZZSvr-g4bJOWiXeqsbFvQCNRXqHNMoWK6x7g==",
            "timestampGroupSize": 1
         }
      ]


/api/verify?hash={hash}
***********************
An endpoint for manual verification whether the supplied hash is present in the hash chain. A missing hash would indicate tampering. Note that it is not necessary to use this endpoint, ad automatic log verification is performed by LogSentinel on regular time intervals.

Hashable content endpoints
**************************
There are endpoints that are equivalent to the above (in terms of path variables, headers and parameters), but instead of ``/log`` begin with ``/getHashableContent``.

These endpoints return the content that is hashed given a particular logging request. This introduces transparency, as you can manually apply the SHA-512 hash function to the returned hashable content and compare it with the hash that LogSentinel computes for each event.

curl example
************
Below is a ``curl``

example to get you started with the API

``
 curl -X POST -u $ORGID:$SECRET --header 'Content-Type: application/json' --header 'Accept: application/json'--header 'Application-Id: 123e4567-e89b-12d3-a456-426655440000' -d '{"details": 1}''https://api.logsentinel.com/api/log/actor-1/ACTION'
``
 
For more experiments, `obtain API credentials <https://app.logsentinel.com/api-credentials>`_ and experiment on our `API page <https://api.logsentinel.com/api>`_
