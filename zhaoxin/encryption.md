# Security Engine
Security Engine is to realize encryption, decryption and authentication.
Different block cipher mode are also need to support. 
![Encryption](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/encryption.png)
## Encryption
The algorithm we support are DES,TDES,AES,SM4. 
The main steps in encryption are rorate shift and xor.
And these steps may repeat many times.

## Block cipher mode
describes how to repeatedly apply a cipher's single-block 
operation to securely transform amounts of data larger than a block. 
Different block cipher mode decides different composition of 
inputs and outputs of different algorithm. 
For example the simplest of the encryption modes is the Electronic Codebook (ECB) mode. 
The message is divided into blocks, and each block is encrypted separately.
![ECB](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/ecb.jpg)
Another example is the Cipher Block Chaining (CBC) mode. 
In CBC mode, each block of plaintext is 
XORed with the previous ciphertext block before being encrypted.
![CBC](https://github.com/CaseyZhu/work_summary/blob/main/zhaoxin/image/cbc.jpg)
There are more block cipher mode such as PCBC,CTR,OFB...
## Authentication
The purpose of the authentication algorithm is to 
create a constant length code from a block of long message.
It can check if the data was revised during the transform.
We support SHA256,SM3.

## Summary
The encryption algorithm is simple, but some other factors need to take care. 
1) Hardware need support save/restore(very similar to the context switch in CPU), 
we may only finish part of the job and an interrupt raised. We need stop reading source data, 
and make sure all data in read buffer have used up, all data in write buffer need flush out, 
and the intermediate data should save into the register. When all these done give an ack to CPU.
2) Security Engine works at different clock domain with the memory, the CDC  need to take care.
[A very good blog](http://www.moserware.com/2009/09/stick-figure-guide-to-advanced.html)

