---
date: 2019-09-09T19:07:42-04:00
draft: false
---
## Anselus Contact Information and Address Books

**Status:** Draft (incomplete)  
**Submission Date:** September 9, 2019  
**Submitted By:** Jon Yoder <jsyoder@mailfence.com>  
**Abstract:** Anselus message structure  

### Changelog

0.0.1: Initial submission

### Sections
- [Structure](#structure)
- [Field Definitions](#field-definitions)
- [Personal Information Profiles](#pips)
- [Updates and Annotations](#updates_and_annotations)

### Description

Contact information and address book synchronization are part of the reason that Anselus is a platform and not merely a new e-mail application. Like other bits of information, each contact is contained in an individual file.

<a name="structure"></a>
### Structure

An Anselus contact file is a [Client Item] with a payload specific to storing contact information. The overall payload structure is that of a list of dictionaries containing contact information. Although it is most common to store the information for one contact, there is support for multiple contacts per client item file. Below is listed an example of a profile which utilizes most of the available fields. Most of these fields map directly to those found in the [vCard standard]. However, unlike the vCard standard, almost every field is optional so that an Anselus workspace account can be maintained with the only identifying information for the account being its numeric address. However, in the interest of cataloguing information for contacts who do not have an Anselus address, even this field is technically not required.

[Client Item]: /spec/serverside/clientserver
[vCard standard]: https://tools.ietf.org/html/rfc6350

**Example:**  
```json
[
	{
		"Version" : "1.0",
		"Sensitivity" : "Private",
		"EntityType" : "individual",
		"Source" : "owner",
		"Update" : "no",
		"ID" : "f81d9ec8-01a5-442e-80f4-09db2c7d4a36",
		"Name" : {
			"Formatted" : "Richard Brannan",
			"Nicknames" : [
				"Rick"
			],
			"Given" : "Richard",
			"Family" : "Brannan",
			"Additional" : [
				"Michael"
			],
			"Prefixes" : [
				"Mr."
			],
			"Suffixes" : [
				"Ph. D"
			]
		},
		"Gender" : "Male",

		"MailingAddresses" : {
			"Work" : {
				"StreetAddress" : "1013 Hickman St.",
				"ExtendedAddress" : "Suite D",
				"Locality" : "Bensenville",
				"Region" : "Illinois",
				"PostalCode" : "60106",
				"Country" : "United States"
			},
			"Home" : {
				"POBox" : "4315",
				"Locality" : "Bensenville",
				"Region" : "Illinois",
				"PostalCode" : "60106",
				"Country" : "United States"
			},
			"Other" : [
			]
		},

		"phone" : {
			"Home" : "555-555-1234",
			"Work" : "555-555-5678",
			"Mobile" : "555-555-9090"
		},

		"Anselus" : {
			"Home" : {
				"UserID" : "cavs4life",
				"Preferred" : "yes",
				"Workspace" : "e3e98b45-b226-4926-b4ea-69ab16dd035e",
				"Domain" : "anselus-example.com",
				"Keys" : {
					"Primary" : {
						"KeyHash" : "BLAKE2B-256:hf;9nd=_ucTPkRisv$!+^i1)A#WLUr09ji3p72E(",
						"Value" : "CURVE25519:!u>#AhMqIC!?b5>LZwl}Vf{Aw2?+`>cMK@OFzEGp"
					},
					"broadcast" : {
						"KeyHash" : "BLAKE2B-256:J=G?Q=(N-G4291qY52MM2IzhAwNog7S?RfZu9T;g",
						"Value" : "XSALSA20:{^ovBGUU<+;))|hPZtYgEs)NRcZbe~K5*l=x^r)w"
					}
				}
			}
		},

		"Anniversary" : "20001004",
		"Birthday" : "19900415",
		"Email" : {
			"Work" : "rbrannan@contoso.com"
		},

		"Organization" : {
			"Name" : "Acme Widgets, Inc.",
			"units" : "Administration; Finance; "
		},

		"Title" : "Chief Financial Officer",

		"Categories" : [
			"Executive"
		],

		"Website" : "https://www.example.com",

		"Photo" : {
			"MimeType" : "image/png",
			"Data" : "iBL{Q4GJ0x0000DNk~Le0000A0000A2nGNE0F5%wy#N3J1am@3R0s$N2z&@+hyVZp7)eAyR2Y?G{Qv*|e+D7|6ETWL6;e+j0BM>85Q>cpXaE2J07*qoM6N<$f&"
		},

		"Languages" : [
			"en"
		],
		"Notes" : "Hobbies: chainsaw carving, free climbing, underwater basket weaving"
	},
	{
		"Version" : "1.0",
		"Sensitivity" : "Private",
		"EntityType" : "individual",
		"Source" : "client",
		"ID" : "f81d9ec8-01a5-442e-80f4-09db2c7d4a36",
	}
]
```

<a name="field-definitions"></a>
### Field Definitions

**Version**: REQUIRED. API version of the payload.  

**Sensitivity**: REQUIRED. How sensitive the contact information is. This may be `public`, `private`, or `secret`. This field has no vCard equivalent.  

**EntityType**: REQUIRED. `entitytype` maps to the vCard field `KIND`. Values are `group`, `individual` (the default), or `org`. The `member` field (listed below) is required for the `org` type and optional for `group`.   

**Source**: REQUIRED. This field denotes the origin of the information. `owner` means that the information is updated by the entity itself -- updates from the entity are sent to keep this information current. `client` means that the user's client, not the owner, maintains this information. More information about this field and how the mechanism works can be found in the section [Updates and Annotations](#updates_and_annotations).

**ID**: CONDITIONAL. `id` is just a unique identifier created by the client to link multiple entries, such as for user annotations. It is a required field in client items on a user's account, but it is never transmitted for any purpose.  

**Name:Formatted**: OPTIONAL. `formatted` maps to the vCard field `FN`. This field is the full formatted version of the entity's name, including prefixes and suffixes.  

**Name:Nicknames**: OPTIONAL. `nicknames` maps to the vCard field `NICKNAME`.  

**Name:Given**:  OPTIONAL. The primary name for an entity. In many cultures, this is an individual's first name.    

**Name:Family**: OPTIONAL. The family name for an entity.  

**Name:Additional**: OPTIONAL. A list of additional names for the entity.  In English-speaking countries, this is generally an individual's middle name(s) or initial.  

**Name:Prefixes**: OPTIONAL. The prefix for an entity. For individuals in the United States, this translates to "Dr", "Mr", "Miss", etc.  

**Name:Suffixes**: OPTIONAL. Suffixes for an entity, such as "Esq." or "MD".  

**Gender**: OPTIONAL. `gender` maps to the vCard `GENDER` field's gender identity component, which is a free-form text field.  

**MailingAddresses**: OPTIONAL. This group contains a dictionary of field groups. Each group in this field contains fields which map to corresponding parameters of the vCard field `ADR`. The name of each group does not have a vCard equivalent, but is used to denote the type of mailing address, such as "Home" or "Work". The mappings of these fields are explained in relation to U.S. mailing addresses merely for the sake of clarity. `pobox` is for postal office boxes. `streetaddress` contains the street address. Apartment or suite numbers should use `extendedaddress` and not included in `streetaddress`. When in doubt, consult the postal organization for a particular country for how these two fields should be used. `locality`, `region`, and `postalcode` map to the city, state, and ZIP code for a U.S. address. `country` is used for the country for an address.  

**Phone**: OPTIONAL. This field contains a list of key-value pairs containing the name of a phone number, such as "Fax" or "Mobile". Note that the vCard field `TEL` roughly maps to this, as the names of the phone numbers are not rigidly defined, unlike the types in the vCard standard. An asterisk ('*') MAY be prefixed to a name to indicate the preferred contact number.  

**Anselus**: OPTIONAL. This field contains a list of field groups containing the components of the contact's Anselus addresses. `userid` contains the friendly part of the address. `workspace` contains the UUID numeric identifier used for the entity's account. `domain` contains the fully-qualified domain. If `userid` is empty, then the client is expected to display the numeric address and domain, separated by a forward slash, e.g. 'cavsfan4life/anselus-example.com' or '5ccc9ba6-9d4e-47d0-9c57-11ade969a88b/anselus-example.com'. `preferred` denotes whether the address is the owner's preferred address. The `anselus` field group is not required, but if it is present, all of its subfields are required to be present.  

**Anselus:Keys**: CONDITIONAL. This field group list contains the contact's Anselus encryption keys. Each key is named by its purpose. These are currently `signing`, `primary`, `social`, or `broadcast`. It is a required part of the `anselus` field group.  

**Anselus:Keys:Name:KeyHash**: CONDITIONAL. This field contains the hash of the encryption key. The hash is Base85-encoded and prefixed by the hashing algorithm. It is a required part of the `anselus` field group.  

**Anselus:Keys:Name:Value**: CONDITIONAL. This field contains the actual encryption key data. For public-key encryption, this is the contact's public key. It is Base85-encoded and prefixed by the algorithm used. It is a required part of the `anselus` field group.  

**Anniversary**: OPTIONAL. `anniversary` maps to the vCard field `ANNIVERSARY`. This is the date of marriage or equivalent for the entity. Format is YYMMDD.  

**Birthday**: OPTIONAL. `birthday` maps to the vCard field `BDAY`. The birth date of the entity. Format is YYMMDD.  

**Email**: OPTIONAL. This field contains a list of key-value pairs containing the name of the e-mail address and the address itself. Each entry in `email` maps an individual vCard `EMAIL` field. An asterisk ('*') MAY be prefixed to a name to indicate the preferred contact address.  

**Organization**: OPTIONAL. `organization` maps to the vCard `ORG` field. Contents of the field are one or more semicolon-separated levels of the units within the organization.

**Title**: OPTIONAL. `title` maps to the vCard `TITLE` field. It contains the title or job position of the entity.  

**Categories**: OPTIONAL. `categories` maps to the vCard `CATEGORIES` field. It contains a list of string values for tags to apply to the entity.  

**Website**: OPTIONAL. `website` specifies the URL of a website for the entity and maps to the vCard field `WEBSITE`.  

**Photo**: OPTIONAL.  

**Photo:Mime**: CONDITIONAL. This field is REQUIRED if the `photo` field group is to be used. It contains the MIME type of the data stored in the `data` field. Anselus clients MUST support `image/png` and `image/jpg` display. They SHOULD also support WEBP, HEIF, and SVG. Support for other formats is optional, but support for animated profile photos is discouraged.    

**Photo:Data**: CONDITIONAL. This field is REQUIRED if the `photo` field group is to be used. The data in this field MUST be no larger than 500KiB before encoding is applied.

**Languages**: OPTIONAL. `languages` roughly maps to the vCard `LANG` field. It is a list of languages used in communications with the entity. The languages are listed in order of preference from most preferred to least. The codes themselves MUST follow the format established in the [ISO 639-3] standard.

[ISO 639-3]: https://en.wikipedia.org/wiki/ISO_639-3

**Notes**: OPTIONAL. Contains miscellaneous text notes stored in [AnTM format](/spec/clientside/antm). This field MAY NOT contain any attachment-type data -- it MUST contain only text -- but it MAY contain any other kind of AnTM-permitted data, such as links or tables.

**Attachments**: OPTIONAL. This field group contains miscellaneous data intended to be associated with the entity.  

**Attachments:Name**: CONDITIONAL. This field is REQUIRED if the `attachments` field is used. It contains the name of the attached data. This name can be a file name, but is not required to be.  

**Attachments:Mime**: CONDITIONAL. This field is REQUIRED if the `attachments` field is used. It contains the MIME type of the encoded data.  

**Attachments:Data**: CONDITIONAL. This field is REQUIRED if the `attachments` field is used. It contains the actual base85-encoded data of the attachment.  

<a name="pips"></a>
### Personal Information Profiles

Individuals and organizations alike have certain contact information which they share freely and other contact information which is more carefully guarded. Personal Information Profiles enable a user to choose easily and quickly which information is shared. Each PIP has an information sensitivity class and a name. The name is chosen by the user and can be something as simple as "Family" or "Private". The information sensitivity class is limited to `public`, `private`, or `secret`.

`public` - Information permitted to be visible by essentially anyone. Name, gender, and Anselus address belong to this class by default.

`private` - Information that is more carefully controlled. Contact fields not listed above for the `public` profile are private by default.

`secret` - Information that must be explicitly shared. This information sensitivity class does not have any default fields, but does exist for users to be able to protect information deemed sensitive.

PIPs make information control simple. Contact Request Initiation (Stage 1) messages only send `public` class information by default, but users may customize the request and add `private` class information. `secret` class information is not permitted in these messages. Contact Request Acknowledgement (Stage 3) messages give the user the option to add information from one of their other profiles. This reponse message automatically sets the `sensitivity` field to sensitivity class of the profile chosen. For example, if a user has a `private`-class "Family" profile, the contact information in the Acknowledgement message will be set to `private`.

Profiles can also be customized. For example, a user may have a Public profile which includes mailing address. In this case, all Contact Request Initiation (Stage 1) messages will be sent including the user's mailing address. Certain information may not be added to the public profile, however: encryption keys except the primary encryption and signing keys are required to be at least `private` class.

<a name="updates_and_annotations"></a>
### Updates and Client-Side Annotations

Anselus contact information is designed from the outset to always be up-to-date and places the responsibility on the information owner to keep it that way. This does, however, present a problem when the contact information is not complete or the user wishes to keep personal notes related to the contact. The solution lies in an information overlay accomplished through the contact's `source` field. A contact's client item may contain an additional entry using the same `id` field and the `source` set to `client`. In this secondary entry only the required fields of a contact must be present along with the `id` field. All other information is optional. If a contact's entry only contains client-owned information and later owner-sourced information is added, the owner-sourced information takes priority and the existing client-owned data is converted into an annotation.	

Information updates are sent whenever users update their contact information. These updates only send the changes. Fields which are deleted are sent with empty data.
