---
title: "Keycard"
date: 2020-05-09T13:43:32-04:00
draft: False
---
## Anselus Keycard Services

**Status:** Draft in progress  
**Submission Date:** June 3, 2020  
**Submitted By:** Jon Yoder <jsyoder@mailfence.com>  
**Abstract:** Keycard Service Architecture  

### Changelog

0.0.1: Initial submission

### TODO

- Complete Lifecycle

### Sections
- [Anselus Keycard Services](#anselus-keycard-services)
	- [Changelog](#changelog)
	- [TODO](#todo)
	- [Sections](#sections)
	- [Description](#description)
	- [Keycard Service Architecture](#keycard-service-architecture)
	- [Keycard Database Structure](#keycard-database-structure)
	- [The Resolution Process](#the-resolution-process)
	- [DNS Management Records](#dns-management-records)
		- [Management Record Fields](#management-record-fields)
	- [Lifecycle: Rotation and Revocation](#lifecycle-rotation-and-revocation)
	- [Field Definitions](#field-definitions)
		- [Organizational Keycards](#organizational-keycards)
		- [Individual Keycards](#individual-keycards)
	- [Protocol Commands](#protocol-commands)


### Description

Keycards are a type of digital certificates created for the purposes of cryptographic verification and identification of senders and recipients. They are coupled with a DNS-like resolver service so that human-readable identification information can be used to easily locate the less-friendly cryptographic information.

<a name="architecture"></a>

### Keycard Service Architecture

Anselus keycard services are designed to be a loosely coupled federation, containing interaction between directories, resolvers, and issuers. An organization server plays several roles: an authoritative directory for its domain, a party in the establishment of user identities, an issuer of its own keycard, and a cache resolver for its users. Users are the issuers of their own keycards, but their cards are also signed by their organization, and keycards for all users in the organization are linked together in a blockchain that is shared in its entirety with any resolver who requests it. Keycards are issued only for organizations and users, each providing slightly different types of information.  

An overview of the resolution process as part of a contact request is as follows:  

1. Alice gives Bob her Anselus address.
2. Bob creates a contact request in his client and clicks Send.
3. Bob's client requests from his organization's server Alice's keycard and that of her organization. If the server is missing the latest index for her keycard, it downloads updates to the entire keycard database for Alice's organization and returns Alice's keycard to Bob's client.
4. Bob's client verifies the user signature on Alice's keycard and the signature on her organization's keycard.
5. Once verified, Bob's client is ready to create and send Bob's contact request. It creates the message, signs it with the signing counterpart to Bob's contact request verification key, encrypts it with Alice's contact request encryption key, and submits it to his organization's server for delivery.

Although referred to as a keycard, it contains multiple entries, from the user's initial, or **root**, entry on down to the user's current one. Because an entry is not very large -- a few hundred bytes -- a full recursive resolution of a keycard does not require much space.

The service architecture for keycards consists of keycard servers and resolvers. Resolvers work similar to DNS resolvers, caching lookups and making requests to authoritative servers for information when needed. Keycard servers store and authoritatively publish keycards for users belonging to the organization.

### Keycard Database Structure

The keycard database is a hybrid blockchain, allowing anyone to download and view its contents, but permitting only authorized users to make any changes. The chain begins with the organization's root keycard entry and each successive database transaction is cryptographically linked to the one before it via cryptographic hashes. There are only two possible operation types: append and revoke; entries can be added to a keycard, and the entire keycard is revoked when its owning workspace is deleted. No other changes to a keycard are possible. A user makes the final signature to an entry before it is added to the database, preventing malicious changes to the database by a compromised server. It is possible for anyone at any time to download the entire database; doing so for large organizations can take quite some time, as the database can easily reach tens of gigabytes, and servers throttle the database downloads to guard against denial-of-service attacks.  

### Anselus Address Fingerprints

As an additional protection against server-based attacks on user data, an Anselus address can also be formatted to include part of the root entry fingerprint. When listed, the address can be separated by a colon from its fingerprint, or the fingerprint can be listed on a separate line. The length of the fingerprint MUST be a minimum of 6 characters, but more may be used. Spaces MAY be used to group characters for better readability IF doing so does not create any parsing problems. Here are some formatting examples:

```
csimons/example.com:V=VdvK

csimons/example.com:V=V dvK

csimons/example.com
V=VdvK

csimons/example.com
V=V dvK
```

Use of the fingerprint is not a requirement, but it is highly encouraged for the additional security provides.

<a name="resolution-process"></a>

### The Resolution Process

A keycard is required for any contact to be made between two users on the Anselus platform. It contains cryptographically verifiable contact and encryption key information. The process is detailed below:  

1. User A, a member of Organization A, begins creating a signed contact request for User B, who belongs to Organization B.
2. The client's keycard resolver checks the card cache for a copy of a keycard for both User B and Organization B.
3. User A's client connects via TLS to Organization A's card service and requests the keycards of both User B and Organization B.
4. Organization A's server checks its cache for Organization B's keycard and, assuming that it does not have any part of Organization B's keycard database or that it has been a while since it obtained updates, contacts Organization B's server for updates.
5. Organization A's card service looks up and returns the keycards for Organization B and User B -- the entire chain of custody -- and returns them to User A's client.
6. User A's resolver, now having both cards and the verification key for Organization B's keycard, verifies Organization B's keycard.
7. Having successfully verified Organization B's keycard, the resolver then verifies User B's keycard.
8. User A's client can trust the information provided. The client creates and signs User A's contact request with his/her contact request key, encrypts it with User B's contact request encryption key, and uploads it to Organization A's server for delivery.

<a name="management-records"></a>

### DNS Management Records

It is unfortunate that so many security-oriented systems are forced to depend on a system such as DNS, enabler of a number of different kinds of exploits. Nevertheless, it is difficult to replace. DNS records are used by keycard resolvers to obtain basic configuration information and as a way to validate organization keycards.

Securing a domain's resource records with DNSSEC cannot be recommended enough. When DNSSEC signatures are present, TLS signatures can be provided in DNS and used to validate the domain's TLS certificate. In such situations, a certificate signed by a recognized third-party certificate authority (CA) is not required. In light of CA compromises in recent years, this is a notable benefit. Without the presence of DNSSEC, resolvers MUST require the TLS certificate of an Anselus server to be signed by a recognized third party CA.

An Anselus management record utilizes the resource record type `AX`, short for Anselus eXchange. Alternatively, if a TXT record type must be used, the name subdomain prefix `_anselus` MUST be used, e.g. `_anselus.example.com`. When searching for a record, resolvers should begin with the fully-qualified domain name for the service and work its way up the domain hierarchy until a matching resource record is found or the top level of the domain has been reached. For example, if a resolver is attempting to resolve `sub.domain.example.com`, it should first look for an `AX` record for `sub.domain.example.com` or a `TXT` record with the name `_anselus.sub.domain.example.com`. Not finding one, then it should follow the same procedure for `domain.example.com`, and then finally `example.com`. If the hierarchy has been traversed and no management record has been found, the domain is to be assumed to not offer Anselus services.

When working with DNS TXT records and the maximum length of 255 characters per string, fields MUST NOT be split across strings. A good policy for TXT record fields would be one string per field. Likewise, for maximum compatibility, DNS responses should be no longer than 512 bytes. Given the short length of Anselus DNS record fields, this should not be difficult.

#### Management Record Fields

In a departure from using the [RFC 1924](https://tools.ietf.org/html/rfc1924) version of Base85 for binary-to-text encoding for the Anselus platform, all management fields utilize Base64 encoding for compatibility purposes.

**pk** - REQUIRED. This contains the Base64-encoded verification key for the organization signatures found in the organization's keycard. More than one `pk` field may be found in the management record, but all signatures in the organization's keycard MUST have been created by the same private signing key and that all signatures are valid. Resolvers MUST reject any keycard which does not validate with a verification key from the management record. A `pk` key may be removed from the management record when a keycard is rotated if the previous Primary Verification Key has been republished as Secondary Verification Key. If, however, the key was compromised, the key should remain in the record until the expiration period has been completed and no keycards are in use which depend on it, even if this means resigning messages or keycards.  

**pka** - REQUIRED. The field contains the fully-capitalized algorithm for the verification key stored in `pk`. It exists for future expansion, and the only currently-supported algorithm is `ED25519`.  

**sk** - OPTIONAL. This contains the secondary verification key and corresponds to `Secondary-Verification-Key`.  

**ska** - OPTIONAL. The field contains the fully-capitalized algorithm for the verification key stored in `sk`. It exists for future expansion, and the only algorithm currently supported is `ED25519`.  

<a name="lifecycle"></a>

### Lifecycle: Rotation and Revocation

<a name="lifecycle-rotation-and-revocation"></a>

Proper key management includes occasional replacement to guard against compromise. Because of their public nature, a user's contact request keys are recommended to be rotated at least every 90 days. While the general purpose keys user encryption keys SHOULD be rotated at the same time, this is not a requirement. Organizational keys require more work to rotate, so their lifespan is 1 to 3 years. A user's other keys should also be rotated, but frequency depends on the user's needs and the size of his/her contacts list -- this type of rotation involves a system message sent to each contact and, depending on how many there are, this can be a significant amount of traffic.

Revocation processes TBD

### Field Definitions

A keycard entry consists of a series of 1-line key-value pairs. Most of a keycard's fields are relatively self-explanatory. Fields are expected to be listed on a keycard in the order below, but with the exception of signature fields, cryptographic hash fields, and the Type field, readers MUST NOT consider a keycard invalid because of a different ordering of fields so long as the the fields themselves meet all other requirements. Each field is terminated by a carriage return-newline sequence (`\r\n`). Keycard fields are required unless indicated otherwise.  

Fields which contain encryption keys, verification keys, and entry hashes follow a particular subformat. First, the keys themselves utilize Base85 encoding like the rest of the platform, but the keys are prepended by the name of the algorithm in all capital letters, and the prefix is separated from the key by a colon. The prefix has a maximum length of 16 characters, not including the colon, and MUST contain only capital letters, numbers, or dashes. An example looks like this: `Contact-Request-Verification-Key:ED25519:q~NVs$%Z82g7ZfniK3@!N+FrzcYJnawDdyYa!}@W`. Currently the only supported algorithms are `ED25519` for signing, and `CURVE25519` for encryption. More hash algorithms are supported: `BLAKE3-256` is preferred for its speed, followed by `BLAKE2`, `SHA-256`, and `SHA3-256`. This subformat exists to enable future expansion.

#### Organizational Keycards

Keycards which represent an organization contain both cryptographic information and some publicly-available contact information about the organization. Because of the extra effort required to update keycards when combined with DNSSEC, organizational keycards are intended to have lifespans of 1-3 years. Organizational keycards are self-signed using the organization's primary signing key. A secondary key MAY be included with an organization's keycard for continuity when the primary signing key has been replaced. When organizational keycard entries are updated, a new primary signing key MUST be created and the previous primary key SHOULD be included as the secondary signing key unless the previous primary has been revoked.

**Purposes**  

1. Signing user keycards
2. Encrypting delivery information (sender, recipient)
3. Signing outgoing messages
4. Making available necessary contact and support information for the organization

**Index:** The index of the entry in the organization's keycard. The index for the first entry in a keycard is always 1. Each successive entry increments this value. Its purpose to easily order all entries in the keycard.  

**Name:** name of the organization represented by the keycard.  

**Contact-Admin:** the workspace address for the party responsible for administrating the Anselus services for the organization. Example: `6321fb6e-c68c-4279-a1f4-68f05a2bb9b0/example.com`. Support requests and abuse reports are sent to this address if the `Contact-Support` and `Contact-Abuse` fields are not populated.  

**Contact-Abuse:** OPTIONAL. The Contact-Abuse field contains a numeric address for reporting abuse to the service administrator. If omitted, abuse reporting is sent to the address in the Contact-Admin field. If included and valid, this field MUST be used for abuse reporting. Provided that the server is configured correctly and the abuse address is valid, an administrator MAY opt to drop abuse messages sent to Contact-Admin to ensure clients follow support protocols. Note that abuse reports have a specific format defined in the [Messaging specification](/spec/messaging).  

**Contact-Support:** OPTIONAL. The Contact-Support fields contains a numeric address for requesting organizational support. It is intended for use ONLY by users from the organization itself, and support requests sent to this address. If omitted, support requests are sent to the address in the Contact-Admin field. If included and valid, this field MUST be used for support requests. Provided that the server is configured correctly and the support address is valid, an administrator MAY opt to drop support requests sent to Contact-Admin to ensure clients follow support protocols. Note that support requests have a specific format defined in the [Messaging specification](/spec/messaging).  

**Language:** Comma-separated list of [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) language codes which indicated languages supported when contacting the organization.  

**Primary-Verification-Key:** The primary signing key for the organization.  

**Secondary-Verification-Key:** The secondary signing key for the organization. When keys are rotated, often this key is the organization's previous `Primary-Verification-Key`.  

**Encryption-Key:** The public encryption key for the organization.  

**Time-To-Live:** Number of days in which the keycard may remain in a resolver cache. Recommended value is 14. After this period of time, a resolver MUST check to ensure that the keycard has not changed.  

**Expires:** The date and time after which this keycard is considered to be expired. Because keycards themselves are not associated with any costs, ensuring an organization ALWAYS has a valid keycard is paramount to the security of its users. Keycard resolvers and clients MUST refuse to deliver messages to domains with expired keycards.  

**Custody-Signature:** The base85-encoded chain-of-custody signature. This field does not exist in an organization's first keycard entry. It is required to follow the last informational field if it exists. The signature includes all previous fields and is signed with private half of the `Primary-Verification-Key` field from the previous entry.  

**Previous-Hash** - The `Hash` field of the previous entry in the keycard database, which is probably not the previous entry of the organization's keycard. The root entry of an organization's keycard database will not have this field, but for all others it is required.   

**Hash** - The hash of all fields listed above. This field is used for identification of the keycard entry in the organization's database. Supported hash types are `BLAKE3-256`, `BLAKE2`, `SHA-256`, and `SHA3-256` in order of preference from greatest to least.  

**Organization-Signature:** Signature of the keycard using the private half of the key in the organization's `Primary-Verification-Key` field. This field is the final field of the entry.  


#### User Keycards

Unlike organizational keycards, individual keycards are designed specifically for setting up encrypted communications between two entities while containing as little personally-identifiable information as possible. Like organizational keycards, all fields are one-line key-value pairs terminated by `\r\n` and all fields are required unless otherwise indicated.

**Index:** The index of the entry in the user's keycard. The index for the first entry in a keycard is always 1. Each successive entry increments this value. Its purpose to easily order all entries in the keycard.  

**Workspace-ID:** a version 4 universally-unique identifier (UUID) which is used to identify the workspace. This number is fixed for the lifetime of the workspace. It also may not be reused once a workspace has been deleted.   

**User-ID:** a human-friendly name for the workspace. Its relationship to the `Workspace-ID` field is similar to that of a DNS name to an IP address. This value may change at any time as per the desire of the workspace user, but it does require the creation of a new keycard entry to do so. It is to be used for human identification of a workspace, such as display in a client application. Any UTF-8 printable character except the forward slash (`/`), the backslash (`\`), and the double quotation mark (`"`). Whitespace characters (tab, space, non-breaking space, etc.) are NOT permitted.  

**Domain:** The domain to which the workspace belongs, such as `example.com`.  

**Contact-Request-Encryption-Key:** the public half of a keypair which is used to encrypt contact requests.  

**Contact-Request-Verification-Key:** the Base85-encoded key for verifying the signature on a contact request.   

**Public-Encryption-Key:** a public key which is reserved for future use. Possible uses could be for authentication, PGP, or another purpose. Although reserved, this field is required.  

**Alternate-Encryption-Key:** another public key reserved for future use. This field is optional.  

**Time-To-Live:** Number of days in which the keycard may remain in a resolver cache. Recommended value is 7. After this period of time, a resolver MUST check to ensure that the keycard has not changed.  

**Expires:** The date and time after which this keycard is considered to be expired. Keycard resolvers and clients MUST refuse to deliver messages to users with expired keycards.  

**Custody-Signature:** The Base85-encoded chain-of-custody signature. This field does not exist in a user's first keycard. It MUST be the first field following the standard informational fields and MUST be the first of the three signatures on a user keycard if it exists. It contains the signature for all previous fields and is signed with the private half of the `Contact-Request-Verification-Key` of the previous keycard entry.  

**Organization-Signature:** The Base85-encoded signature of all fields listed above, including the `Custody-Signature` field if it exists. It is signed using the private component of one of the organization's public signing keys, preferably the primary one.  

**Previous-Hash** - The `Hash` field of the previous entry in the keycard database.  

**Hash** - The hash of all fields listed above. This field is used for identification of the keycard entry in the organization's database. Supported hash types are `BLAKE3-256`, `BLAKE2`, `SHA-256`, and `SHA3-256` in order of preference from greatest to least.  

**User-Signature:** The Base85-encoded signature of all previous fields. This signature is the final field in the entry.  

<a name="protocol-commands"></a>

### Protocol Commands

**ADDENTRY**  
*Adds a keycard entry to the database*  
Parameters: None  
Returns: 200 OK fingerprint

Begins the process to submit a keycard entry to the organization's database. 

1. Client sends the `ADDENTRY` command.
2. When the server is ready, the server responds with `100 CONTINUE`.
3. The client uploads the data for entry, transmitting the entry data between the `----- BEGIN USER KEYCARD -----` header and the `----- END USER KEYCARD -----` footer. 
4. The server then checks compliance of the entry data. Assuming that it complies, the server generates a cryptographic signature and responds with `100 CONTINUE`, returning the fingerprint of the data and the hash of the previous entry in the database.
5. The client verifies the signature against the organization's verification key
6. The client appends the hash from the previous entry as the `Previous-Hash` field
7. The client generates the hash value for the entry as the `Hash` field
8. The client signs the entry as the `User-Signature` field and then uploads the result to the server using the same header and footer as the first time.
9. Once uploaded, the server validates the `Hash` and `User-Signature` fields, and, assuming that all is well, adds it to the keycard database and returns `200 OK`.

This extensive process is designed to prevent either side from doing anything improper, such as server-side man-in-the-middle attacks, uploading invalid data, or other tricks. When added, it is safe to assume that the data is mutually validated and that the data itself is trustworthy even if neither party is trusted by the other. Each line in the entry MUST be terminated by a carriage return-line feed (`\r\n`) sequence to ensure that the signatures remain valid. The client is expected to generate a hash for the entry using the same algorithm as the previous entry.  


**GETENTRIES**  
*Requests updates made to the keycard database**  
Parameters: (start_fingerprint)  
Returns: 102 ITEM item_index item_count  

`GETENTRIES` requests updates made to a server's keycard database. A client is not required to be authenticated to issue this command. The client may specify a hash of the last entry it has. If no fingerprint is specified, the client is requesting the entire database. Each keycard entry is transmitted as a `102 ITEM ` line, then a `----- BEGIN USER KEYCARD -----` line, the actual entry data, and finally followed by the line `----- END USER KEYCARD -----`. Both are markers of the entry data without being part of the entries themselves. The `ITEM` line returned consists of the return value, the string `ITEM`, the 1-based index of the keycard, the total count of items to be returned, and the fingerprint for the item. As a precaution, for every 10 entries transmitted, the server will wait for the client to transmit a `100 CONTINUE` before transmitting the next 10 entries. Failure to do so will result in the server idling until the connection times out.  



**ISCURRENT**  
*Verifies that an entry is the current one*  
Parameters: domain_or_workspace_address index  
Returns: 200 OK response  

This command verifies that the fingerprint supplied is the current one for the domain or workspace specified. On success, the response is either `YES` or `NO`. The client is not required to be authenticated for this command. 


**ORGCARD**  
*Requests an organization keycard*  
Parameters: domain start_index (end_index)  
Returns: 102 ITEM item_index item_count  

Requests part or all of an organization's keycard, given a domain, the starting index, and possibly an optional end index. The client is not required to be authenticated for this command. If the ending index is omitted, all cards from the specified entry through the organization's current keycard are returned. If the starting index is set to 0 or a negative number, only the organization's most recent entry is returned. Each keycard entry is transmitted as a `102 ITEM ` line, then a `----- BEGIN USER KEYCARD -----` line, the actual entry data, and finally followed by the line `----- END USER KEYCARD -----`. Both are markers of the entry data without being part of the entries themselves. The `ITEM` line returned consists of the return value, the string `ITEM`, the 1-based index of the keycard, the total count of items to be returned, and the fingerprint for the item.   


**USERCARD**  
*Retrieve user keycard*  
Parameters: workspace_address start_index (end_index)  
Returns: 102 ITEM item_index item_count  

Requests part or all of a user's keycard, given a workspace address, the starting index, and possibly an optional end index. The client is not required to be authenticated for this command. If the ending index is omitted, all entries starting with the specified entry through the user's current keycard are returned.  If the starting index is set to 0 or a negative number, only the user's most recent entry is returned. Each keycard entry is transmitted as a `102 ITEM ` line, then a `----- BEGIN USER KEYCARD -----` line, the actual entry data, and finally followed by the line `----- END USER KEYCARD -----`. Both are markers of the entry data without being part of the entries themselves. The `ITEM` line returned consists of the return value, the string `ITEM`, the 1-based index of the keycard, the total count of items to be returned, and the fingerprint for the item.   

