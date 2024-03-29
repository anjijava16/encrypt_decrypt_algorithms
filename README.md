# encrypt_decrypt_algorithms
 encrypt and decrypt methods
 
 
 ---

title: Using the Java Cryptographic Extensions
layout: col-sidebar
author:
contributors:
tags: java, cryptography
auto-migrated: 1
permalink: /Using_the_Java_Cryptographic_Extensions

---

## Meta

The code included in this article has not been reviewed and should not
be used without proper analysis. If you have reviewed the included code
or portions of it, please post your findings back to the Java Project
mailing list or contact the [OWASP Java and JVM
Project](OWASP_Java_Project "wikilink") team.

## Overview

Java Cryptographic Extensions (JCE) is a set of Java API's which
provides cryptographic services such as encryption, secret Key
Generation, Message Authentication code and Key Agreement. The ciphers
supported by JCE include symmetric, asymmetric, block and stream
ciphers. JCE was an optional package to JDK v 1.2.x and 1.3.x. JCE has
been integrated into JDK v1.4.
JCE API's are implemented by Cryptographic Service Providers. Each of
these cryptographic service providers implements the Service Provider
Interface which specifies the functionalities which needs to be
implemented by the service providers. Programmers can plugin any Service
Providers for performing cryptographic functionalities provided by JCE.
J2SE comes with a default provider named SunJCE.

### Symmetric Encryption Algorithms provided by SunJCE

1.  DES - default keylength of 56 bits
2.  AES -
3.  RC2, RC4 and RC5
4.  IDEA
5.  Triple DES – default keylength 112 bits
6.  Blowfish – default keylength 56 bits
7.  PBEWithMD5AndDES
8.  PBEWithHmacSHA1AndDESede
9.  DES ede

### Modes of Encryption

1.  ECB
2.  CBC
3.  CFB
4.  OFB
5.  PCBC

### Asymmetric Encryption Algorithms implemented by SunJCE

1.  RSA
2.  Diffie-Hellman – default keylength 1024 bits

### Hashing / Message Digest Algorithms implemented by SunJCE

1.  MD5 – default size 64 bytes
2.  SHA1 - default size 64 bytes

# There are two general categories of key based algorithms:

# Symmetric encryption algorithms: 
   Symmetric algorithms use the same key for encryption and decryption. These algorithms, can either operate in block mode (which works on fixed-size blocks of data) or stream mode (which works on bits or bytes of data). They are commonly used for applications like data encryption, file encryption and encrypting transmitted data in communication networks (like TLS, emails, instant messages, etc.). 
 # Asymmetric (or public key) encryption algorithms: 
  Unlike symmetric algorithms, which use the same key for both encryption and decryption operations, asymmetric algorithms use two separate keys for these two operations. These algorithms are used for computing digital signatures and key establishment protocols. 
# To configure any basic encryption scheme securely, it's very important that all of these parameters (at the minimum) are configured correctly:

  Choosing the correct algorithm
  Choosing the right mode of operation
  Choosing the right padding scheme
  Choosing the right keys and their sizes
  Correct IV initialization with cryptographically secure CSPRNG
  It's very important to be vigilant about configuring all of these parameters securely. Even a tiny misconfiguration will leave an entire crypto-system open to attacks.

# Note: 
   To keep this discussion simple, I will discuss only algorithm-independent initializations of a Cipher. Unless you know what you are doing, let provider defaults do their job of configuring more algorithm-dependent configurations, like p and q values of the RSA algorithm, etc.

## Examples



### SecureRandom

