---
title: "Anselus Client-Server API"
date: 2019-08-10T09:18:00-04:00
draft: false
---

**Status:** Draft (incomplete)  
**Submission Date:** August 10, 2019  
**Submitted By:** Jon Yoder <jsyoder@mailfence.com>  
**Abstract:** Spec for Anselus client-server communications

### Changelog

0.0.1: Initial submission

### Description

Client-server communications on the Anselus platform take inspiration from a number of other of protocols that have gone before it, including [POP3], [SMAP], and [JMAP]. The protocol operates under three guiding principles:

1. Minimal Metadata: The server must know only the bare minimum amount of information necessary to perform its duties.  
2. Guarded Trust: Trust in an entity (user, device, server, etc.) does not imply free rein for that entity. Data is still checked for integrity, steps are taken to prevent abuse, and boundaries are kept to prevent problems from scaling larger. For example, it is possible that a contact in the user's address book is unaware that he or she has been compromised and has a device that is attempting to send out spam. Thus, even though the device is given a level of trust, boundaries for behavior are maintained.  
3. Safety and Portability: technologies leveraged for the Anselus platform have been chosen for their portability and/or safety so that it is possible to run an Anselus server on NetBSD just as easily as on Windows.  
4. The responsibilities of the server are:
	- To provide a transport mechanism for messages to and from other servers.
	- To provide a synchronization mechanism for a user's devices.
	- To provide identity services

[POP3]: https://tools.ietf.org/html/rfc1939
[SMAP]: http://www.courier-mta.org/cone/smap1.html
[JMAP]: https://jmap.io/spec.html
[keycard]: /spec/keycard

Communication between Anselus clients and servers is a synchronous text-based protocol which operates on TCP port 2001 by default. The protocol is line-oriented and meant to be easy to parse and validate. Because client data is encrypted by default, it is assumed to be opaque and, thus, client-server commands focus on authentication, synchronization, and message delivery.

### Technology Conventions

Much has changed in the more than 40 years since e-mail's invention. Great heroics have been made in the name of compatibility with the legacy technologies POP, IMAP, and SMTP, but the architecture of these technologies is the reason behind the many problems which plague modern e-mail. A conscious choice to start fresh has been made, but not from scratch. Instead, technologies and methods have been selected which represent some of the best-of-breed in the technology industry at this time.  

Some of these technologies will sound very familiar. The protocols are text-based to enable greater flexibility and easier troubleshooting. UTF-8 has been selected as the default text encoding to provide a well-supported means of representing text for all communities, not just English-speaking ones. JSON is the default data serialization format because it is human-readable, flexible, easy-to-parse, and easy-to-validate. The version of Base85 encoding defined in [RFC 1924] is intended to be compatible with source code, making it desirable for use with JSON and yielding less overhead (25%) than that of Base64 (33%). It is the standard encoding except where noted.

[RFC 1924]: https://tools.ietf.org/html/rfc1924

For encryption, recent developments in cryptography have created strong algorithms which are also efficient. ChaCha20 and Elliptic Curve 25519 have been selected as preferred algorithms for symmetric and asymmetric encryption, respectively. BLAKE2b is a fast hashing algorithm which yields comparable or better results than SHA2 and SHA3 algorithms, and BLAKE3 shows promise of even greater performance gains. Argon2id is utilized for password hashing.

Because governmental IT tends to move at the speed of bureacracy, alternative methods are also permitted, even if not preferred. SHA2-256, AES-256 and the 2048- and 4096-bit versions of RSA are options for environments which require certified algorithms.

### Client Items

The Anselus server has a relatively small role: management of opaque data files, called **client items**. These data files are JSON-formatted text files containing an encrypted payload and a header containing some minimal metadata. The actual content of the message is stored in the payload field and, by default, is encrypted.

