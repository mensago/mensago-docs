---
date: 2019-09-06T19:28:55-04:00
draft: false
---
## Anselus Messaging Specification

**Status:** Draft  
**Submission Date:** September 6, 2019  
**Submitted By:** Jon Yoder <jsyoder@mailfence.com>  
**Abstract:** Anselus message structure  

### Changelog

0.0.4: Keycard integration, abuse reports, support requests  
0.0.3: Encryption and encoding changes, add contact request keys  
0.0.2: Addition of key request keys  
0.0.1: Initial submission  

### Description

The reason for Anselus' existence is to replace e-mail with a secure, private, and otherwise better technology. Messages are long-form text communications which can be divided into two classes: system messages and user messages. User messages are just that: communications written by users to be sent to other users. System messages, on the other hand, are generated to be used for management of communications on the Anselus platform. This specification describes messaging workflows and the payload portion of a client item which is used for messaging. The wrapper information for the client item, i.e. encryption type, recipient address, etc., is described in the [Anselus Client-Server API] specification.

[Anselus Client-Server API]: /spec/clientserver

### Encryption Keys

Each Anselus workspace uses multiple encryption keys. Two of the reasons for this are flexibility and security: concerns are better separated and if a key should be compromised, it is easier to update the one key for one component of the system than to update all of them.

**Primary Key** : The account primary key is an asymmetric keypair. This key is used for one-to-one messaging and encrypting other security keys for transport.  

**Signing Key** : This asymmetric keypair is the workspace's identity key.  

**Contact Request Key** : This asymmetric key is only used for contact requests. Unlike all other user keys, the public half of this one is published the user's keycard and is easily-accessible public information. 

**Contact Request Signing Key** : This asymmetric keypair is for signing contact requests before they are encrypted.  

**Broadcast Key** : This key is a symmetric key used for group messaging. The encryption standard may vary, but as of this writing, there are several supported standards: XChaCha20, ChaCha20, AES128, and AES256. Because this key can potentially be given out to large numbers of recipients, a workspace may have multiple broadcast keys.

**System Key**: Another symmetric key, the system key is used for encrypting system messages. Users never directly interact with this key.

**Social Key**: This symmetric key will be used specifically for the Terrazzo social networking module. This key type may also have multiple keys.

**Storage Key**: This symmetric key is used to encrypt client items for storage and synchronization on the server.  

**Key Request Key** : This one-time-use asymmetric key is generated when encryption keys are requested for the first of a user's devices to access a shared workspace. It is discarded after the encryption keys for a shared workspace have been obtained.

### Payload Structures

#### Sample User Message

```json
{
	"Type" : "usermessage",
	"Version" : "1.0",
	"From" : "3cb11ab3-5482-4154-8ca1-dfa1cc79371c/contoso.com",
	"To" : [ "662679bd-3611-4d5e-a570-52812bdcc6f3/anselus-example.com" ],
	"CC" : [ "dfe1e724-cfa6-4d7a-aa3a-226f87a1a456/contoso.com" ],
	"BCC" : [ "225e9d7c-c8c3-4959-966e-691941cfc1ac/anselus-example.com" ],
	"Date" : "20190905 155323",
	"ThreadID" : "2280c1f2-d9c0-440c-a182-967b16ba428a",
	"Subject" : "Sample User Message",
	"Body" : "It was a bright cold day in April, and the clocks were striking thirteen.",
	"Attachments" : [
		{
			"Name" : "foobar.png",
			"Type" : "image/png",
			"Data" : "iBL{Q4GJ0x0000DNk~Le0000A0000A2nGNE0F5%wy#N3J1am@3R0s$N2z&@+hyVZp7)eAyR2Y?G{Qv*|e+D7|6ETWL6;e+j0BM>85Q>cpXaE2J07*qoM6N<$f&"
		}
	]
}
```

**User Message Fields**

*Type* - REQUIRED. This field is required for all payloads. It defines the type of payload.  

*Version* - REQUIRED. The messaging API version.  

*From* - REQUIRED. The numeric Anselus address of the sender.  

*To* - REQUIRED. The numeric Anselus address of the recipient.  