SecureRandom class is used to generate a cryptographically strong pseudo
random number by using a PRNG Algorithm. The following are the
advantages of using SecureRandom over Random. 1. SecureRandom produces a
cryptographically strong pseudo random number generator. 2. SecureRandom
produces cryptographically strong sequences as described in [RFC 1750:
Randomness Recommendations for
Security](http://www.ietf.org/rfc/rfc1750.txt)

    package org.owasp.java.crypto;

    import java.security.SecureRandom;
    import java.security.NoSuchAlgorithmException;

    import sun.misc.BASE64Encoder;

    /**
     * @author Joe Prasanna Kumar
     * This program provides the functionality for Generating a Secure Random Number.
     *
     * There are 2 ways to generate a  Random number through SecureRandom.
     * 1. By calling nextBytes method to generate Random Bytes
     * 2. Using setSeed(byte[]) to reseed a Random object
     *
     */


    public class SecureRandomGen {

        /**
         * @param args
         */
        public static void main(String[] args) {
            try {
                // Initialize a secure random number generator
                SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG");

                // Method 1 - Calling nextBytes method to generate Random Bytes
                byte[] bytes = new byte[512];
                secureRandom.nextBytes(bytes);

                // Printing the SecureRandom number by calling secureRandom.nextDouble()
                System.out.println(" Secure Random # generated by calling nextBytes() is " + secureRandom.nextDouble());

                // Method 2 - Using setSeed(byte[]) to reseed a Random object
                int seedByteCount = 10;
                byte[] seed = secureRandom.generateSeed(seedByteCount);

                // TBR System.out.println(" Seed value is " + new BASE64Encoder().encode(seed));

                secureRandom.setSeed(seed);

                System.out.println(" Secure Random # generated using setSeed(byte[]) is  " + secureRandom.nextDouble());

            } catch (NoSuchAlgorithmException noSuchAlgo)
            {
                System.out.println(" No Such Algorithm exists " + noSuchAlgo);
            }
        }

    }

### AES Encryption and Decryption

    package org.owasp.java.crypto;

    import java.security.InvalidAlgorithmParameterException;
    import java.security.InvalidKeyException;
    import java.security.NoSuchAlgorithmException;
    import java.security.SecureRandom;

    import javax.crypto.BadPaddingException;
    import javax.crypto.Cipher;
    import javax.crypto.IllegalBlockSizeException;
    import javax.crypto.KeyGenerator;
    import javax.crypto.NoSuchPaddingException;
    import javax.crypto.SecretKey;
    import javax.crypto.spec.IvParameterSpec;

    import sun.misc.BASE64Encoder;

    /**
     * @author Joe Prasanna Kumar
     * This program provides the following cryptographic functionalities
     * 1. Encryption using AES
     * 2. Decryption using AES
     *
     * High Level Algorithm :
     * 1. Generate a AES key (specify the Key size during this phase)
     * 2. Create the Cipher
     * 3. To Encrypt : Initialize the Cipher for Encryption
     * 4. To Decrypt : Initialize the Cipher for Decryption
     *
     *
     */

    public class AES {
        public static void main(String[] args) {

            String strDataToEncrypt = new String();
            String strCipherText = new String();
            String strDecryptedText = new String();

            try {
                /**
                 * Step 1. Generate an AES key using KeyGenerator Initialize the
                 * keysize to 128 bits (16 bytes)
                 *
                 */
                KeyGenerator keyGen = KeyGenerator.getInstance("AES");
                keyGen.init(128);
                SecretKey secretKey = keyGen.generateKey();

                /**
                 * Step 2. Generate an Initialization Vector (IV)
                 *      a. Use SecureRandom to generate random bits
                 *         The size of the IV matches the blocksize of the cipher (128 bits for AES)
                 *      b. Construct the appropriate IvParameterSpec object for the data to pass to Cipher's init() method
                 */

                final int AES_KEYLENGTH = 128;  // change this as desired for the security level you want
                byte[] iv = new byte[AES_KEYLENGTH / 8];    // Save the IV bytes or send it in plaintext with the encrypted data so you can decrypt the data later
                SecureRandom prng = new SecureRandom();
                prng.nextBytes(iv);

                /**
                 * Step 3. Create a Cipher by specifying the following parameters
                 *      a. Algorithm name - here it is AES
                 *      b. Mode - here it is CBC mode
                 *      c. Padding - e.g. PKCS7 or PKCS5
                 */

                Cipher aesCipherForEncryption = Cipher.getInstance("AES/CBC/PKCS7PADDING"); // Must specify the mode explicitly as most JCE providers default to ECB mode!!

                /**
                 * Step 4. Initialize the Cipher for Encryption
                 */

                aesCipherForEncryption.init(Cipher.ENCRYPT_MODE, secretKey,
                        new IvParameterSpec(iv));

                /**
                 * Step 5. Encrypt the Data
                 *      a. Declare / Initialize the Data. Here the data is of type String
                 *      b. Convert the Input Text to Bytes
                 *      c. Encrypt the bytes using doFinal method
                 */
                strDataToEncrypt = "Hello World of Encryption using AES ";
                byte[] byteDataToEncrypt = strDataToEncrypt.getBytes();
                byte[] byteCipherText = aesCipherForEncryption
                        .doFinal(byteDataToEncrypt);
                // b64 is done differently on Android
                strCipherText = new BASE64Encoder().encode(byteCipherText);
                System.out.println("Cipher Text generated using AES is "
                        + strCipherText);

                /**
                 * Step 6. Decrypt the Data
                 *      a. Initialize a new instance of Cipher for Decryption (normally don't reuse the same object)
                 *         Be sure to obtain the same IV bytes for CBC mode.
                 *      b. Decrypt the cipher bytes using doFinal method
                 */

                Cipher aesCipherForDecryption = Cipher.getInstance("AES/CBC/PKCS7PADDING"); // Must specify the mode explicitly as most JCE providers default to ECB mode!!

                aesCipherForDecryption.init(Cipher.DECRYPT_MODE, secretKey,
                        new IvParameterSpec(iv));
                byte[] byteDecryptedText = aesCipherForDecryption
                        .doFinal(byteCipherText);
                strDecryptedText = new String(byteDecryptedText);
                System.out
                        .println(" Decrypted Text message is " + strDecryptedText);
            }

            catch (NoSuchAlgorithmException noSuchAlgo) {
                System.out.println(" No Such Algorithm exists " + noSuchAlgo);
            }

            catch (NoSuchPaddingException noSuchPad) {
                System.out.println(" No Such Padding exists " + noSuchPad);
            }

            catch (InvalidKeyException invalidKey) {
                System.out.println(" Invalid Key " + invalidKey);
            }

            catch (BadPaddingException badPadding) {
                System.out.println(" Bad Padding " + badPadding);
            }

            catch (IllegalBlockSizeException illegalBlockSize) {
                System.out.println(" Illegal Block Size " + illegalBlockSize);
            }

            catch (InvalidAlgorithmParameterException invalidParam) {
                System.out.println(" Invalid Parameter " + invalidParam);
            }
        }
    }

### Des Encryption and Decryption

    package org.owasp.crypto;

    import javax.crypto.KeyGenerator;
    import javax.crypto.SecretKey;
    import javax.crypto.Cipher;

    import java.security.NoSuchAlgorithmException;
    import java.security.InvalidKeyException;
    import java.security.InvalidAlgorithmParameterException;
    import javax.crypto.NoSuchPaddingException;
    import javax.crypto.BadPaddingException;
    import javax.crypto.IllegalBlockSizeException;

    import sun.misc.BASE64Encoder;

    /**
     * @author Joe Prasanna Kumar
     * This program provides the following cryptographic functionalities
     * 1. Encryption using DES
     * 2. Decryption using DES
     *
     * The following modes of DES encryption are supported by SUNJce provider
     * 1. ECB (Electronic code Book) - Every plaintext block is encrypted separately
     * 2. CBC (Cipher Block Chaining) - Every plaintext block is XORed with the previous ciphertext block
     * 3. PCBC (Propogating Cipher Block Chaining) -
     * 4. CFB (Cipher Feedback Mode) - The previous ciphertext block is encrypted and this enciphered block is XORed with the plaintext block to produce the corresponding ciphertext block
     * 5. OFB (Output Feedback Mode) -
     *
     *  High Level Algorithm :
     * 1. Generate a DES key
     * 2. Create the Cipher (Specify the Mode and Padding)
     * 3. To Encrypt : Initialize the Cipher for Encryption
     * 4. To Decrypt : Initialize the Cipher for Decryption
     *
     * Need for Padding :
     * Block ciphers operates on data blocks on fixed size n.
     * Since the data to be encrypted might not always be a multiple of n, the remainder of the bits are padded.
     * PKCS#5 Padding is what will be used in this program
     *
     */

    public class DES {
        public static void main(String[] args) {

            String strDataToEncrypt = new String();
            String strCipherText = new String();
            String strDecryptedText = new String();

            try{
            /**
             *  Step 1. Generate a DES key using KeyGenerator
             *
             */
            KeyGenerator keyGen = KeyGenerator.getInstance("DES");
            SecretKey secretKey = keyGen.generateKey();

            /**
             *  Step2. Create a Cipher by specifying the following parameters
             *          a. Algorithm name - here it is DES
             *          b. Mode - here it is CBC
             *          c. Padding - PKCS5Padding
             */

            Cipher desCipher = Cipher.getInstance("DES/CBC/PKCS5Padding"); /* Must specify the mode explicitly as most JCE providers default to ECB mode!! */

            /**
             *  Step 3. Initialize the Cipher for Encryption
             */

            desCipher.init(Cipher.ENCRYPT_MODE,secretKey);

            /**
             *  Step 4. Encrypt the Data
             *          1. Declare / Initialize the Data. Here the data is of type String
             *          2. Convert the Input Text to Bytes
             *          3. Encrypt the bytes using doFinal method
             */
            strDataToEncrypt = "Hello World of Encryption using DES ";
            byte[] byteDataToEncrypt = strDataToEncrypt.getBytes();
            byte[] byteCipherText = desCipher.doFinal(byteDataToEncrypt);
            strCipherText = new BASE64Encoder().encode(byteCipherText);
            System.out.println("Cipher Text generated using DES with CBC mode and PKCS5 Padding is " +strCipherText);

            /**
             *  Step 5. Decrypt the Data
             *          1. Initialize the Cipher for Decryption
             *          2. Decrypt the cipher bytes using doFinal method
             */
            desCipher.init(Cipher.DECRYPT_MODE,secretKey,desCipher.getParameters());
             //desCipher.init(Cipher.DECRYPT_MODE,secretKey);
            byte[] byteDecryptedText = desCipher.doFinal(byteCipherText);
            strDecryptedText = new String(byteDecryptedText);
            System.out.println(" Decrypted Text message is " +strDecryptedText);
            }

            catch (NoSuchAlgorithmException noSuchAlgo)
            {
                System.out.println(" No Such Algorithm exists " + noSuchAlgo);
            }

                catch (NoSuchPaddingException noSuchPad)
                {
                    System.out.println(" No Such Padding exists " + noSuchPad);
                }

                    catch (InvalidKeyException invalidKey)
                    {
                        System.out.println(" Invalid Key " + invalidKey);
                    }

                    catch (BadPaddingException badPadding)
                    {
                        System.out.println(" Bad Padding " + badPadding);
                    }

                    catch (IllegalBlockSizeException illegalBlockSize)
                    {
                        System.out.println(" Illegal Block Size " + illegalBlockSize);
                    }

                    catch (InvalidAlgorithmParameterException invalidParam)
                    {
                        System.out.println(" Invalid Parameter " + invalidParam);
                    }
        }

    }

# Reference Link
https://security.blogoverflow.com/2013/09/about-secure-password-hashing/