```json
{
	"Item": {
		"ClientItemVersion" : "1.0",
		"KeyType" : "social",
		"KeyHash" : "BLAKE2B-256:uWpi?&AVVKIo&m;O2r=i4tKT|KJ!o~fI0|@l};li",
		"To" : "ec6214be-766b-4b82-9c53-893e048755ae/example.com"
	},
	"Payload" : "XSALSA20:p!!IiyNgX$CYU+3Hqqb}9wng#^S^-k({8$Ou~NMjT?o6-ARS}uOcGZE1~>_2pLWQt={W1(?L)v&kT*;img18NVf|blhL*_lo-)Th@gQ`vkGy<4MS+M*`A5fI!=U+J;!l1(uhtuR_F>4);OpvDmSITh+|pDcN&i)61Y^n+SLO25gZ>4g&Au3adOrg}+kO%c#E@Db%jpN0rh}$e9Bk;7"
}
```

**Client Item Fields**

*Item* - REQUIRED. A dictionary containing header information for the item.  

*ClientItemVersion* - REQUIRED. A numeric string set to the client item API version. Until such time after this standard has reached final 1.0 status, this will be set to "1.0".  

*KeyType* - REQUIRED. This is the category of encryption key used. Current values may be `primary`, `system`, `broadcast`, or `social`.  

*KeyHash* - REQUIRED.  A hash of the encryption key used. The hash itself is Base85-encoded and is prefixed by the algorithm used, which can be `BLAKE2B-256`, `SHA256`, `SHA512`, `SHA3-256`, or `SHA3-512`. `BLAKE2B-256` is preferred because of its speed.  

*Payload* - REQUIRED. A string containing the real data for the item. The data is encrypted by default, and it includes the useful bits of metadata. The actual structure of the payload depends on the context, but text data is JSON-formatted. For details on payload structure, consult the Aneslus client-side API standards. The string itself is a Base85-encoded string with a prefix of the algorithm used. For speed, a symmetric key is used to encrypt the payload. The algorithm must be `AES256`, `AES128`, or `XSALSA20`. The XSalsa20 cipher uses Poly1305 for message authentication and the AES ciphers use GCM. XSalsa20 is preferred because it is faster overall without depending on hardware acceleration.  

### Limitations, Maximums, and Timeouts

E-mail has an imposed limit of 998 characters per line to account for the additional carriage return and line feed. This works for e-mail because it is limited to the ASCII character set. UTF-8 is the standard for the Anselus platform, and because UTF-8 characters can vary in size, maximums are measured in bytes. In order to accommodate 4096-bit RSA keys, one line in an client-server message may be up to 8KiB (8192 bytes), including the line ending (`\r\n`). Similar to [SMAP], Anselus commands and replies MUST be no more than 16 KiB  (16384 bytes) including line ending. This maximum applies only to commands and replies themselves and not to file transfer data. 

Although the format for client items has no theoretical limit, there are some practical limits placed on user messages. For efficency of transmission and storage, messages SHOULD be no more than 50 MiB. Server administrators MAY impose a hard limit of some size, but it SHOULD be no less than this. Client items which are not used for messages MAY be of any size, although server administrators MAY impose a maximum size for client items in general.

As part of the Guarded Trust principle and also general resource conservation, there are some soft limitations imposed on clients. An individual device is limited to 25 recipients per minute. This is a configurable soft default limit. It is intended to prevent spam and Reply All storms and encourage more thoughtful inclusion of others in group conversations.

Idle sessions MAY be ended by a server. A server MUST wait a minimum of 30 minutes before terminating a connection. Likewise, clients which are left idle for extended periods of time should wait no more than 29 minutes to periodically send `NOOP` commands to keep the connection alive.

[SMAP]: http://www.courier-mta.org/cone/smap1.html

In order to prevent a denial-of-service on servers which permit public account registration, by default a server limits account registration to once per 10 minute time period from an individual IP address. This timeout does not apply to an administrator creating accounts locally on the server itself. 10 minutes is the default, but an administrator may change this value.

### Filesystem Access

Because a server is not permitted to know more than is necessary about the information it processes, the filesystem itself utilizes opaque, but unique, identifiers for files and directories.

Universally Unique Identifiers (UUIDs) are used extensively. Files utilize a three-part naming system, consisting of the number of seconds since the UNIX epoch, the size of the file in bytes, and the file's version 4 UUID. An example looks like this: `1535760000.9457.8ba70831-d189-4aaa-b6e6-5cca0823b205`. Directories also utilize UUIDs instead of alphanumeric names.  

