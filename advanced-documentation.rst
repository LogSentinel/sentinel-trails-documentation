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



* ApplicationId – obtained from the “API credentials” section in the admin panel and passed with a
* .. code:: text

 Application-Id


* header
* OrganizationId – obtained from the “API credentials” section and passed, alongside with the Secret parameter, in an
* .. code:: text

 Authorization


* header
* Secret – obtained again from the “API credentials”

You’d also have to make a choice whether you want to send the full details of events, their hashes, or an encrypted version of the details. The latter two would practically disable parts of the search functionality, but will mean that the log server does not have any privacy related details

Additional headers
******************


* .. code:: text

 Signature


* – passing the
* .. code:: text

 Signature


* header is recommended. It contains a signature of the whole request body. The signature is an RSA “signature” using a locally (on your application end) generated key that the LogSentinel server does not have. The premise for LogSentinel is that even if a log server is compromised, modification of the records will be detectable. In the unlikely event of a LogSentinel server being compromised, an attacker could insert fake records, but if they don’t possess the private key to sign the records, it will be detectable upon inspection.
* .. code:: text

 Audit-Log-Entry-Type


* – the type of the audit log entry. Allowed values are
* .. code:: text

 BUSINESS_LOGIC_ENTRY


* ,
* .. code:: text

 DATABASE_QUERY


* and
* .. code:: text

 SYSTEM_EVENT


* . The header is optional and the default value is
* .. code:: text

 BUSINESS_LOGIC_ENTRY


* as this is the default use case for LogSentinel. However, database monitor agents and other system logging solutions can be attached to LogSentinel as well, and this header allows for that.

The algorithm used for signing should be.. code:: text

 SHA256withRSA

and the result should be Base64 encoded and set in the.. code:: text

 Signature

header. You can configure your public key in the application configuration so that verification is performed automatically.

Additional parameters
*********************
You can specify any number of additional query parameters (after ? in the URL) for all endpoints below. There are are a few standard ones



* actorRoles, actorDisplayName – with them you can assign a search-friendly and dashboard-friendly roles of the actor as well as a human-readable alias (in addition to the ID passed in the path)
* encryptedKeywords – with it you can enable search in encrypted payload. See more details in the next section

` <>`_

Searchable encryption
*********************
LogSentinel’s API provides mechanism of searching in encrypted details. To be able to do this, you must perform the following 3 steps before sending data to the API:



1. Extract parameters and keywords from the payload, by which events will be searchable
2. Encrypt payload details with symmetric key. The algorithm must be AES (128 or 256). Important: for better security you should put a random block of 16 bytes in front of plain message before encrypting.
3. Encrypt each keyword with the same key as in step 2 and hash it with SHA-256

Make a request to any of the.. code:: text

 /api/log/

endpoints with the encrypted payload and parameters and add additional request param.. code:: text

 encryptedKeywords

with value=comma separated list of all encrypted and hashed keywords.On the LogSentinel application dashboard page you can provide your key (Base64 encoded), thus enabling both decrypting encrypted details and searching by keywords in it. Note that **the key is not sent to the server** , so your encrypted details remain secret.

Response
********
All of the methods below return a JSON or XML response (depending on the supplied Accept header) which contains two fields:



* .. code:: text

 logEntryId


* – the internal ID of the inserted log record
* .. code:: text

 lastKnownHash


* – the last known computed hash in the hash chain. Note that this is not the hash corresponding to the current item, as the hashes are computed asynchronously. You can store this value and call the
* .. code:: text

 /log/verify


* endpoint, but this is not necessary, as automatic verification is performed by the LogSentinel service on regular intervals

.. code:: text

 /api/log/simple


**********************************
The simple logging endpoint requires no specific parameters to be passed (apart from the authorization headers). You are free to pass anything in the body of the POST request, including encrypted or hashed versions of the event details.

.. code:: text

 /api/log/{actorId}/{action}


**********************************************
The simple logging endpoint requires no specific parameters to be passed (apart from the authorization headers). You are free to pass anything in the body of the POST request, including encrypted or hashed versions of the event details.

.. code:: text

 /api/log/{actorId}/{action}/{entityType}/{entityId}


**********************************************************************
The full logging endpoint passing the following parameters as part of the path:



* .. code:: text

 actorId


* – the ID of the actor who performed the action leading to this event. Normally this is a userId, but in some cases it can be a system process name or plugin name (e.g. in the case of WordPress), in case the action is performed in the background
* .. code:: text

 action


* – you can use any action name that makes sense for your application. The regular can be INSERT/UPDATE/DELETE/GET, but there is no limitation.
* .. code:: text

 entityType


* – the type of the entity that is modified. If there is no entity, use the endpoint below. Usually that would correspond to a database table name or an ORM-mapped class name
* .. code:: text

 entityId


* – the ID of the entity that this event is about

.. code:: text

 /api/log/{actorId}/{action}


**********************************************
Same as the above endpoint, but used in case there is no particular entity (for example a user kicks-off a background process, or performs a search)

.. code:: text

 /api/log/document/{actorId}/{documentAction}/{documentId}


