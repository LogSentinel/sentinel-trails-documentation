Searchable encryption
=====================
LogSentinel’s API provides mechanism of searching in encrypted details. To be able to do this, you must perform the following 3 steps before sending data to the API:

1. Extract parameters and keywords from the payload, by which events will be searchable
2. Encrypt payload details with symmetric key. The algorithm must be AES (128 or 256). Important: for better security you should put a random block of 16 bytes in front of plain message before encrypting.
3. Encrypt each keyword with the same key as in step 2 and hash it with SHA-256

Make a request to any of the ``/api/log/`` endpoints with the encrypted payload and parameters and add additional request param ``encryptedKeywords`` with value=comma separated list of all encrypted and hashed keywords. 

On the LogSentinel application dashboard page you can provide your key (Base64 encoded), thus enabling both decrypting encrypted details and searching by keywords in it. Note that **the key is not sent to the server**, so your encrypted details remain secret.