Server-side paths are represented in a unique way: the start of the path is always a single slash (`/`) followed by directory elements. Each directory element is separated by a space. Because filesystem entries follow a very specific format, accounting for whitespace and special characters in paths is not necessary. An example path looks like this: `/ 0cfb91e8-256b-420b-b37d-db28004120f5 aa7347c1-a837-460f-8cf0-698d4411758a ac7971bf-fe44-400c-8605-eb499b9274ad`. Server-side paths are always absolute--relative references using `.` and `..` are not supported; any path using them MUST be rejected. Lastly, path references are always relative to the root folder of a workspace; no access outside of the workspace directory hierarchy is permitted for any client.  

Although there is a standard filesystem layout for Anselus workspace data, the server is not responsible for creating any directories; it is all handled by the client. Clients are expected to maintain a mapping of the real name of a directory in the workspace to the UUID used for its name on the server side. In this manner, a malicious actor associated with the service provider is able to obtain very little useful information about any of the files stored on the system.  

### Command Reference

For any command listed below, `400 BAD REQUEST` is returned by the server if a command does not match expected syntax. It also may be returned if a command argument contains invalid data.  

**COPY**  
*Copies an item from the selected folder to another on the server*  
Parameters: file name, destination path  
Returns: 200 OK new_file_name  
Possible Errors: 401 UNAUTHORIZED, 403 FORBIDDEN, 409 QUOTA INSUFFICIENT, 404 NOT FOUND  
Example:  
```
C: COPY 1577396236.8003.7c1d6aa5-cc79-474f-ac05-201147e80e53 / f736e26e-59ee-4c21-b044-80b4c37c7044 26bae0d2-6408-4f1a-b6fc-b43b9c3d01f0  
S: 200 OK 1577396236.8003.8803eb44-a086-4b2a-81c2-0ceb9a56d9b8
```

Creates a duplicate of an item and returns the name of the item as determined by the server. Each file on the server is expected to have a unique name, so the name of the copy is returned if successful. The destination path is expected to be a list of folders. If there is not sufficient space in the filesystem or the workspace quota, `409 QUOTA INSUFFICIENT` is returned. `404 NOT FOUND` is returned if the item or the destination folder does not exist.


**DELETE**  
*Deletes a file from the selected folder*  
Parameters: filename  
Returns: 200 OK  
Possible Errors: 401 UNAUTHORIZED, 403 FORBIDDEN, 404 NOT FOUND  
Example:  
```
C: DELETE / f736e26e-59ee-4c21-b044-80b4c37c7044 26bae0d2-6408-4f1a-b6fc-b43b9c3d01f0 1577396236.8003.7c1d6aa5-cc79-474f-ac05-201147e80e53  
S: 200 OK  
```

Deletes a file from the selected folder.

**DELIVER**  
*Transfers an item from one identified server to another*  
Parameters: size in bytes, prefixed hash, destination address  
Returns: 200 OK filename  
Possible Errors: 401 UNAUTHORIZED, 409 QUOTA INSUFFICIENT  
Example:  

```
C: DELIVER 4096 BLAKE2B-256:ek#gB)U|eAN5tb=nnw#-EiEdwfol02nI&~jyDH*y 48070e99-3c2d-4a7e-978b-9c7377bb1085
S: 100 CONTINUE 1577232538.4096.a263fd0c-4de6-4644-9423-796bbe44eb50
C: (binary data uploaded)
S: 200 OK
```

A server may issue this command ONLY after receiving a `200 OK ` from a SERVERPWD command. It operates much like the UPLOAD and SEND commands. The actual DELIVER command is a request for upload, submitting the size of the item in bytes, the hash value computed on the sender's side prefixed with the algorithm used, and the numeric address of the recipient. If the specified workspace does not exist, `404 NOT FOUND` is returned and the error is logged on the receiving server. Aside from this, the commands continue in the same way as UPLOAD and SEND, including handling of lack of space, interruptions, and resuming.

