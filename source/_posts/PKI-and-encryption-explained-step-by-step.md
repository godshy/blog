---
title: PKI and encryption explained step by step
date: 2024-07-18 11:40:47
tags: PKI, encryption, TLS
category: security
---

# PKI and data transfer encryption explained

It is import to understand the concept of data layer encryption and security. Let's understand how it works from scratch. I will explain this by using different scenarios. 

## Index
1. [Basic concept](#what-we-want-to-achieve)
2. [Scenario 1](#scenario-1-no-encryption)
3. [Scenario 2](#scenario-2-symmetric-encryption)
4. [Scenario 3](#scenario-3-asymmetric-encryption)
5. [Scenario 4](#scenario-4-digital-signature)
6. [Summary](#summary)
## What we want to achieve?
It is always import to understand why we need data layer encryption in the first place. Here is some points that we need to consider if we want to protect the data when sending it to remote peers.
- confidentiality: Only targeted user or device can read and decrypt the data you sent. If attacker want to decrypt your data, they cannot achieve this within the time that this message or data is valid. Confidentiality is achieved by __encryption algorithms__.
- integrity: You need to ensure that the data you sent is not alternated by third parties. Receiver can check to see if the data is original and the same as you sent. This is achieved by using __Hash algorithms__.
- Source verification: Sender of data can be verified. Not by forged or created by unknown person. This is achieved by __HMAC__, hash message authentication code.
- Deniability prevention: Sender of message or data cannot deny the fact that he sent this message. This is achieved by __digital signature__.

Next let's talk about how to encrypt and secure the connection between users.

## Scenario 1: no encryption
Imagine A want to send data to B, in which contains sensitive data. 
In first scenario, A is sending plain text, hackers can easily stole the data by capturing packet. This is, obviously, not safe.
![Scenario 1, A is sending plain text o B.](https://s2.loli.net/2024/07/18/6tMxpNq2OAoFT5a.png)

## Scenario 2: symmetric encryption
Let's consider a encryption method to protect data transportation.
There is one method called __symmetric encryption__, data is encrypted and decrypted by a so-called key. In symmetric encryption, both encryption/decryption is used by same key. So this doesn't solve the problem of data leakage. If attackers stole the key, they are also able to see the content of data.
![Scenario 2, symmetric encryption, but not solving problems](https://s2.loli.net/2024/07/18/FSnuHhs178GkKxe.png)

## Scenario 3: asymmetric encryption
To solve the problem in scenario 2, we need to think of a way to protect the key from being stolen while transferring to remote side.
Here we introduce __asymmetric encryption algorithm__. Asymmetric encryption use a private/public key pair, instead of one key to encrypt and decrypt data. You can either choose to use private key to encrypt and public key to decrypt or vice versa. In our case, A/B firstly generate their only private/public key-pair. Then encrypt the data using their private key and exchange the public key with others.
Data are decrypted by the corresponding public key.

The problem of asymmetric algorithm is that it has a huge compute complexity so using this method to encrypt  large data is time costing.
To mitigate the issue, we use asymmetric algorithm to encrypt/decrypt the symmetric key only. The data is still encrypted by symmetric key. Then the key itself is encrypted by private key of sender. Receiver received the encrypted symmetric key and the public key of sender, it uses the public key to decrypt the symmetric key, and use this key to decrypt the data. In this case, even if hacker get the public key, he cannot decrypt the encrypted symmetric key, thus not able to stole the data.
![Scenario 3, by using asymmetric and symmetric encryption, data are now secured](https://s2.loli.net/2024/07/18/mQ7xhypnv9uMS4I.png)

But hacker doesn't give up, in fact he still has some way to stole the data. Actually the method of exchanging public keys between each other, will cause source verification issue. That's what I mentioned in the first place. In other word, neither A nor B can figure out whether he is exchanging keys with proper peer. Hacker could be playing A or B, and we don't have a way to identify then. Also, we are not sure whether data is not tampered during transportation or not. Let's look two attack pattern below.

In first pattern, hacker is playing A and exchanging key with B, B actually received public key from hacker, but hacker claims the key is actually from A, B cannot verify it.

![Attack pattern 1. Hacker is playing role as A and send malicious data](https://s2.loli.net/2024/07/18/KvucmGXQnyFtji8.png)

In second pattern, hacker is playing B and exchanging key with A. A cannot verify whether the incoming public key is from B or others. By encrypting the data using the unknown public key, it is actually sending data to hacker.
![Attack pattern 2. Hacker is staling data from A by playing role as B](https://s2.loli.net/2024/07/18/oITn98uUgYlOft1.png)

## Scenario 4: digital signature
So how we fix this issue? You see, in real world, we use signature to 
identify the source of executer. You cannot deny what you have done with signature as evidence. We can use same method to identify the sender in our case, by using method so-called digital signature. To verify the source of sender, sender uses his private key to encrypt the hash of his data, hash of the data is called digest, and encrypted hash is called signature. Sender and receiver exchanges public key, then perform Data encryption and symmetric key encryption, when sender sends his encrypted data and key, he also sends a signature in which contains the hash result of the data encrypted by his private key. Receiver then do the same thing, decrypt data and symmetric key, but also decrypt the signature and compare with the hash between the sent data digest and signature digest.
![A uses his private key to do digital signature](https://s2.loli.net/2024/07/18/kdZbijFreGV7BgI.png)

![A send his digital signature to prove the data is sent to B not by others and data is not alternated](https://s2.loli.net/2024/07/18/EHZzMIxsUWX6tuL.png)

However, still one problem exists.. Since there is no way to verify the identity of key owner, everyone can claim they are the correct sender.. Signature can make sure data is not alternated while transporting. But every can claim that they are A using their own signature. 

In real world, every one's identity is proven by your government or a third party authority that everyone can believe into. Here we introduce a concept called CA (centralized authority) to deal with the identity problem. CA acts as a trusted third-party to help validate and verify the identity of each senders/receivers. 
CA uses his private key to create digest and prepare the certificate for each public key of end user. The certificate has specific format. But in general, it contains public key of the end-user, valid data and digital signature of CA.
Certificate is verified by CA's public key, if the digest matches, then senders' identity are verified.

![A here send a CSR(certificate sign request) to CA, CA signs this CSR using its private key. A can now use his signed certificate to identify himself.](https://s2.loli.net/2024/07/18/cKhp8g7CRjeBrLA.png)

However, how to verify CA is trusted? Not signed by anyone else? Actually everyone can setup its own CA and sign for himself. To prevent it, CA will self-sign a root certificate, and install on each users. Users use CA's public key to verify the identity of CA.
![CA also need to be trusted.](https://s2.loli.net/2024/07/18/wnUILl3hHWcZFiX.png)

![Generally, CA sign its own public key. And installed on end-users. For example, every PC would install CA cert when it was manufactured. CA cert is decrypted by CA public key](https://s2.loli.net/2024/07/18/T5R7HOtiSdYIKFl.png)

## Summary
To establish a secured connection between A and B, 
in which A's identity is verified by B and B's identity is verified by A. On A and B there should be these files existing below.
![A/B need to install CA cert, then they exchange their certificates.](https://s2.loli.net/2024/07/18/lRpKxEeHkhTUsr7.png)