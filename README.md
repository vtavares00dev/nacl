# nacl
Fork of NACL

From http://nacl.cr.yp.to/index.html

NaCl (pronounced "salt") is a new easy-to-use high-speed software library for network communication, encryption, decryption, signatures, etc. NaCl's goal is to provide all of the core operations needed to build higher-level cryptographic tools.

From http://nacl.cr.yp.to/features.html

#Features

##C NaCl, C++ NaCl, and Python NaCl

The current version of NaCl supports C and C++. Support for Python is a high priority.
NaCl takes advantage of higher-level language features to simplify the APIs for those languages. For example:

* A message is represented in C NaCl as two variables: an array variable m and an integer variable mlen. Higher-level APIs use a single string variable m that knows its own length.
* The C NaCl functions return error codes to indicate, e.g., invalid signatures; a caller that neglects to check for those error codes will blithely proceed as if the signatures were valid. Higher-level APIs raise exceptions.
* The C NaCl functions write output strings via pointers. Higher-level APIs simply return the strings as function values.
High-speed implementations of high-speed primitives

NaCl aims to, and in many cases already does, provide record-setting speeds for each of its cryptographic operations.
This means not merely record-setting speeds for (e.g.) bulk data encryption with AES, but record-setting speeds for bulk data encryption with the best secret-key cipher available. Note that a state-of-the-art stream cipher such as Salsa20 is considerably faster than AES.

Sometimes NaCl includes slow implementations of primitives that are expected to set speed records with better implementations. Applications built using these NaCl functions will eventually benefit from higher-speed implementations.

##Automatic CPU-specific tuning

NaCl, like other speed-oriented cryptographic libraries, supports multiple implementations of the same function.
Most of these libraries attempt to recognize CPUs and select implementations (and compiler options) known to work well on those CPUs. NaCl instead compiles all implementations (with all specified compiler options) and uses speed measurements to select the fastest implementation for the user's CPU.

One advantage of NaCl's automatic tuning is that new CPUs automatically use the best option available, without any human intervention. Another advantage is that new implementations (and new compiler options) can be added with a minimum of fuss.

In theory, there is a compile-time cost for automatic tuning, and the cost grows with the number of implementations (and compiler options). In practice, the cost of NaCl's automatic tuning is quite small, and if it ever becomes troublesome then it can easily be split across many machines.

##Support for standard primitives

Whenever NaCl includes—for speed reasons or for security reasons—a newly proposed cipher, a newly proposed signature system, etc., it also includes an older standard cipher (e.g., AES), [TO DO:] an older standard signature system (e.g., ECDSA using the NIST P-256 elliptic curve), etc. Some users avoid new proposals as a matter of policy; NaCl accommodates those users.
This does not mean that it is a high priority for NaCl to support every cryptographic standard. Example: An implementation of triple DES might be of interest for bank-industry applications, but is a lower priority for NaCl than an improved implementation of AES.

##Expert selection of default primitives

Typical cryptographic libraries force the programmer to specify choices of cryptographic primitives: e.g., "sign this message with 4096-bit RSA using PKCS #1 v2.0 with SHA-256."
Most programmers using cryptographic libraries are not expert cryptographic security evaluators. Often programmers pass the choice along to users—who usually have even less information about the security of cryptographic primitives. There is a long history of these programmers and users making poor choices of cryptographic primitives, such as MD5 and 512-bit RSA, years after cryptographers began issuing warnings about the security of those primitives.

NaCl allows, and encourages, the programmer to simply say "sign this message." NaCl has a side mechanism through which a cryptographer can easily specify the choice of signature system. Furthermore, NaCl is shipped with a preselected choice, namely a state-of-the-art signature system suitable for worldwide use in a wide range of applications.

##High-level primitives

A typical cryptographic library requires several steps to authenticate and encrypt a message. Consider, for example, the following typical combination of RSA, AES, etc.:
* Generate a random AES key.
* Use the AES key to encrypt the message.
* Hash the encrypted message using SHA-256.
* Read the sender's RSA secret key from "wire format."
* Use the sender's RSA secret key to sign the hash.
* Read the recipient's RSA public key from wire format.
* Use the recipient's public key to encrypt the AES key, hash, and signature.
* Convert the encrypted key, hash, and signature to wire format.
* Concatenate with the encrypted message.
Sometimes even more steps are required for storage allocation, error handling, etc.
NaCl provides a simple crypto_box function that does everything in one step. The function takes the sender's secret key, the recipient's public key, and a message, and produces an authenticated ciphertext. All objects are represented in wire format, as sequences of bytes suitable for transmission; the crypto_box function automatically handles all necessary conversions, initializations, etc.

Another virtue of NaCl's high-level API is that it is not tied to the traditional hash-sign-encrypt-etc. hybrid structure. NaCl supports much faster message-boxing solutions that reuse Diffie-Hellman shared secrets for any number of messages between the same parties.

A multiple-step procedure can have important speed advantages when multiple computations share precomputations. NaCl allows users to split crypto_box into two steps, namely crypto_box_beforenm for message-independent precomputation and crypto_box_afternm for message-dependent computation.

##No data-dependent branches

The CPU's instruction pointer, branch predictor, etc. are not designed to keep information secret. For performance reasons this situation is unlikely to change. The literature has many examples of successful timing attacks that extracted secret keys from these parts of the CPU.
NaCl systematically avoids all data flow from secret information to the instruction pointer and the branch predictor. There are no conditional branches with conditions based on secret information; in particular, all loop counts are predictable in advance.

This protection appears to be compatible with extremely high speed, so there is no reason to consider weaker protections.

##No data-dependent array indices

The CPU's cache, TLB, etc. are not designed to keep addresses secret. For performance reasons this situation is unlikely to change. The literature has several examples of successful cache-timing attacks that used secret information leaked through addresses.
NaCl systematically avoids all data flow from secret information to the addresses used in load instructions and store instructions. There are no array lookups with indices based on secret information; the pattern of memory access is predictable in advance.

The conventional wisdom for many years was that achieving acceptable software speed for AES required variable-index array lookups, exposing the AES key to side-channel attacks, specifically cache-timing attacks. However, the paper "Faster and timing-attack resistant AES-GCM" by Emilia Käsper and Peter Schwabe at CHES 2009 introduced a new implementation that set record-setting speeds for AES on the popular Core 2 CPU despite being immune to cache-timing attacks. NaCl reuses these results.

##No dynamic memory allocation

C NaCl is intended to be usable in environments that cannot guarantee the availability of large amounts of heap storage but that nevertheless rely on their cryptographic computations to continue working. C NaCl functions do not call malloc, sbrk, etc. They do use small amounts of stack space; these amounts will eventually be measured by separate benchmarks.
This feature applies only to C NaCl. Higher-level languages such as Python are not currently usable in restricted environments.

##No copyright restrictions

All of the NaCl software is in the public domain.