404 errors are logged by servers receiving delivered items to ensure good behavior and prevent spam. Should the number of permitted delivery failures of this type exceed the limit configured on the server, `307 DELIVERY FAILURE LIMIT EXCEEDED` is returned and the connection is closed. By default, this threshold is recommended to be 500, but it can be configured to be more or less permissive. The offending server is not banned, but a configurable cooldown period must pass before delivery may be attempted. The default cooldown period is 60 minutes. If the offending server attempts to deliver before the cooldown has expired, it will receive a `308 DELIVERY DELAY NOT REACHED` response to the SERVERID command. Server implementors MAY want to log the sending workspace whenever a 404 error is received and ensure that a few misbehaving workspaces do not cause a delivery delay for the entire server to a particular domain.  


**DEVICE**  
*Finishes PLAIN authentication*  
Parameters: device_ID public_key  
Returns: 100 CONTINUE challenge, 200 OK  
Possible Errors: 101 PENDING, 401 UNAUTHORIZED, 403 FORBIDDEN  
Example:  
```
C: DEVICE d313c834-b8e9-4622-a208-40b6c03ddd2e CURVE25519:Da1`e?<a%1?x7etG?C#a^ndPgG^fc1wO_4O>Mzj>  
S: 100 CONTINUE aWgbIIXPx+Yil!WVRK|`3I  
C: DEVICE d313c834-b8e9-4622-a208-40b6c03ddd2e CURVE25519:Da1`e?<a%1?x7etG?C#a^ndPgG^fc1wO_4O>Mzj> q34999fmkk3kasdl  
S: 200 OK  
```

The `DEVICE` command is the final step in the PLAIN authentication process. The client submits the device's unique ID and device's base85-encoded public key with prefix. Devices are responsible for generating their own IDs.  

If a device's key is not found, `101 PENDING` is returned if device checking is enabled and approval is pending. While approval is pending, this command MAY be reissued to check approval status. Checks MUST NOT be performed more than once every 10 seconds. While approval is pending, `101 PENDING` will still be returned. If device approval is denied, `403 FORBIDDEN` is returned.

If the device is approved or is already in the server's device list, `100 CONTINUE` is returned along with a challenge encrypted with the device's public key. The device MUST respond with the same command along with the decrypted challenge. The challenge itself is merely a 32-bit string of base85-encoded bytes. If the device does not respond with the correct string, `401 UNAUTHORIZED` is returned and the login process is ended. Assuming that the correct challenge is sent, `200 OK` is returned and the PLAIN authentication process is successful.  


**DOWNLOAD**  
*Download an item from the selected folder*  
Parameters: filename, optional offset  
Returns: 100 CONTINUE size, 200 OK filename  
Possible Errors: 401 UNAUTHORIZED, 403 FORBIDDEN  
Example:  
```
C: DOWNLOAD 1577232538.4096.a263fd0c-4de6-4644-9423-796bbe44eb50  
S: 100 CONTINUE 4096  
C: 100 CONTINUE  
S: (file data transferred)
```

The client downloads data from a file on the server. The client first makes the request, which includes name of the file in the current folder. Assuming that all goes well, the server returns `100 CONTINUE` along with the size of the file in bytes. The client acknowledges readiness for the transfer by sending `100 CONTINUE`. The server then transmits the data. If an offset is supplied by the client, the server is expected to begin transmission starting at the specified offset in order to resume a previously-interrupted transmission.  


**EXISTS**  
*Checks for the existence of a file or folder on the server*  
Parameters: path  
Returns: 200 OK    
Possible Errors: 401 UNAUTHORIZED, 403 FORBIDDEN, 404 NOT FOUND
Example:  
```
C: EXISTS 1577232538.4096.a263fd0c-4de6-4644-9423-796bbe44eb50  
S: 200 OK  
```

Returns `200 OK` if the file or folder exists.

**GETUPDATES**  
*Requests all changes since the time specified*  
Parameters: time  
Returns: 100 CONTINUE number_of_items, change items  
Possible Errors: 401 UNAUTHORIZED  

The client requests a list of updates since the requested time. Time is submitted in seconds since the Epoch (UNIX time), UTC time. The server responds with `100 CONTINUE` and the number of update items to follow. Each successive line is a `104 UPDATE` line.

Updates begin with `104 UPDATE` followed by the update type and one or more tokens which depend on which type is used. There are only three types of updates: CREATE, DELETE, and MOVE.

`104 UPDATE CREATE / 721a1b2f-8703-4d23-8f9e-7275c647b63e 1579216613.5143.ec795b28-ea77-4b5d-b860-6d484222feb1`  
`104 UPDATE MOVE  / 721a1b2f-8703-4d23-8f9e-7275c647b63e 1579216613.5143.ec795b28-ea77-4b5d-b860-6d484222feb1 / ec795b28-ea77-4b5d-b860-6d484222feb1`  
`104 UPDATE DELETE / ec795b28-ea77-4b5d-b860-6d484222feb1 1579216613.5143.ec795b28-ea77-4b5d-b860-6d484222feb1`  

`Create` and `Delete` updates list the full path of the new item and is received even if the item is not part of the selected folder. `Move` updates provide the full path of the item prior to the move and then the new folder to which it was moved.


**LIST**  
*Gets list of items in selected folder*  
Parameters: time (optional)  
Returns: 200 OK line count, entries  
Possible Errors: 401 UNAUTHORIZED
Example:  
```
C: SELECT 8ba660ab-ca5f-44fb-931a-e20da8c6442c
S: 200 OK
C: LIST 1594329418
S: 102 ITEM 2
1594333333.26631.52df2ce2-4aae-49e0-a4dc-9e74ec225a6d
1594337219.39986984.f194415a-9fc0-44a0-867e-19ef6e6245cc
```

Obtains a list of the entries in the current folder. This command will return entries which only matches the expected filename format on the server side, consisting of a timestamp, file size, and file UUID, all three joined together with periods. This command takes an optional timestamp parameter. As with GETUPDATES, the timestamp is expected to be submitted in seconds since the Epoch (UNIX time), UTC time. If provided, only the times at or after the timestamp are returned. If omitted, all items in the current folder are returned. The server's response consists of `102 ITEM ` followed by the number of items to be returned. Each item is then returned, one item per line. If there are no items in the current directory to be returned, `102 ITEM 0` is the only response from the server.  


**LOGIN**  
*Initiates authentication*  
Parameters: login type, workspace ID  
Returns: 100 CONTINUE  
Possible Errors: 404 NOT FOUND, 405 TERMINATED
Example:  
```
C: LOGIN PLAIN b853a601-96f9-4e9e-9165-37f20cc35097/example.com
S: 100 CONTINUE
```

Initiates authentication. Currently the only type of authentication is PLAIN, which is a multistep username/password/device challenge-response login. If the workspace ID doesn't exist, `404 NOT FOUND` is returned. If multiple failures are made and reaches the server's failure limit, `405 TERMINATED` is sent and the connection is closed. If the workspace is pending moderator approval, 101 PENDING is returned. Success is indicated by `100 CONTINUE`, at which point the authentication continues to the PASSWORD command step.  


**LOGOUT**  
*Logs out of session*
Parameters: none  
Returns: 200 OK  
Possible Errors: None  

Returns the session to an unauthenticated state. This does not close the connection.


**MKDIR**  
*Creates a new folder*  
Parameters: folder path  
Returns: 200 OK  
Possible Errors: 401 UNAUTHORIZED, 403 FORBIDDEN, 408 RESOURCE EXISTS  
Example:  
```
C: MKDIR / e36556f2-07d0-4e98-b3db-043b1ff94292 58cca317-eae3-49b0-8ce3-1dd83eda9a9d
S: 200 OK
```

Create a workspace directory. The directory path is a standard Anselus server-side path which indicates the path to be created relative to the root of the workspace. The command works similarly to the UNIX command `mkdir -p`, which creates folders and parent folders as needed to ensure that the entire path exists. If the leaf already exists, `408 RESOURCE EXISTS` is returned.


**MOVE**  
*Moves an item from the selected path to another on the server*  
Parameters: source file, destination path  
Returns: 200 OK  
Possible Errors: 401 UNAUTHORIZED, 403 FORBIDDEN, 404 NOT FOUND, 408 RESOURCE EXISTS  
Example:  
```
MOVE 1577232538.29485.7c1d6aa5-cc79-474f-ac05-201147e80e53 / f736e26e-59ee-4c21-b044-80b4c37c7044 26bae0d2-6408-4f1a-b6fc-b43b9c3d01f0
```  

Moves an item. The item is expected to be in the current directory. The destination path is expected to be a standard Anselus server-side path to a folder. `404 NOT FOUND` is returned if the item does not exist. `404 RESOURCE EXISTS` is returned if an entry in the destination already exists with that name.  


**PASSWORD**  
*Submits password for authentication*  
Parameters: hash of user's password  
Returns: 100 CONTINUE  
Possible Errors: 402 AUTHENTICATION FAILURE, 405 TERMINATED

Continues PLAIN authentication. MUST be sent only if client receives 100 CONTINUE from a LOGIN command. If the hashes don't match, `402 AUTHENTICATION FAILURE` is returned and the session state returns to its original, unauthenticated state--after any PASSWORD failure, a successful LOGIN MUST be sent to be permitted to send another PASSWORD command. If the password failure max for the server is reached, `405 TERMINATED` is sent and the connection is closed. After the successful response, the PLAIN login process continues to the DEVICE phase.   


**PREREG**  
*Provisions a new workspace*  
Parameters: User ID (optional)  
Returns:  
	200 OK workspace_id registration_code opt_user_id  
Possible Errors: 408 RESOURCE EXISTS  
Example:  
```
C: PREREG  
S: 200 OK 78fac9f7-f352-433a-a2f2-480fa4f2506d ConsolingFreeingTipperUnissuedWaggleUproot  

