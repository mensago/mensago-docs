---
date: 2020-02-014T18:18:55-04:00
draft: false
---
## Notetaking Specification

**Status:** Draft  
**Submission Date:** February 14, 2020  
**Submitted By:** Jon Yoder <jsyoder@mailfence.com>  
**Abstract:** Anselus note structure  

### Changelog

0.0.1: Initial submission

### Description

Although there are an abundance of online cloud-synchronized notetaking applications available, few open source solutions, if any, approach the richness and flexibility offered by services like OneNote Online or Evernote. These two services are very much proprietary. Considering the expressiveness available in regular messaging on the Anselus platform, it would be a major oversight to not adapt the messaging standard for notetaking.

### Payload Structures

#### Sample User Note

```json
{
	"Type" : "note",
	"Version" : "1.0",
	"Created" : "20200105 123357",
	"Updated" : "20200202 082412",
	"Title" : "Sample Note",
	"Body" : "When at the first I took my pen in hand\nThus for to write, I did not understand\nThat I at all should make a little book\nIn such a mode; nay, I had undertook\nTo make another; which, when almost done,\nBefore I was aware, I this begun.",
	"Tags" : [ "books", "public domain", "John Bunyan" ],
	"Group" : "default",
	"Attachments" : [
		{
			"Name" : "cover.png",
			"Type" : "image/png",
			"Encoding" : "base85",
			"Data" : "iBL{Q4GJ0x0000DNk~Le0000A0000A2nGNE0F5%wy#N3J1am@3R0s$N2z&@+hyVZp7)eAyR2Y?G{Qv*|e+D7|6ETWL6;e+j0BM>85Q>cpXaE2J07*qoM6N<$f&"
		}
	]
}
```

**Note Fields**

*Type* - REQUIRED. This field is required for all payloads. It defines the type of payload.  

*Version* - REQUIRED. The messaging API version.  

*Created* - REQUIRED. The UTC date and time of when the note was created. The date MUST be sent using [ISO 8601] basic (compact) format. The time MUST be sent in the format HHMMSS.  One space MUST be used to separate the two as in the following example: `YYMMDD HHMMSS`.  

*Updated* - REQUIRED. The time the note was last updated. This uses the same format as `created`.

*Title* - REQUIRED. A string up to 100 characters in length. The characters MUST be valid printable UTF-8 characters or a space. Note that while the field is required, the field itself MAY be empty.  

*Body* - REQUIRED. AnTM-formatted text of any length. Escapement of content for JSON compliance is required.  

*Tags* - OPTIONAL. A list of strings containing UTF-8 user-defined tags.  

*Group* - REQUIRED. The group in which the note may be found. The default value for `group` should be "default".   

*Attachments* - OPTIONAL. A list of dictionaries containing attached data using the same format as Anselus messages.

[ISO 8601]: https://en.wikipedia.org/wiki/ISO_8601

#### Note Content

With notes being merely cloud-synchronized text files, there isn't much to be said about them. The body content may contain UTF-8 encoded plain text or AnTM-formatted text.
