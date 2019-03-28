Getting Started
===============
Overview
********
LogSentinel is a service that lets you log all the business logic events in your application and allows searching through them. The main feature, however, is the guaranteed integrity of the log – once it enters the system, it cannot be modified without being detected. That way it follows the `OWASP recommendations <https://www.owasp.org/index.php/Error_Handling,_Auditing_and_Logging#Audit_Trails>`_ on audit trails.

LogSentinel can be used either directly through its RESTful API (e.g. when your project is entirely under your control), or via one of the plugins for 3rd party software.

RESTful API
***********
In order to get started, you can take a look at `the simple RESTful API for sending audit log events <https://app.logsentinel.com/api>`_ . There are four methods, all of which accept an arbitrary request body – it can be JSON, base64-encoded binary format, or (again encoded) encrypted data.


* ``/api/log/simple`` – this endpoint accepts just any request body (using POST) and stores it as a new event entry. It is the recommended endpoint if you want full secrecy of the data – you should encrypt it at your end and just send us the encrypted payload. Note that you won’t be able to use the search capabilities that way.
* ``/api/log/{actorId}/{action}`` - this endpoint accepts an actorId and an action in addition to the request body. The actorId is normally the userId that performed the action, and the “action” parameter is an action specific to your system that is not about a particular entity – e.g. “PERFORM_SEARCH”, “START_BACKGROUND_PROCESS”, etc.
* ``/api/log/{actorId}/{action}/{entityType}/{entityId}`` – this endpoint accepts the actorId, an action (any action, including the recommended INSERT/UPDATE/DELETE/GET and custom ones like CHECKOUT_BASKET, DISCARD_ITEMS, etc) and the entity type and ID. This is intended to store database-related, entity-oriented events. This is expected to be the most often used endpoint in your application. It is also the most transparent one, since generic functionality can be plugged in to automatically send the events to LogSentinel. Some of the client libraries provide such features. It is recommended to pass “old” and “new” values in the body of UPDATE events in order to be able to reconstruct the whole data modification process, similarly to how event sourcing works.
* ``/api/log/{actorId}/auth/{action}`` – this endpoint is specifically intended for authentication events – it only takes an actorId and authentication action (LOGIN, LOGIN_FAILED, LOGOUT, SIGNUP, LOGIN_AS (for staff acting as user), AUTO_LOGIN (in case of remember-me functionality)). In order to have some additional legal strength you can have the user sign their login event with their password (e.g. `like described here <https://techblog.bozho.net/electronic-signature-using-webcrypto-api/>`_) and pass the result of the signing using the custom headers Signature and User-Public-Key

If the metadata isn’t critical and doesn’t need to be encrypted, you can only send an encrypted body and still use the more specific endpoints. If bandwidth is an issue, e.g. in IoT context, the body itself can be a hash of the original resource (e.g. a photo)

The response contains the ID of the inserted log entry and the last known entry hash. Since forming the hash chain is sequential, we cannot return the hash of the entry you just inserted, so instead we return the last computed hash. You can choose to store that hash somewhere (or at least log it).

The hash needs to be published/stored in a place other than the audit log server, because if an attacker gets hold of the audit log and tries to re-write it, he won’t be able to do that without being detected – the hash that has been published/stored elsewhere will not be found, which will indicate tampering with the log. LogSentinel has better ways of guaranteeing that a given hash was indeed computed, thus proving the integrity of the whole audit log, but it will be easier to do periodic checks if you store some of the hashes returned.

Authentication
**************
In order to use the API, you have to authenticate your calls. For that you need to `register <https://app.logsentinel.com/app/login#signup>`_ , login to the dashboard and obtain the organization key and secret and an application id from the “API credentials” menu. Then you should pass two headers for each request:

* ``Authorization: Basic<base64(organizationId:secret)>``

* ``Application-Id:<applicationId>``


Libraries and plugins
*********************
Here you can find a `Libraries & Plugins </libraries-plugins>` for various languages and frameworks.

In addition to the libraries, we support agents and plugins for various systems. The most basic integration can be done at the database level, using the `LogSentinel database agent <https://github.com/LogSentinel/logsentinel-agent/>`_ .

Why not …?
**********
The problem of securely storing audit logs is not a strictly defined one. The reasons why certain aspects were not implemented in a particular way are discussed below

* Why not provide built-in anonymization of actors and privacy mechanisms for the data? One of the features of LogSentinel is the ability to easily search, visualize and analyze audited events. Using actor pseudonyms, encrypting the data or using bit masks when computing the trees and hashes (all as suggested in the literature) would yield the search features useless. The premise of most of the papers includes only preserving the integrity of the logs, not analyzing them. Not having the entries fully encrypted would allow machine-learning based risk analysis and alerting on malicious activities. However, the privacy features are not ignored – they can easily be achieved on the client-side, before sending. The RESTful API acknowledges that opportunity and provides an endpoint for that.
* Why not provide binary serialization support, in addition to the HTTP API? LogSentniel uses HTTP2, which is a binary protocol. Adding gzip ontop of that reduces much of the overhead of typical RESTful APIs and allows for high performance
* Why not use merkle tree for everything? The data model chosen wouldn’t benefit much from merkle trees over simple hash chains – we need to store all the data for all the entries anyway, and looking up each hash is pretty quick due to the performed indexing. We also don’t use merkle proofs, as in the typical use case there is no need for a thin (verification) client. Additionally, representing all the data as one big append-only merkle tree would introduce complexity in terms of storage and retrieval (we must not assume we will be able to fit the whole tree in memory), We use merkle trees for the timestamping process, as additional integrity guarantee. Currently it doesn’t serve as much more than a means to concatenate elements, but timestamp group merkle trees can be useful in the future. Note that even blockchains do not represent the whole chain as a single merkle tree. Instead, each block is represented by a merkle tree, whose roots represent a hash chain.
* Why not use a blockchain? The blockchain has features that are not needed in the “tamper-evident audit log” scenario. The distributed consensus algorithm (with proof or work, for example) and the distributed storage are not necessary in order to achieve the goal of ensuring the integrity of audit logs, therefore using a full-blown blockchain would be an overdesign (even though it would sound cool) and would make setting up and maintaining the system more complex. That’s why we use only the relevant bits – the hash chaining / merkle trees.
* Why not use a custom solution or syslog instead of LogSentinel? Custom solutions rarely cover the necessary features and take time and resources to implement. Using syslog or something like splunk or logstash again doesn’t cover the security requirements. One can get hash chaining ontop of syslog, as shown by one of the cited papers, but it requires additional development and knowledge on the syslog server internals. LogSentinel is a “drop-in” solution, which is used by a very simple and straightforward API.
* Why not use just timestamping? Timestamping guarantees the integrity of the timestamped groups (blocks) of entries, but does not guarantee that no record was inserted with a date in the past or that no group was deleted. The hash chain provides a strong guarantee that there were no modifications on the entire log

For more details, read the :doc:`Advanced Documentation </advanced-documentation>`