C: PREREG csmith  
S: 200 OK 7d4b9e50-b924-4323-9c74-d15a15b64b43 OmitSwoonedSpriteRevengeSmasherRelatable csmith  
```

Requests the creation of a new workspace. Unlike `REGISTER`, this command is intended to be issued locally by an administrator from an authenticated state. This command may be used with any of the registration modes, although it is of limited utility for servers configured for public or network registration. It returns a workspace ID, a registration code, and if a user ID was supplied, the user ID originally requested. The workspace ID (or user ID) is given to the user along with the preregistration code. If a user ID was supplied by the command, it is returned, as well. The code is not a password--only a means of authenticating a user for registration without the administrator ever knowing the user's password. If a workspace ID is submitted as a user ID, `400 BAD REQUEST` is returned.  


**QUIT**  
*Request closing the connection*  
Parameters: none  
Returns: none  

Requests the server close the connection. The server does not return anything; instead the server closes the connection.


**REGCODE**  
*Logs in with preregistration information*  
Parameters: registration name, password hash, device ID, prefixed device public key  
Returns: 201 REGISTERED  
Possible Errors: 401 UNAUTHORIZED  
Example:  
```
C: REGCODE csmith OmitSwoonedSpriteRevengeSmasherRelatable 0cf4bd1c-08fe-49ff-9f19-359bc9e7af94 CURVE25519:)?!*s*y&)^?ie}qcsh~q*%AIKeZ<0`Wp66eJK(DV  
S: 201 REGISTERED  
```

