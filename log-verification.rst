Log Verification
================

Even though our technology is heavily influenced by blockchain, verification of the chain is not a given. Even in popular blockchains, it is done only in certain scenarios and only partially. Public blockchains do not perform full chain verification by default because that is not their core usecase. It is ours, however, and we need to make sure that the logs have not been tampered with. That's why there are multiple ways to verify that the chain of logs is intact.

Automatic background verification
*********************************

We perform regular verification of the log, going from the newest to the oldest log entry and matching their hashes, their timestamps, optionally their signatures as well as the hashes of the blocks  they belong to. Full verification is time-consuming, so it can be turned off for very large logs (hundreds of millions of retained entries). The period for full verification is configurable through the dashboard and usually goes from 12 to 36 hours depending on the subscription plan and perceived risks.

There's also the partial verification which is run every 15-30 minutes and it verifies only the latest inserted logs. 

In case of failure in any of these automated verificaion processes, a configured list of recipients is notified via email or :doc:`via Telegram </telegram>`.

Merkle tree proofs
******************

Merkle trees are at the core of many blockchain implementations. We support multiple `Merkle Tree endpoints <https://api.logsentinel.com/api#/Verification>`_ that allow performing consistency and inclusion proofs. `What those proofs mean and how they work is explained here <http://www.certificate-transparency.org/log-proofs-work>`_ and we recommend reading that section if you want to perform independent verification of a SentinelTrails chain. In short:

* A `consistency proof <https://api.logsentinel.com/api#!/Verification/getConsistencyProof>`_ guarantees that the chain has not been tampered with between two publicly known root hashes. Normally you obtain the two roots (e.g. from those pushed to Ethereum or a third party service), you get the consistency proof and validate it.

* An `inclusion proof <https://api.logsentinel.com/api#!/Verification/getInclusionProof>`_ guarantees that a particular entry is in the chain. You pass the expected standalone hash of the entry (as per the `Hashable contenet endpoints result <https://api.logsentinel.com/api#/Hash>`_) and then verify whether the proof is valid. 

.. note::
  
    We have built a sample verification application that uses merkle proofs and hashes, published to Ethereum in order to verify the chain. You can `look through its code here <https://github.com/LogSentinel/logsentinel-java-client-verification-ui/>`_

Incremental verification
************************

When you know a certain set of hashes, e.g. those published to Ethereum,  pushed to Twitter, sent via Email, or stored in Glacier, you can request all entires that lie between two known hashes in the chain by using `the /api/verification/entries endpoint <https://api.logsentinel.com/api#!/Verification/getEntriesBetweenHashes>`_.

.. note::

    Because we form hashes by concatenating the log entry data in specific order, you can use the `hashable content endpoints <https://api.logsentinel.com/api#/Hash>`_ to obtain the content for a given entry would look like and then apply SHA-512 on it to obtain the hash. 

When you have all entries between the two known hashes, you can make sure that the chain is properly computed or whether some log entry has been tampered with.

Manual verification
*******************

We offer a simple UI to verify if a given hash is present in the chain. It does verify its basic integrity properties but it doesn't guarantee that the whole chain is intact, only the particular entry.

