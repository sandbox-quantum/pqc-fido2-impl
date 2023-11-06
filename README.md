# PQC FIDO2
In this project, we add support for post-quantum algorithms on FIDO2. We implemented Dilithium3 for Signing in WebAuthn and Kyber768 for KEM in CTAP2. The current implementation only supports discoverable (resident) credentials.

FIDO2 involves three main entities, (1) authenticator, (2) browser and (3) server. We needed to change all these three entities. In our prototype, we use solo2 firmware for authenticator, firefox browser and java-webauthn-server from Yubico. 

## Setup
1. Clone repo 
```
git clone git@github.com:sandbox-quantum/pqc-fido2-impl.git --recurse-submodules 
``` 
2. Install liboqs (requires a Java Development Kit (JDK) such as OpenJDK >= 8 and Apache Maven and the Ninja build system)
Follow the instructions from https://github.com/open-quantum-safe/liboqs-java#building to:
* Build the main branch of liboqs
* Build the Java OQS wrapper
3. Server setup
 ```
cd java-webauthn-server && ./gradlew run
```
4. Firefox setup (requires Mercurial)
```
curl https://hg.mozilla.org/mozilla-central/raw-file/default/python/mozboot/bin/bootstrap.py -O
python3 bootstrap.py
cd mozilla-unified
```
In "Cargo.toml" file add `authenticator = { path = "../pqc-fido2-impl/authenticator-rs", version = "0.4.0-alpha.18", features = ["gecko"] }
` under `[patch.crates-io]`. 
Then, run `cargo update -p authenticator`
*(In case troubleshooting is needed, check instructions from https://firefox-source-docs.mozilla.org/toolkit/components/glean/dev/local_glean.html. Replace 'glean' with 'authenticator-rs'.)*
Then, run:
```
./mach build
./mach run
``` 
We tested with 118.0a1 Firefox nightly. The new versions of firefox should also work as long as modified `authenticator-rs` is used. 

5. Authenticator setup
Hardware required: LPCXpresso55S69 development board and 2 USB cables
Connect USBs to "Power-only" and "Debug" ports on the board.   
Make sure the board is identified as an authenticator. In Linux, you can use `fido2-token -L`
Terminal 1: `JLinkGDBServer -strict -device LPC55S69 -if SWD -vd`
Terminal 2: `cd solo2 && make run-dev`


## Test
1. Visit "https://localhost:8443" in the firefox browser and click on "Create account with passkey" button. This will initiate registration process using resident key. 
2. For authentication click "Authenticate with passkey" to authenticate with the resident key.
3. Use the *user button* in the board to authorize operations in the development board