This command registers an account with a preprovisioned registration code. The registration name can either be the workspace ID provided to the administrator by the `PREREG` command or by the optional user ID passed to it. The user provides either one along with the one-time-use registration code to the client application, which submits said information along with device identity info. Assuming that all the information matches, the server provisions the workspace the same way as with `REGISTER`. The registration code itself is a free-form string which may contain up to 128 UTF-8 non-whitespace code points. `401 UNAUTHORIZED` is returned if the registration combination is invalid. This command handles preregistration failures similar to  `LOGIN` failures: if the login failure limit is reached, `405 TERMINATED` is returned and the connection is closed.  


**REGISTER**  
*Creates a new workspace*  
Parameters: WID of workspace owner, password hash, device ID, prefixed device public key  
Returns:  
	Public mode: 201 REGISTERED  
	Network mode: 201 REGISTERED  
	Moderated mode: 101 PENDING  
	Private mode: 304 REGISTRATION CLOSED  
Example:  
```
C: REGISTER b9de5af4-a592-4210-b4db-843725fe0759 $argon2id$v=19$m=65536,t=2,p=1$WbKj86UxOAeSom71nKrAlw$uemRFB5eZ0D4TI13Mj6O7KRWsd2eXMjtWINbDao4cgM 244df58c-6eca-4384-af44-491d9084e1f0 CURVE25519:+?1C<<8Y9AV6pDPD@sgkiu(Qd2JIXjOF+Q&)S3-t
S: 201 REGISTERED

C: REGISTER b9de5af4-a592-4210-b4db-843725fe0759 $argon2id$v=19$m=65536,t=2,p=1$WbKj86UxOAeSom71nKrAlw$uemRFB5eZ0D4TI13Mj6O7KRWsd2eXMjtWINbDao4cgM 244df58c-6eca-4384-af44-491d9084e1f0 CURVE25519:+?1C<<8Y9AV6pDPD@sgkiu(Qd2JIXjOF+Q&)S3-t
S: 101 PENDING
```