****************************************************************************
Useful when working with documents, rather than audit log events. Each time a document is created, modified or deleted, this can be logged... code:: text

 documentAction

could be one of.. code:: text

 CREATE_DOCUMENT

,.. code:: text

 UPDATE_DOCUMENT

,.. code:: text

 DELETE_DOCUMENT

or.. code:: text

 RETRIEVE_DOCUMENT

. The request body could be the hash of the document, or in rare cases – Base64-encoded binary... code:: text

 documentId

can be the document name or another identifier. Document type can be specified via a query param (e.g... code:: text

 ?documentType=PDF

. When documents are logged, you can perform regular verifications on the integrity of your documents – do a search for particular document names and check if the hashes that you’ve originally passed match the ones stored at LogSentinel.

.. code:: text

 /api/log/{actorId}/auth/{action}


***************************************************
This endpoint is about authentication-related actions. The allowed values for the.. code:: text

 action

parameter are:.. code:: text

 LOGIN, LOGOUT, SIGNUP, AUTO_LOGIN, LOGIN_AS, LOGIN_FAILED

The.. code:: text

 LOGIN_AS

is used when a staff member logs in on behalf of a user and the.. code:: text

 AUTO_LOGIN

can be used to distinguish regular login from remember-me functionality

This endpoint allows two additional optional headers –.. code:: text

 Signed-Login-Challenge

and.. code:: text

 User-Public-Key

. In case your users are authenticating using a private key (or a password-derived private key, `e.g. using WebCrypto API <https://techblog.bozho.net/electronic-signature-using-webcrypto-api/>`_ ), you can have them sign a login challenge with their private key and provide the signature and the public key. The login challenge can be the login event details, or a custom challenge that you can pass as an additional parameter. That way their authentication bears more legal strength, as they cannot deny having logged in (the signature has the non-repudiation property).

.. code:: text

 /api/log/batch


*********************************
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
Note that if you want to provide a signature, you have to provide it in.. code:: text

 additionalParams

with a field.. code:: text

 "signature"

per entry, rather than one signature for the whole request.

.. code:: text

 /api/search


******************************
With that endpoint you can perform programmatic search on your stored events. The parameters are:



* .. code:: text

 query


* – the Lucene query to perform against the search engine. You can read more about the query syntax here.
* .. code:: text

 startTime


* – epoch millis of the start of the period you want to limit your search to
* .. code:: text

 page


* – the page number for the search results
* .. code:: text

 pageSize


* – the page size for the search results

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
.. code:: text

 /api/verify?hash={hash}


******************************************
An endpoint for manual verification whether the supplied hash is present in the hash chain. A missing hash would indicate tampering. Note that it is not necessary to use this endpoint, ad automatic log verification is performed by LogSentinel on regular time intervals.

Hashable content endpoints
**************************
There are endpoints that are equivalent to the above (in terms of path variables, headers and parameters), but instead of.. code:: text

 /log

begin with.. code:: text

 /getHashableContent

These endpoints return the content that is hashed given a particular logging request. This introduces transparency, as you can manually apply the SHA-512 hash function to the returned hashable content and compare it with the hash that LogSentinel computes for each event.

curl example
************
Below is a.. code:: text

 curl

example to get you started with the API... code:: text

 curl -X POST -u $ORGID:$SECRET --header 'Content-Type: application/json' --header 'Accept: application/json'--header 'Application-Id: 123e4567-e89b-12d3-a456-426655440000' -d '{"details": 1}''https://api.logsentinel.com/api/log/actor-1/ACTION'
 
 
 
 For more experiments, `obtain API credentials <https://app.logsentinel.com/api-credentials>`_ and experiment on our `API page <https://api.logsentinel.com/api>`_
 
 GDPR-related functionality
 **************************
 The General Data Protection Regulation requires systems to be upgraded to follow certain rules. Some of the requirements can’t be handled by a drop-in solution, but some can. That’s why LogSentinel supports a number of features – a GDPR register and GDPR-specific logging endpoints.
 
 The GDPR endpoints are under.. code:: text
 
  /api/log-gdpr/
 
 . There you can log consent and all requests by data subjects in a way that you can prove to regulators the events that happened. More details can be found in the `API console <https://app.logsentinel.com/api>`_ .
 
 The GDPR record register, where a company should enlist all its processing activities (what types of data about what types of data subjects it processes). What does this have to do with audit logs? Since the regulation requires processing of data to be authorized, and the integrity of the data to be guaranteed, audit logs can be mapped to a particular processing activity in the register. That for each processing activity you’ll be able to track the relevant actions. This is done simply by providing an extra GET parameter to the logging call –.. code:: text
 
  gdprCorrelationKey
 
 . Each processing activity can be assigned a unique correlation key to make it match the audit log records.
 
 Additionally, you can use the GDPR register (via the.. code:: text
 
  /gdpr
 
 API endpoints) to fetch information about processing activities in order to display it to users for the purpose of collecting their consent. It’s best to have the register and your website in sync, so that all consent-dependent activities are covered and the user explicitly agrees to each of them.
 


