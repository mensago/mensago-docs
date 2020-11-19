---
title: ""
date: 2020-10-22T21:09:00-04:00
draft: true
---
### Anselus Offline Messaging

- [Introduction](#intro)

------

<a name="intro"></a>
### Introduction

Because great effort has been made to reduce trust in infrastructure as far as possible, it stands to reason that underlying transport mechanisms don't matter. Nothing could be further from the truth. The Anselus transport protocol has been designed to maximize privacy and to put controls in place which enable service providers to prevent bad behavior. There are situations where completely avoiding standard server transport is desired. This specification concerns these situations and tooling to accommodate them.

### Offline Message Format

```json
{
	"Type" : "offlinemessage",
	"Version" : "1.0",
	"From" : "3cb11ab3-5482-4154-8ca1-dfa1cc79371c/example.com",
	"Date" : "20201022 120000",
	"ThreadID" : "2280c1f2-d9c0-440c-a182-967b16ba428a",
	"Subject" : "Sample Offline Message",
	"Body" : "The body of an offline message could very well be empty",
	"Attachments" : [
		{
			"Name" : "foobar.png",
			"Type" : "image/png",
			"Data" : "iBL{Q4GJ0x0000DNk~Le0000A0000A2nGNE0F5%wy#N3J1am@3R0s$N2z&@+hyVZp7)eAyR2Y?G{Qv*|e+D7|6ETWL6;e+j0BM>85Q>cpXaE2J07*qoM6N<$f&"
		}
	]
}
```
