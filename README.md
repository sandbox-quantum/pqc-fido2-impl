# pqc-fido2-impl

In this project, we plan to implement PQC on FIDO2. Currently, we only consider discoverable credentials as they are more popular because users are not required to enter usernames while using passwordless authentication. We start our implementation with the assertion process. For that, we need to implement the following six steps:
1. implement PQC key generation and storing when a user registers the FIDO2 authenticator.
2. transmit the public PQC to the server. 
3. server parse it and stores it for verification.
4. during authentication, the authenticator signs the challenge with PQC's private key.
5. transmit the PQC signature to the server.
6. server verifies the PQC signature.

The next step is to implement PQC in the attestation process, which involves following additional steps, assuming we finished the assertion:
1. During registration, sign the registration response with PQC private key for self-attestation
2. transmit the PQC signature to the server
3. server verifies the signature.

The last step in this project to implement PQC KEM in the authenticatorClientPIN API.

FIDO2 terms for reference:

  - **Discoverable credentials and Resident key**: Historically discoverable credentials have been called "resident keys", and this terminology can still be found in aspects of the protocol. (For example the name of the rk option key comes from the term “resident key”.) However, the word “resident” conflated the concepts of being discoverable and being statefully maintained by the authenticator, when it’s only the former that is externally observable and thus important. Reference: https://fidoalliance.org/specs/fido-v2.1-ps-20210615/fido-client-to-authenticator-protocol-v2.1-ps-20210615.html

  - **Attestation**: An attestation signature is produced when a new public key credential is created via an authenticatorMakeCredential operation. An attestation signature provides cryptographic proof of certain properties of the authenticator and the credential. For instance, an attestation signature asserts the authenticator type (as denoted by its AAGUID) and the credential public key. The attestation signature is signed by an attestation private key, which is chosen depending on the type of attestation desired. 
For benchmarking PQC we can use a self attestation signature, which uses credential private key. 


  - **Assertion**: An assertion signature is produced when the authenticatorGetAssertion method is invoked. It represents an assertion by the authenticator that the user has consented to a specific transaction, such as logging in, or completing a purchase. Thus, an assertion signature asserts that the authenticator possessing a particular credential private key has established, to the best of its ability, that the user requesting this transaction is the same user who consented to creating that particular public key credential. It also asserts additional information, termed client data, that may be useful to the caller, such as the means by which user consent was provided, and the prompt shown to the user by the authenticator.