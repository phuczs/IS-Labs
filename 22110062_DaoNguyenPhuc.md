# Task 1:Public-key based authentication 

**Question 1**:
Implement public-key based authentication step-by-step with openssl according the following scheme.

**Answer 1**:

I use Git Bash to implement all OpenSSL commands

## Step 1:
Open Git Bash and create 2 directory `Client` And `Server`
![alt text](/img1.png)

## Step 2:Generate key pairs
In `Server`'s directory:

Generate `Server`'s private key
```sh
 openssl genrsa -out server_private.pem 2048
```

Generate `Server`'s public key
```sh
 openssl rsa -in server_private.pem -pubout -out server_public.pem
```

In `Client`'s directory:
Generate `Client`'s private key
```sh
 openssl genrsa -out client_private.pem 2048
 ```
Generate `Server`'s public key
```sh
 openssl rsa -in client_private.pem -pubout -out client_public.pem
```

## Step 3:Sever creates a challenge
Switch to `Server`'s directory

Create a challenge message and save it in a text file
```sh
 echo "hello,this is a challenge message">challenge.txt
```

Encrypt the challenge using the client's public key
```sh
 openssl pkeyutl -encrypt -inkey ../client/client_public.pem -pubin -in challenge.txt -out challenge_encrypted.bin
```

Simulate sending the challenge to the client by moving the encrypted file
```sh
 mv challenge_encrypted.bin ../client/
 ```

## Step 4: Client Decrypts and Signs the Challenge

Switch to `Client`'s directory


Decrypt the challenge using the client's private key
```sh
 openssl pkeyutl -decrypt -inkey client_private.pem -in challenge_encrypted.bin -out challenge_decrypted.txt
```

Sign the decrypted challenge using the client's private key
```sh
openssl dgst -sha256 -sign client_private.pem -out challenge_signed.bin challenge_decrypted.txt
```
Simulate sending the signed challenge back to the server by moving the signed file
```sh
 mv challenge_signed.bin ../server/
 ```

 ## Step 5: Server Verifies the Signed Challenge
 Switch to `Server`'s directory

 Verify the signature using the client's public key:
 ```sh
 openssl dgst -sha256 -verify ../client/client_public.pem -signature challenge_signed.bin challenge.txt
```
if verify  is valid,you will see:
```sh
Verified OK
```

# Task 2: Encrypting large message 
Create a text file at least 56 bytes.

**Question 1**:
Encrypt the file with aes-256 cipher in CFB and OFB modes. How do you evaluate both cipher as far as error propagation and adjacent plaintext blocks are concerned. 

**Answer 1:**

## 1.Create file named `large_message.txt`
```sh
 echo "This is a sample text file with more than fifty-six bytes for encryption." > large_message.txt
```

Verify the file's size:
```sh
ls -l large_message.txt
```

## 2. Encrypt the File with AES-256

Generate a 256-bit (32-byte) encryption key
```sh
 openssl rand -hex 32 > aes_key.key
 ```

Generate a 128-bit (16-byte) IV
```sh
 openssl rand -hex 16 > aes_iv.iv
 ```

**Encrypt in CFB Mode**:

Use the AES-256 cipher with CFB mode to encrypt the file
```sh
 openssl enc -aes-256-cfb -in large_message.txt -out large_message_cfb.enc -K $(cat aes_key.key) -iv $(cat aes_iv.iv)
```

Decrypt to verify
```sh
 openssl enc -aes-256-cfb -d -in large_message_cfb.enc -out decrypted_cfb.txt -K $(cat aes_key.key) -iv $(cat aes_iv.iv)
```

Compare the decrypted file with the original:
```sh
diff large_message.txt decrypted_cfb.txt
```

**Encrypt in OFB Mode**:

Use the AES-256 cipher with OFB mode to encrypt the file
```sh
 openssl enc -aes-256-ofb -in large_message.txt -out large_message_ofb.enc -K $(cat aes_key.key) -iv $(cat aes_iv.iv)
 ```