Requests the creation of a new workspace. This command MAY be sent from unauthenticated or authenticated states. For public and network registration modes, success is returned unless a problem has been encountered. If network registration is used and the client is outside the permitted subnet, `304 REGISTRATION CLOSED` is returned. This response is also given if a registration request is made to a server with private registration. A successful request for moderated registration returns `101 PENDING`, indicating that the user must wait for the administrator to approve the request before login is possible, but the device is tentatively given a device ID. For servers utilizing private registration, this command is not used. Instead, an administrator uses the PREREG command while logged into the server locally to register accounts on users' behalf.  

Concerning the command parameters, the first is the requested workspace ID. In the unlikely event that the workspace ID already exists on the server, `408 RESOURCE EXISTS` is returned. In such an event, it is permissible -- and even expected -- that the client will generate a new workspace ID and resubmit. The device ID is simply another UUID. Finally, the device public key is a base85-encoded public encryption key with the algorithm used as a prefix.


**RESUME**  
*Finishes an upload*  
Parameters: direction, filename, byte offset, hash value  
Returns: as per UPLOAD  

This command continues a previous upload. If the previous remnant was somehow removed, 404 NOT FOUND is returned. Otherwise, this command operates as per UPLOAD.  


**SELECT**  
*Sets the current path for the session*  
Parameters: path  
Returns: 200 OK  

If the path does not exist or the path is not permitted, such as one which is out of the permitted filesystem area, `404 RESOURCE NOT FOUND` is returned. The path is a standard Anselus filesystem path.  


**SEND**  
*Sends an item to another server*  
Parameters: size in bytes, hash value, destination  
Returns: As per UPLOAD  

This command works exactly like UPLOAD except that it also takes a destination as a parameter. The destination is expected to be a numeric Anselus address, consisting of the workspace ID of the destination, a forward slash, and the destination domain. The message is enqueued for delivery after the item upload is complete.  


**SERVERID**  
*Enables a server to identify itself prior to delivering items*  
Parameters: fully-qualified domain name  
Returns: 200 OK (Encrypted one-time password)
Possible Errors: 308 DELIVERY DELAY NOT REACHED  

Servers who intend to deliver items must initiated a session with this command. The receiving server performs a DNS request for the TXT record containing the server's base64-encoded public key. If the receiving server is unable to obtain this record, it returns `306 KEY REQUEST FAILURE`. If the request succeeds but the server's FQDN is not part of the record, `403 FORBIDDEN` is returned. Otherwise, the receiving server generates a one-time random password, encrypts it, and returns 200 OK with the base85-encoded encrypted password. Should the sending server exceed the delivery failure rate configured on the receiver, the receiving server will send `308 DELIVERY DELAY NOT REACHED` in response to this command until the cooldown period has passed.


**SERVERPWD**  
*Enables a server to log in for item delivery*  
Parameters: (one-time password)  
Returns: 403 FORBIDDEN, 200 OK  

A server which intends to deliver items MUST send this command after receiving a 200 OK response from a SERVERID command. The sending server must take the encoded encrypted password, decrypt it, and send this command with the decrypted password. If the decrypted matches the one sent by the receiving server, `200 OK` is returned and the sending server may follow with DELIVER, QUIT, or RESUME commands. After this type of authentication, ONLY the commands DELIVER, QUIT, or RESUME may be sent--others MUST receive a `401 UNAUTHORIZED`. 


