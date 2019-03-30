Advanced FAQ
============
**Q:**  What if someone deletes data in my system, how does LogSentinel help?  
**A:**  There are several complementary measures that can be taken. First, there’s the `entity history API endpoint <https://app.logsentinel.com/api#!/audit-log-controller/getEntityHistoryUsingGET>`_ which can be used to reconstruct the current state of an entity and compare it to what is stored in a database. Additionally, you can write your backups to WORM (write-only) storage so that nobody can tamper with the backups and then once you discover tampering has happened, you can restore the particular backup and restore the data.

**Q:**  What if a system administrator is malicious?  
**A:**  Malicious administrators on our end cannot tamper with the log without being detected. If there’s a malicious administrator on your end (or if LogSentinel is run on-premise), the only thing they can do is insert fake events and block access to the LogSentinel installation on a network level. They cannot do Man-in-the-middle attacks (communication is encrypted). A best practice would be to also log admin actions, e.g. commands they execute on systems, so that the activity of blocking network traffic or inserting fake events manually is visible. An additional good practice is to sign all events with a private key (potentially stored in a hardware security module) which can later be verified using the corresponding public key.

**Q:**  It’s cool that you do periodic verifications, but how can I be sure your service is not compromised and the verifications don’t report fake successes?  
**A:**  You can call a `“get entries between hashes” <https://app.logsentinel.com/api#!/verification-controller/getEntriesBetweenHashesUsingGET>`_ endpoint to fetch all entries between two known hashes that you know for sure are in the log (e.g. they have been published to Ethereum, sent to twitter, emailed to stakeholders or printed on paper). Then you can use the `hash endpoints <https://app.logsentinel.com/api#/hash-controller>`_ and do the consecutive hashing yourself to verify whether the chain is not modified and all hashes in between exist and match.

**Q:**  Can I prove to 3rd parties that a log exists and is not tampered with?  
**A:**  Yes, there’s an option to give limited auditor access in case of audits. There’s also a user interface to verify individual hashes that you can give to your customers

**Q:**  Should I store personal data in the logs if I have to follow GDPR (The EU General data protection regulation)?  
**A:**  It is discouraged, as deleting data from the log is not desirable (the whole purpose of the log is to make it impossible to delete entries). However, depending on the legal analysis, limited personal data can be stored, pursuant to Article 17(3)(e) of GDPR if it would be used “for the establishment, exercise or defence of legal claims.”

**Q:**  What if we don’t want to risk sending sensitive data to a cloud solution?  
**A:**  LogSentinel supports :doc:`searchable encryption </searchable-encryption>` – you encrypt all the data before sending it but you can still search in it through our dashboard without us knowing its contents.