*CC* - OPTIONAL. A list of numeric Anselus addresses of any carbon-copied recipients.  

*BCC* - OPTIONAL. A list of numeric Anselus addresses of any blind carbon-copied recipients.  

*Date* - REQUIRED. The UTC date and time when the message was sent. The date MUST be sent using [ISO 8601] basic (compact) format. The time MUST be sent in the format HHMMSS.  One space MUST be used to separate the two as in the following example: `YYMMDD HHMMSS`.  

*ThreadID* - REQUIRED. A UUID is used to represent the conversation ID.  

*Subject* - REQUIRED. A string up to 100 characters in length. The characters MUST be valid printable UTF-8 characters or a space. Note that while the field itself is required, the field itself MAY be empty.  

*Body* - REQUIRED. A string of UTF-8 characters of any length. Escapement of content for JSON compliance is required.  

*Attachments* - OPTIONAL. A list of dictionaries containing attached data. Attachment format is listed below.  

[ISO 8601]: https://en.wikipedia.org/wiki/ISO_8601

**Attachment Fields**

*Name* - REQUIRED. The name of the attached file.  

*Type* - REQUIRED. The MIME type of the attached file.

*Data* - REQUIRED. The Base85-encoded data. 

#### System Messages

System messages are not sent directly to a user. Instead, they facilitate communications and protocol state and are encrypted unless stated otherwise. Aside from those directly-related to messaging, system messages are defined in the specification to which they are related.

**Abuse Report**  

Abuse reports are sent to the address specified in the organization's keycard, or if not specified, the Admin address. It is structurally similar to a standard user message except that the subtype is `abusereport` and the subject MUST be the numeric address of the offender. The body of the message MUST contain the description of the abuse report. The submitter MAY attach a sample of the message to the administrator, if need be.

```json
{
	"Type" : "sysmessage",
	"Subtype" : "abusereport",
	"Version" : "1.0",
	"From" : "3cb11ab3-5482-4154-8ca1-dfa1cc79371c/contoso.com",
	"To" : [ "662679bd-3611-4d5e-a570-52812bdcc6f3/anselus-example.com" ],
	"Date" : "20190905 155323",
	"ThreadID" : "8e24ab6b-b466-492b-a3b1-4ce736a59563",
	"Subject" : "df7c310a-b947-4f9d-a66b-600d5fdd7e0c/anselus-example.com",
	"Body" : "This user purposely sent me malware which raised my insurance rates by 15%.",
}
```

**Support Request**  

Support requests are sent to the address specified in the organization's keycard, or if not specified, the required Admin address. Like an abuse report, a support request is structurally similar to a standard user message except that the subtype is `supportrequest`. The subject MUST contain a summary of the problem, and the body of the message MUST contain the description of the problem experienced by the submitter. Note that administrators are well within their rights to mute users who abuse the support request system, and service providers are not restricted from charging users for support.

```json
{
	"Type" : "sysmessage",
	"Subtype" : "supportrequest",
	"Version" : "1.0",
	"From" : "3cb11ab3-5482-4154-8ca1-dfa1cc79371c/contoso.com",
	"To" : [ "662679bd-3611-4d5e-a570-52812bdcc6f3/anselus-example.com" ],
	"Date" : "20190905 155323",
	"ThreadID" : "8e24ab6b-b466-492b-a3b1-4ce736a59563",
	"Subject" : "I can't find the Any key",
	"Body" : "Connect tells me to press Any key, but I can't find it on my keyboard anywhere!",
}
```

### Contact Requests

Unlike e-mail, communication with other users on the Anselus platform is on an opt-in basis. A contact request exchange similar to those found on social media must take place before any sort of communication can take place between two entities. The result is a simple, familiar concept which places users in control and provides a means to exchange encryption keys. Filtering and organizing communications is part of the design of the platform.

The contact request process is as follows:

1. User #1 retrieves and validates User #2's keycard. The keycard request is sent both through the user's server and from the user's client itself to ensure no sneaky tricks by either server. The keycard for User #2 contains an encryption key used to encrypt the contact request. More information on keycards can be found in the [Keycard Specification](/spec/keycard).
2. User #1 sends a request to User #2. This request contains whatever contact information User #1 wishes to share (name, address, etc.) in the form of a Personal Information Profile (PIP). It is signed by User #1's request signing key so that User #2 can verify that the request actually came from User #1 and encrypted with User #2's request encryption key so that no one except User #2 can read it. Once received, User #2 can determine if contact should be permitted. More information on PIPs can be found in the [Contacts Specification](/spec/contacts). 
3. User #2 may drop the request and optionally block future requests. If User #2 approves the request, an encrypted response is sent with User #2's PIP. Unlike the initial request, the acceptance message contains the full information provided in the PIP provided by User #2. 
4. User #1 receives the approval and is asked to share his/her personal information with User #2. How much information is shared is up to User #1. This response also includes other encryption keys, such as User #1's broadcast and system keys.

This process enables exchange of information without exposure to infrastructure and a minimum of back-and-forth to enable the information exchange. The combination of contact requests and required encryption enables several security advantages:

- Encryption can be computationally expensive, which makes mass messaging harder to hide on a compromised machine and slows throughput without placing undue hardships on individuals sending a message to a few friends.
- Message broadcasts are possible with shared symmetric encryption keys, but they are exchanged only after a contact request exchange is complete.
- Phishing is much more difficult because the sender's identity is required.
- Only contact requests may be sent to the user with their contact request key. Other types of messages encrypted with it are silently dropped.

**Contact Request: Stage 1 (Lookup)**

Initiated by a client when a user requests contact with another user. The client requests and resolves the other user's keycard. 

**Contact Request: Stage 2 (Initiation)**

Sent after the potential contact's request key has been received. The client is not required to provide any more personal information than that which is already available in the user's keycard. However, users are encouraged to share additional information to help the recipient validate who the sender is. With the exception of non-primary encryption keys, any field found in the [Contacts Specification](/spec/clientside/contacts) can be found as part the contact request payload. A sample payload is shown below.

```json
{
	"Type" : "sysmessage",
	"SubType" : "ContactReq.1",
	"Version" : "1.0",
	"From" : "3cb11ab3-5482-4154-8ca1-dfa1cc79371c/contoso.com",
	"To" : [ "662679bd-3611-4d5e-a570-52812bdcc6f3/anselus-example.com" ],
	"Date" : "20190905 155323",
	"Sensitivity" : "Public",
	"EntityType" : "individual",
	"Name" : {
		"Given" : "Richard",
		"Family" : "Brannan",
	},
	"Gender" : "Male",
	"Keys" : {
		"Primary" : {
			"Key-Hash" : "BLAKE2B-256:Ce?6fLm)-h{el{F7%A{9R76X_+N{96MQ-qUP?S?Q",
			"Value" : "CURVE25519:h=x-k3#Xvkq6nw;ow(pWSH82r%#gI$WLRf*TRi1a""
		}
	}
}
```

**Contact Request: Stage 3 (Response)**

Sent by a contact request recipient to approve a contact request. Should the recipient approve the request, the approval message is sent with the recipient's contact information. Unlike the sender's initial request, this response contains all of the contact information which the recipient intends to share with the sender. This payload uses the subtype `ContactReq.2`. A recipient can report a contact request as spam to the Abuse address at the server of the sender's organization.

**Contact Request: Stage 4 (Acknowledgement)**

Sent by the initial contact request sender to fill in any information not initially sent. Additional information is not required for the acknowledgement, but this third step enables a sender to share enough information to be identified by the recipient in the initial message without sending potentially sensitive information visible to the sender's provider or the recipient's provider. This payload uses the subtype `ContactReq.3`. Note that the information sent in this message is supplemental to that sent in the initial request. The recipient's address book information is updated when this message is received. When this message is sent, the client application should make a note of what information profile was used for future change updates.

**Contact Information Update**

Sent by a user to notify contacts of a change in contact information. The payload sent uses the subtype `ContactUpdate`. The fields and structure are exactly the same as the contact requests, but the update message is encrypted with the user's system key, not the recipient's contact request key. Empty fields which are sent are intended to delete information which was previously available. Note that any client-side annotations made by the recipients to the sender's contact information are retained, but the information provided by the sender is not.

