Specification for Gaia/OpenPGP project
============

## Goals of the project
As [Gaia's Email app](https://wiki.mozilla.org/Gaia/Email), is currently implemented, there is no provision for end-to-end encryption of email messages. The OpenPGP standard of email encryption offers a relatively popular method for email encryption that doesn't require a third-party certificate authority. We intend to create a patch for the Gaia Email app that offers a simple user-interface tightly integrated with the email user interface, for OpenPGP-compatible encryption, decryption, signing and verification of email messages, and a simple user interface for managing private and public PGP keys.

##Implementation
For simplicity, the initial version will provide "traditional" PGP/Inline formatting for encryption, with PGP/MIME as an optional extra.

For initial prototyping of the user interface, we plan to use the OpenPGP.js library. However, the license of OpenPGP.js is LGPL, and [Mozilla prefers to avoid the use of LGPL libraries in Gaia](https://groups.google.com/forum/#!topic/mozilla.dev.gaia/kWBOY1WzBrw). Therefore another javascript library (such as an OpenPGP implementation using the WebCrypto API) will need to be used in any version intended for deployment.

### Key management

PGP support will require local storage of public and private keys. Public keys should be associated with designated email addresses, while private keys should be locally encrypted and password-protected. Separate "signing" and "decryption" private keys should be maintained.

The availability of public keys should be displayed clearly in the contacts list of Gaia's Contacts app, adjacent to the email address. A "Lock" symbol can indicate the presence of a PGP public key. Next to each Lock, the public key fingerprint will be displayed, (perhaps using the [PGP Word List](https://en.wikipedia.org/wiki/PGP_word_list)), as well the associated expiration date.

### Encryption and signing

When the user is composing a message, the Gaia Email app should tailor its default behavior to the list of intended recipients. When the user has added recipients to the message (in the To, Cc, and/or Bcc fields) the app should check the public key store for the presence of an entry containing that email address, and draw a "Lock" symbol next to each recipient's name, indicating the intention to use PGP encryption for that recipient. When the user presses the "Send" button, the app should opportunistically encrypt the message with the recipient(s) associated public key(s). If a particular email address has multiple associated public keys, then the app should prompt the user with a choice of which public key to use when the user presses "Send".

Email messages with multiple recipients should be automatically encrypted, provided there is a public key available for each email address. If the user has provided multiple recipients, where some recipients have no associate public key, then the user should be prompted with a choice to either (1) Cancel Send or (2) Send unencrypted.

A notification should indicate to the user that encryption is taking place. 

### Decryption and authentication
