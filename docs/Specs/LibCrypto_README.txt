Entropy CRYPTOGRAPHY LIBRARY

-------------------------------------------------------------------------------

Purpose: To encrypt or decrypt a given section of RAM (the message) securely.

Algorithm: 2048 bit symmmetric key ARCFOUR variant (CipherSaber-2).

-------------------------------------------------------------------------------

Notes:

********************
EXTREMELY IMPORTANT:
********************

This library treats the key and data as octets, not words. The high 8 bits of
every word are ignored. This is in order not to add any overhead or modify data
outside of the message.

As such, if the key is made up of 128 words, it must be split up into
256 words, each with only its low 8 bits set. For example:

0x1234 0x5678 ... : WRONG

0x0012 0x0034 0x0056 0x0078 ... : CORRECT

It is up to the programmer how to implement this and where to store the
expanded key. The library doesn't want to accidentally overwrite anything by
creating an expanded key itself. All of this goes for messages as well. A
message must be expanded before it can be encrypted or decrypted, then
compressed afterwards if necessary.

***********************
INITIALIZATION VECTORS:
***********************

The algorithm is vunerable to attack unless an IV is used with every message.
LibRNG has been provided for this purpose. It generates a 16 byte IV
(16 words, see above). This is part of the key, so a 240 byte key can be
generated once outside 0x10c, such as on Random.org, and reused ad infinitum.
However, with each new message, a new IV must be generated.

For encryption, a random 16 byte IV should be generated and appended
(prepending could decrease security) to the 240 byte key. AFTER the message has
been encrypted, the IV should be prepended (not appended) to the ciphertext.

For decryption, the first 16 bytes (i.e., the IV) should be read from the
ciphertext and appended to the 240 byte key, which has been shared in advance.
Decryption should then proceed as normal, starting from right after the last
byte of the IV.

**********
REGISTERS:
**********

This library conforms to the coding standards and only clobbers the values
of the A, B, and C registers. IJXYZ are unchanged. The stack is preserved
exactly as well, even the iterations value pushed on it stays in place.

-------------------------------------------------------------------------------

How to Use:

1.  Load the library into RAM.

2.  Set the A register to the location of the beginning of the key in memory.
    Remember that it must be 256 words long and only the low 8 bits of each
    word will be read. This is done after processing the IV as described above.

3.  Set the B register to the first byte (a.k.a. word) in memory to be
    encrypted or decrypted. Except for how the IV is handled, encryption and
    decryption are identical operations.

4.  Set the C register to the last byte in memory to be encrypted or decrypted.
    Note that if B > C, the behavior of the program is undefined.

5.  Push a value of 20 or greater to the stack. This is a parameter in the key
    setup algorithm. The higher it is, the more secure the encryption, but the
    longer it takes to process data. The sender and receiver of encrypted data
    must have this same value, but there is no point in keeping it secret. Both
    parties should agree on the highest number above 20 that still allows for
    reasonable calculation time.

6.  Call lib_crypto_wrapper from the library's Public API.

7.  The data will now have been overwritten with its encrypted/unencrypted
    version. Nothing else outside the space taken up by the library should
    have changed.

8.  Do what you need to do with IV's.

9.  If you're done with the library, go to step 10. Otherwise, go to step 2.

10. Once you're completely finished with all cryptographic operations, call
    lib_crypto_cleanup. This prevents information about the state from
    leaking into the hands of adversaries. Also delete the key from RAM when it
    is no longer needed. However, keep it stored elsewhere to be used when it
    is required again.