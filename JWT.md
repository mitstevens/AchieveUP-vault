JSON Web Token - 

Used to securely transmit information between parties as a JSON object.  

They are: **Stateless** (does not store details in memory or a database), **Self-Contained** (Token carries user identity and permissions), **Tamper-Resistant** (digitally signed so the server knows it hasn't been altered in transit) 

A JWT has: **Header** (Metadata about the token), **Payload** (information about the user, ex: UserID, Role, etc), **Signature** (created by hashing the encoded header, payload, and a secret key. Ensures no data tampering, verifies the sender's authenticity) 

## Best Practices: 

***NO SECURE DATA*** - the header and payload are only *Base64-encoded* **NOT** Encrypted - **NEVER** put passwords or highly sensitive information inside the payload 

Always include a expiration claim (so tokens stop working after a set period) 

Secure storage - Store JWT's safely on the client side to protect them from cross-site scripting attacks. 