**SETADDR**  
*Sets the address for the workspace*  
Parameters: Anselus address  
Returns: 200 OK, 408 RESOURCE EXISTS, 400 BAD REQUEST   
Examples:  
```
C: SETADDR "Corbin Smith@example.com"
S: 400 BAD REQUEST

C: SETADDR csmith@example.com
S: 408 RESOURCE EXISTS

C: SETADDR CorbinSmith@example.com  
S: 200 OK
```

The user's full address is sent to the server. It may not contain whitespace. If the address submitted is not valid, `400 BAD REQUEST` is returned. If the address submitted belongs to another workspace, `408 RESOURCE EXISTS` is returned.  


**SETPASSWORD**  
*Updates the password for the workspace*  
Parameters: oldhash newhash  
Returns: 200 OK, 401 UNAUTHORIZED, 402 AUTHENTICATION FAILURE  

A client will send this command when the user wishes to update their password. It requires the hash of both the old and new passwords in order to process the request. This command may be sent ONLY during an authenticated session. A `401 UNAUTHORIZED` MUST be sent otherwise. If the old hash does not match the current password's hash, a `402 AUTHENTICATION FAILURE` response is sent and no password change is made.


**UNREGISTER**  
*Deletes a workspace*  
Parameters: hash of user's password, session ID  
Returns:  
	Public mode: 201 UNREGISTERED device_session_ID  
	Moderated mode: 101 PENDING  
	Network mode: 201 UNREGISTERED device_session_ID  
	Private mode: 101 PENDING  


**UPLOAD**  
*Upload an item to the temp folder*  
Parameters: size in bytes, prefixed hash value  
Returns: 200 OK filename  
Example:  
```
C: UPLOAD 4096 BLAKE2B-256:l%z_UaZC<q1so92vDA#*FCejWBLYfXj^@Py4(WYz  
S: 100 CONTINUE 1577232538.4096.a263fd0c-4de6-4644-9423-796bbe44eb50  
C: (data uploaded)  
S: 200 OK
```  

The client uploads data to a file on the server. First is the request for the upload, submitting the size of the upload in bytes, the hash function used, and the hash value computed on the client side. The size is expected to be accurate, as the data is treated as binary and will not be reformatted or otherwise modified. `409 QUOTA INSUFFICIENT` is returned if the workspace does not have sufficient space (or if the filesystem on the server lacks sufficient space). Once upload is complete, the server calculates the hash value of the data received, and if the value matches that sent by the client, `200 OK` is returned. If an error on the server side of the connection is experienced, `305 INTERRUPTED` is returned along with the size of the data received and the command is considered complete at that point. To finish the upload, a RESUME command must be performed.


### Status Codes

Most commands require the context of an authenticated login session. Attempts to use such a command outside of an authenticated session will result in a `401 UNAUTHORIZED` response. Likewise, if a user does not have sufficient permissions to execute a command or execute a command on a specific client item, `403 FORBIDDEN` is returned.


- 1xx: Info Codes
	- 100 Continue
	- 101 Pending
	- 102 Item
	- 103 Update
- 2xx: Success Codes
	- 200 OK
	- 201 Registered
	- 202 Unregistered
- 3xx: Server-Related Error Codes
	- 300 Internal Server Error
	- 301 Not implemented
	- 302 Server maintenance
	- 303 Server unavailable
	- 304 Registration closed
	- 305 Interrupted
	- 306 Key request failure
	- 307 Delivery failure limit exceeded
	- 308 Delivery delay not reached
	- 309 Encryption type not supported
- 4xx: Client-Related Codes
	- 400 Bad Request
	- 401 Unauthorized
	- 402 Authentication Failure
	- 403 Forbidden
	- 404 Not Found
	- 405 Terminated
	- 406 Payment Required
	- 407 Unavailable
	- 408 Resource Exists
	- 409 Quota Insufficient
	- 410 Hash Mismatch
	- 411 Bad Keycard Data