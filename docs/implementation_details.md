# FIDO2 PQC

 **Goal**: Add PQC to FIDO2 protocol.

### Implementation Infrastructure
 - Authenticator
   - solo2 
        - fido-authenticator
        - trussed
        - cosey
        - ctap-types
        - pqclean
 - Server
   - java-webAuthn-server 
   - liboqs-java
 - Firefox browser
     - authenticator-rs: FIDO2 rust crate used in firefox. 

### Implementation changes
`fido-authenticator` library is used by solo2 and nitrokey3 to perform fido2 operation. It is built using on top of trussed app, which provides secure functionality for key generation, signing, storage etc. In our case, we needed to add PQC algorithms Dilithium3 and Kyber768 in FIDO2 library. The best place for this would be trussed app as it isolates all crypto operations. However, trussed app also makes a copy of inputs and outputs which increases the stack space usage. Dilithium3 requires a lot of stack size during signing. So, we implementation key generation and signing directly on `fido-authenticator` repo. However, we also needed to modify `trussed` app to support reading, writing, encrypting, decrypting large data as PQ based keys and signatures are larger in size. We modified `cosey` and `ctap-types` repo to support COSE encoding (serialization/deserialization) for Dilithium3 and Kyber768.
- `fido-authenticator`: worked on `pqc-develop`branch. Changes can be found at https://github.com/sandbox-quantum/fido-authenticator/compare/main...pqc_develop.
- `trussed`: worked on `pqc-develop2`branch. Changes can be found at https://github.com/sandbox-quantum/trussed/compare/main...pqc_develop2.
- cosey and ctap-types: https://github.com/sandbox-quantum/cosey/commits/main, https://github.com/sandbox-quantum/cosey/commits/main 

java-webAuthn-server is a server end library for FIDO2. We added support for additional signing algorithm Dilithium3.
- `java-webauthn-server`: worked on `main` branch. Most changes can be found in commit https://github.com/sandbox-quantum/java-webauthn-server/commit/970c97fb2f85134b6b7a0b6280813e99d8a950eb.

On firefox, we modify their `authenticator-rs` crate to support Dilithium3 and Kyber768. 
- `authenticator-rs`: worked on `ctap2-pqc-develop` branch. Changes can be found at https://github.com/sandbox-quantum/authenticator-rs/compare/ctap2-2021...ctap2-pqc-develop


### Other details 
1. Dilithium3 and Kyber768 have no defined COSE ids. So, in our implementation we use `-20` and `-24` for Dilithium3 and Kyber 678 resp. For COSE key type, we use `LWE = 5` and `PQCKEM = 6` for Dilithium3 and Kyber678 resp.
2. We use self-attestation for our PQ FIDO2 prototype. If a server would require DIRECT attestation, our implementation will break.
3. For CTAP we added KEM support for *authenticatorClientPIN* API. This command exists so that plaintext PINs are not sent to the authenticator. We added *pinUvAuthProtocol = 3* for Kyber768. The `fido-authenticator` is updated to support kyber and set *pinUvAuthProtocol=3*. We updated `authenticator-rs` to initiate KEM using kyber if it sees *pinUvAuthProtocol=3* in *authenticatorGetInfo*.
4. Dilithium3 takes a lot of stack size (~90-100 KB) during signing. We optimed the stack usage by scoping variables for as short as we can and increased the stack length in *runners/lpc55/build.rs* file of solo2.
5. Generate static library for dilithium3 and kyber768 from pqclean: Use MCUXpressoIDE to load all files for dilithium3 and kyber768 and compile them in a static library. As an example,
   1. In MCUXpresso, click "create a new C/C++ project" and select LPCXpresso55s69 board. Then select "C Static library" from Project .
   2. Load all files from https://github.com/PQClean/PQClean/tree/master/crypto_sign/dilithium3/clean to source folder.
   3. Create a common folder and load required files from https://github.com/PQClean/PQClean/tree/master/common. Add common folder in the path: project properties -> Settings -> Tool Setting -> Includes -> add location.
