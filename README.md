Specification for Gaia Email/OpenPGP project (Draft)
============

## Goals of the project
As [Gaia's Email app](https://wiki.mozilla.org/Gaia/Email), is currently implemented, there is no provision for end-to-end encryption of email messages. The OpenPGP standard of email encryption offers a relatively popular method for email encryption that doesn't require a third-party certificate authority. We intend to create a patch for the Gaia Email app that offers a simple user interface for managing private and public PGP keys, and user controls, tightly integrated with the standard email user interface, for OpenPGP-compatible encryption, decryption, signing and verification of email messages.

## Implementation
The initial version will provide "traditional" PGP/Inline formatting for encryption. Support for encrypted attachments and PGP/MIME can be added as time is available.

For initial prototyping of the user interface, we plan to use the OpenPGP.js library. However, the license of OpenPGP.js is LGPL, and [Mozilla prefers to avoid the use of LGPL libraries in Gaia](https://groups.google.com/forum/#!topic/mozilla.dev.gaia/kWBOY1WzBrw). Therefore another javascript library (such as an OpenPGP implementation using the WebCrypto API) will need to be used in any version intended for deployment.

Cryptographic operations should run in a separate "PGP" service process, which can be responsible providing error messages and password prompts. That will allow other apps to take advantage of the PGP functionality.

It's crucial that any deployed PGP implementation uses "cryptographic blinding" to avoid timing attacks, and should be as performant as possible, subject to security requirements.

### Key management

PGP support will require local storage of public and private keys. Public keys should be associated with designated email addresses, while private keys should be locally encrypted and password-protected. Separate "signing" and "decryption" private keys should be maintained.

The availability of public keys should be displayed clearly in the contacts list of Gaia's Contacts app, adjacent to the email address. A "Lock" symbol can indicate the presence of a PGP public key. Next to each Lock, the public key fingerprint will be displayed, (perhaps using the [PGP Word List](https://en.wikipedia.org/wiki/PGP_word_list)), as well the associated expiration date.

The Edit page for each entry in the contact list should include a place to add PGP keys. To add a key, the user will be able to "Import" the key from a key file, or "Enter" (typically paste) in the hexadecimal contents of a PGP key using a text field. It should be possible to add multiple PGP keys for a single email address.

The Email app's configuration settings should also offer a place for a user to generate a PGP key pair and associate it with an email address. The default key settings should be RSA, 2048 bits  (NIST recommends using at least 2048-bit and has deprecated 1024-bits. See page 5 of http://csrc.nist.gov/publications/nistpubs/800-131A/sp800-131A.pdf)

### Encryption and signing

When the user is composing a message, the Gaia Email app should tailor its default behavior to the list of intended recipients. When the user has added recipients to the message (in the To, Cc, and/or Bcc fields) the app should check the local public key store for the presence of an entry containing that email address, and draw a "Lock" symbol next to each recipient's name, indicating the intention to use PGP encryption for that recipient. When the user presses the "Send" button, the app should opportunistically encrypt the message with the recipient(s) associated public key(s). If a particular email address has multiple associated public keys, then the app should prompt the user with a choice of which public key to use when the user presses "Send". By default, the message should also be encrypted with the user's public key, so that it can be decrypted on the local phone or on another computer that shares the private key.

Email messages with multiple recipients should be automatically encrypted, provided there is a public key available for each email address. If the user has provided multiple recipients, where some recipients have no associate public key, then the user should be prompted with a choice to either (1) Cancel Send or (2) Send unencrypted.

Next, the user should be prompted to sign the message by entering a password associated with one of the local private keys designated for signing. A notification should indicate to the user that encryption and/or signing is taking place. Once signing and encryption is finished, the message should be sent and sending should be reported to the user in the Gaia Email app's standard manner. The Sent message should be saved, in ciphertext only, in the Gaia Email app's "Sent" folder. If the user enters that folder and attempts to open the message, then the user will be asked for a password required to decrypt the message.

If the user chooses to Reply to a previously encrypted message, and a public key for the Sender is available then the Compose interface should appear as usual with the original message quoted. The Reply message and the quoted message will then be encrypted before sending. If, on the other hand, a public key is not available, then the User should be prompted with a warning indicating "Public Key for replying to Sender is not available" and the choice to Continue or Cancel. If the user chooses to Continue, the original message should not be quoted to avoid sending it in plaintext.

Similarly, if the User chooses to Forward a decrypted message, a Warning should be displayed, indicating that "You may be forwarding a secret message. Are you sure?" with the option to Continue or Cancel.

### Decryption and authentication

When the User has received an encrypted message it will be decorated with a "Lock" icon to indicate that the message is encrypted. If the user chooses to open that message for reading, then a password prompt should appear, indicating the private key required for decryption. If the User enters the correct password, then the message should be decrypted if possible, or a Warning indicating decryption failure should be displayed to the user. Email messages that were received encrypted should be locally cached in ciphertext only, for better security.

By default, any decrypted messages encoded in HTML should not display images or otherwise cause the phone to connect to the web to retrieve further resources. Prevention of automated resource retrieval will help to prevent timing attacks.

Signed messages (either cleartext or decrypted) should be automatically verified whenever possible. The Email app will search the contacts for the appropriate public key, and authenticate the message. If the message is correctly authenticated, a green "Check mark" icon should appear next to the Sender's name and/or email address (in the "To" field). If authentication fails then a red "X" icon should appear instead, and the User should be prompted with a Warning, saying "Message Authentication Failed -- this email message may be have been altered or faked by a third party."

### Password policies

When the Mail app is entered, if the User requests the decryption of an incoming or cached message, a password prompt should appear. To avoid prompting the user too often, the entered password should remain in memory to allow promptless decryption of a certain number of messages (say, 20) or until the Mail app has exited. Then the password should be erased from memory and the user re-prompted for the password to decrypt the next message.




