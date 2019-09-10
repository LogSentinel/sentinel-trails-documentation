White Paper
===========
Motivation
**********
Almost every system needs to keep audit logs – information on “who did what and when”. For example:



* Who changed the patient’s medical history
* Which bank employee viewed the customer bank account
* Which CCTV camera has taken a particular video and when

It’s not just important who did it, but also what exactly they changed or viewed and when. The whole set of details (actor, action, entity modified, details of the action) has to be stored. But simply storing it won’t be sufficient, as the stored data can be modified – there’s a need for securely storing it and guaranteeing its integrity.

Whenever confronted with the need to store an audit log (which is the case for almost all applications), companies go for custom home-grown solutions. In the best case scenario they use some plug-in to their database access technology (e.g. an ORM) that automatically handles modifications. Very rarely these solutions cover the following set of requirements:



* Business-logic level events as well as insert/update/delete/get events have to be stored – it’s not sufficient to just see the database rows that were updated, sometimes high-level operations like “basket checkout”, “bank transfer initiated” or “medical examination performed” need to be stored as well, as they carry more meaning to the business than a bunch of database table updates.
* The audit-log has to be tamper-proof, i.e. nobody, even system administrators, can alter it without being detected. That way the audit log has a more significant legal strength in courts. And the business can make certain guarantees to their clients.
* The audit-log has to be easily searchable and navigable – anyone with proper access (a high level manager, line manager or even external auditor) can easily see sequences of events that lead to a particular issue

And when speaking of issues – data manipulation attacks, both from insiders and outside attackers, `are a serious threat <http://www.darkreading.com/attacks-breaches/data-manipulation-an-imminent-threat-/a/d-id/1326864>`_ nowadays. `Wired has put it in the top security threats prediction for 2016 <https://www.wired.com/2016/01/the-biggest-security-threats-well-face-in-2016/>`_ . Without taking additional measures, no business is safe from having their data manipulated for the benefit of other parties. And often without even detecting that until it’s too late (this is why `NIST <http://csrc.nist.gov/publications/nistbul/itl97-03.txt>`_  `recommends <https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-92.pdf>`_ as well as `OWASP <https://www.owasp.org/index.php/Error_Handling,_Auditing_and_Logging#Audit_Trails>`_ recommend using audit logs)

Last, but not least, public sector entities, as well as regulated businesses, need to comply with a set of security regulations. Ticking the “compliance” box may not be that hard, but the liabilities for not protecting customers’/citizens’ data are still a serious risk to be taken into account.

Speaking of regulations, the General Data Protection Regulation in the EU mandates that every system that processes personal data (which is true for most systems) must have an audit log and not complying with the regulation may lead to significant fines for any business.

While many companies think they have audit logs, they are almost never properly protected. And as `one book on ISO 27001 <https://books.google.rs/books/about/Information_Security_Management_Professi.html?id=TiRIDwAAQBAJ&redir_esc=y>`_ warns:

 System logs need to be protected, because if the data can be modified or data in them deleted, their existence may create a false sense of security.There needs to be a product that addresses all of the above concerns and observations, and LogSentinel aims to be such a product.

Features
********
LogSentinel offers the following core set of features



* Easy integration – the RESTful API is very simple and can be plugged in new and existing systems painlessly. Using one of the provided client libraries simplifies this process even further. No need for complex setups or lengthy integration processes.
* LogSentinel runs in the cloud, but can also be deployed on-premise. This gives sufficient flexibility, based on the particular buainess case – small businesses may not need or be able to manage their own additional servers (even though the setup is easy), whereas big enterprises may not wish to trust their sensitive audit logs to a cloud solution.
* The audit log is protected by a technology similar to the way the integrity of the blockchain is protected. No modification to the log can be made without becoming evident almost immediately. This also bears legal strength – e.g. if a given event data is not tampered with, it is guaranteed that the event really happened.
* The authentication-specific API endpoints provide a way for actors to digitally sign their logins, for example, which according to the relevant legislation may give additional legal strength in case an issue is brought to court.
* A user-friendly dashboard is provided for searching and navigating through the audit events – for example “who modified that user account at 6 pm yesterday” or “what exactly did employee X do on his last shift”
* Protecting the integrity of all the data – while the integrity of the data itself is not the goal of LogSentinel (as opposed to protecting the integrity of the audit log), having a full secure audit log on all data changes means that the audited data is also protected – you can always trace the data modifications in the log and verify whether a given change occurred “naturally”, or was altered inappropriately.

Technology
**********
The technology used is based on a number of peer-reviewed papers, e.g. Audit Logs to Support Computer Forensics `[1] <https://www.schneier.com/academic/paperfiles/paper-auditlogs.pdf>`_ by Bruce Schneier and John Kelsey and Efficient Record-Level Keyless Signatures for Audit Logs `[2] <https://eprint.iacr.org/2014/552.pdf>`_ by Ahto Buldas et al.

The implementations is customized to account for the various business cases that should be addressed. Some properties have been dropped or moved to the client/client libraries, and the need for scaling the product and providing multi-tenancy has lead to a bit more complicated processing logic.

Notably, the technology relies on consecutively hashing all incoming audit log events, where each subsequent hash is formed by the data of the current entry combined with the hash of the previous entry. That way the audit log entries form a hash chain that cannot be “broken” – i.e. any manipulation to any of the entries will result in invalid hashes from that moment on.

In addition to forming a hash chain, entries form groups, which are timestamped using a client-provided timestamping authority. The group is represented by a single value, which is the root of the merkle tree of the hashes of all the entries in a group. A merkle tree is a data structure used in the blockchain to guarnatee the integrity of each block. The timestamping provides additional integrity guarantees and legal strength (EU Regulation 910/2014 defines the rules for trusted timestamping)

The integrity of the log is guaranteed even if a malicious actor gains access to the log database. So you don’t have to trust the LogSentiel cloud service, or even your own employees who manage the hosted solution. In order to achieve that, there is one “catch” – the latest hash in the hash chain at a given moment has to be kept in an unmodifiable way. When you have a given hash, the fact that it is present in the audit log guarantees that it hasn’t been tampered with. And vice-versa – if a hash that was previously stored is missing from the audit log, it means the log has been tampered with. There are multiple ways to store these latest hashes in an unmodifiable way, and LogSentinel supports:


* Store it on the Ethereum blockchain. Public blockchains are the perfect candidate, as they are immutable – once an entry is stored there, it cannot be removed. Transaction fees are relatively cheap, so LogSentinel can regularly push the last known hashes to the Ethereum blockchain
* Print it on paper – it can be published in newspapers (which is less practical), or printed on a blank paper and stored in physically protected cases, or snail-mailed to multiple stakeholders, including auditors.
* Store it on a write-only medium. Be it a CD-R, or more generally – any `WORM storage <https://en.wikipedia.org/wiki/Write_once_read_many>`_ 
* LogSentienl returns the latest known hashes which you can decide when and how to store.
* Email it to multiple stakeholders – while email can be manipulated as well, having it distributed to multiple people, potentially with different email servers, increases the complexity of changing the hash in all places.

Conclusion
**********
LogSentinel is a secure audit log service that is simple to integrate and guarantees the integrity of all your audit data. It can be integrated in any system with minimal effort. The use of state-of-the-art cryptography and original research makes sure the data is verifiably protected and this can be proven to both customers and law enforcement.
