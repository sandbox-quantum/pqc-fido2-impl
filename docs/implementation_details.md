# FIDO2 PQC

 **Goal**: Add PQC to FIDO2 protocol.

### Implementation Infrastructure
 - Authenticator
   - Solokey 
   - Solo: STM32L432 (C)
   - Solo2: NXP LPC55S69 (LPCXpresso55S69 development board) (Rust)
   - OpenSK
 - Server
   - Yubico: JavaWebAuthn Server 
 - PQC library
   - Liboqs (Has Go, Rust and Java wrappers)
   - Pqm4 - C (ARM Cortex-M4 family of microcontrollers)
 - Firefox browser: In case we will need to make some tweaks to the FIDO2 client,  it will be easier to do with firefox as it is open source. '


### PQC changes 
 - **Question**: The new hybrid algorithm we will implement, will not have a COSE identifier. Therefore, it would be hard to allow authenticator and RP to decide from a set of hybrid PQC. 
   - We can use a custom COSEidentifiers that is not in use for benchmarking purposes. 
   - We hardcode the PQC we use on both authentication and RP side. This will prevent the need of exchanging PQC cipher details.

1. *authenticatorMakeCredential* and *authenticatorGetAssertion* 
For both attestation and assertion signature, we need to add PQC signing on authenticator and verification on server end. 
   - authenticatorMakeCredential API returns an attStmt which contains (1) sig: signcredPrivateKey(authData||clientDataHash) if self attestation used. 
   - authenticatorGetAssertion API returns signcredPrivateKey(authData||clientDataHash)
   - Both of the above signatures needs to be verified on the server end. 

2. *authenticatorGetInfo*
   - Authenticator returns algorithms, which is a list of supported algorithms for credential generation (Array of PublicKeyCredentialParameters). PublicKeyCredentialParameters. It is a list of COSEAlgorithmIdentifier. If we use a custom COSEidentifier, we update them. Otherwise, it only returns classical algorithms.
   - remainingDiscoverableCredentials: Authenticator estimates this based on the assumption that all future discoverable credentials will have maximally-sized fields. Therefore, we do not need to worry about this but need to confirm this while implementing.

3. authenticatorClientPIN
  This authenticatorClientPIN command allows a platform to use a PIN/UV auth protocol to perform a number of actions: Setting a PIN, Changing a PIN, Obtaining the pinUvAuthToken. Platforms obtain a shared secret for each transaction. The authenticator does not have to keep a list of sharedSecrets for all active sessions. If there are subsequent authenticatorClientPIN transactions, a new sharedSecret is generated every time. 
   The authenticator interface needs to be updated:
   - regenerate(): Generates a fresh private public keys.
   - getPublicKey() → coseKey: Returns the authenticator’s public key as a COSE_Key structure.
   - decapsulate(peerCoseKey) → sharedSecret | error: Processes the output of encapsulate from the peer and produces a shared secret, known to both platform and authenticator.
   - decrypt(sharedSecret, ciphertext) → plaintext | error: Decrypts a ciphertext, using sharedSecret as a key, and returns the plaintext.
   - verify(key, message, signature) → success | error: Verifies that the signature is a valid MAC for the given message. If the key parameter value is the current pinUvAuthToken, it also checks whether the pinUvAuthToken is in use or not.
   
    The platform interfaces needs to be updated is:
   - encapsulate(peerCoseKey) → (coseKey, sharedSecret) | error : Generates an encapsulation for the authenticator’s public key and returns the message to transmit and the shared secret.
   - encrypt(key, demPlaintext) → ciphertext: Encrypts a plaintext to produce a ciphertext, which may be longer than the plaintext. The plaintext is restricted to being a multiple of the AES block size (16 bytes) in length.
   - decrypt(key, ciphertext) → plaintext | error: Decrypts a ciphertext and returns the plaintext.
   - authenticate(key, message) → signature: Computes a MAC of the given message.
