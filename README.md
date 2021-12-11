# Mensago Platform Documentation

This respository contains architectural design documents for the Mensago end-to-end encrypted communications platform. These should be considered canonical design documents for developers wishing to implement software for the platform. It may also be interesting for anyone wanting to learn more about how the platform works and for rationale behind design decisions. All documentation found here is found are in draft state and should be largely be considered complete. Draft documents will brought to final, complete state when related code reaches its first beta release.

## The Documents

For a high-level introduction to the platform, consult the [Platform Overview](https://github.com/mensago/mensago-docs/blob/master/Mensago%20Platform%20Overview.adoc).

Those wishing to use Mensago services to confirm a user's identity or utilize their cryptography keys should read about [Identity Services](https://github.com/mensago/mensago-docs/blob/master/Identity%20Services.adoc). Also helpful if you want to implemented [Passwordless Logins](https://github.com/mensago/mensago-docs/blob/master/Passwordless%20HTTP%20Auth.adoc) using Mensago.

The [Security](https://github.com/mensago/mensago-docs/blob/master/Security.adoc) document offers information for those interested in how Mensago handles different types of threats.

Other documents are described below:

- [Safe Formatted Text Markup (SFTM)](https://github.com/mensago/mensago-docs/blob/master/Safe%20Formatted%20Text%20Markup.adoc) - a rich text formatting system designed to perform the same kinds of tasks as HTML in e-mail without any of the security problems that go with it.
- [Client-Server API](https://github.com/mensago/mensago-docs/blob/master/Client-Server%20API.adoc) - commands used by Mensago clients to communicate with servers.
- [Contact Information](https://github.com/mensago/mensago-docs/blob/master/Contact%20Info%20and%20Address%20Books.adoc) and the [Messaging Specification](https://github.com/mensago/mensago-docs/blob/master/Messaging%20Specification.adoc) are both needed to send messages.
- [Notetaking Specification](https://github.com/mensago/mensago-docs/blob/master/Note%20Specification.adoc) for those wishing to implement a notes app for Mensago.
- The [Offline Messaging and File Transport](https://github.com/mensago/mensago-docs/blob/master/Offline%20Messaging%20and%20File%20Transport.adoc) specification is helpful for anyone wanting to implement a utility similar to [GNU Privacy Guard](https://gnupg.org/).

## Licensing

All documents found here are released under the [Creative Commons BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) license.