Decrypt to verify
```sh
openssl enc -aes-256-ofb -d -in large_message_ofb.enc -out decrypted_ofb.txt -K $(cat aes_key.key) -iv $(cat aes_iv.iv)
```

Compare the decrypted file with the original:
```sh
 diff large_message.txt decrypted_ofb.txt
```

## 3.Evaluate CFB and OFB Modes

The basic idea of `CFB` is to use the encryption of the IV as a one-time pad for the first block, and use the encryption of the first ciphertext block as a one-time pad for the second block, and so on: if the plaintext is P=P1∥P2∥P3∥⋯
, then the ciphertext is C=C1∥C2∥C3∥⋯
, where
```sh
C1=P1⊕Ek(IV), C2=P2⊕Ek(C1), C3=P3⊕Ek(C2),......
```

The basic idea of `OFB` is to use the encryption of the IV as a one-time pad for the first block, and use the encryption of the first pad block as a one-time pad for the second block, and so on:
```sh
C1=P1⊕Ek(IV), C2=P2⊕Ek(Ek(IV)), C3=P3⊕Ek(Ek(Ek(IV))),......
```

**Question 2**:
Modify the 8th byte of encrypted file in both modes (this emulates corrupted ciphertext).
Decrypt corrupted file, watch the result and give your comment on Chaining dependencies and Error propagation criteria.

**Answer 2**:

## 1. Modify the 8th Byte of the Encrypted File
We will use the `xxd` command to modify the 8th byte of the ciphertext for both files (`large_message_cfb.enc` and `large_message_ofb.enc`).

**Modify the 8th Byte in CFB Mode Encrypted File**

Open the encrypted file in hexedit mode using `xxd`
```sh
xxd large_message_cfb.enc > cfb_corrupted.hex
```
Edit the 8th byte:

Open `cfb_corrupted.hex` in a text editor 
```sh
nano cfb_corrupted.hex
```
I changed the 8th byte from `89a6` to `89f6`
![alt text](/img2.png)

Convert the modified hex file back to binary
```sh
xxd -r cfb_corrupted.hex > large_message_cfb_corrupted.enc
```

**Modify the 8th Byte in OFB Mode Encrypted File**

Repeat the same process for the OFB-encrypted file
```sh
xxd large_message_ofb.enc > ofb_corrupted.hex
```
Edit the 8th byte as described above, then convert it back to binary
```sh
xxd -r ofb_corrupted.hex > large_message_ofb_corrupted.enc
```
## 2.Decrypt the Corrupted Files

Decrypt the Corrupted CFB File
```sh
openssl enc -aes-256-cfb -d -in large_message_cfb_corrupted.enc -out decrypted_cfb_corrupted.txt -K $(cat aes_key.key) -iv $(cat aes_iv.iv)
```
View the output
![alt text](/img3.png)

The output is decrypted at 8th byte.

**Decrypt the Corrupted OFB File**

Use OpenSSL to decrypt the corrupted OFB file
```sh
openssl enc -aes-256-ofb -d -in large_message_ofb_corrupted.enc -out decrypted_ofb_corrupted.txt -K $(cat aes_key.key) -iv $(cat aes_iv.iv)
```
Then view the output
![alt text](/img4.png)

## 3.Analyze and Compare the result

`CFB Mode` (Cipher Feedback Mode):

-Chaining Dependency: CFB uses feedback chaining, so errors are limited to one block of plaintext.

-Error Propagation: Localized. Corruption in one ciphertext byte corrupts the corresponding plaintext byte and possibly one block at most.

`OFB Mode` (Output Feedback Mode):

-Chaining Dependency: OFB does not use chaining; it generates a keystream independent of the plaintext.

-Error Propagation: Minimal. Corruption in one ciphertext byte only corrupts the corresponding plaintext byte.

