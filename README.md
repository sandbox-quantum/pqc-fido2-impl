# PQC FIDO2
In this project, we add support for post-quantum algorithms on FIDO2. We implemented Dilithium3 for Signing in WebAuthn and Kyber768 for KEM in CTAP2. The current implementation only supports discoverable (resident) credentials.

FIDO2 involves three main entities:

1. authenticator,
2. browser and
3. server.

We needed to change all these three entities. In our prototype, we use Nitrokey 3 firmware for `authenticator`, Firefox browser and `java-webauthn-server` from Yubico.

## Setup

Clone this repository and its submodules.

```
git clone git@github.com:sandbox-quantum/pqc-fido2-impl.git --recurse-submodules
```

## Web Authentication Server

### Build `liboqs-java`

*Note*: You have to build `liboqs` first.

Go to `liboqs-java/`, and follow the [Building the OQS dependency](https://github.com/sandbox-quantum/liboqs-java_fork#building-the-oqs-dependency) instructions.

Next, follow the [Building the Java OQS wrapper instructions](https://github.com/sandbox-quantum/liboqs-java_fork#building-the-java-oqs-wrapper):

```
mvn package -P macosx -Dliboqs.include.dir="/usr/local/include" -Dliboqs.lib.dir="/usr/local/lib"
```

### Start Java Web Auth Server

Go to `java-webauthn-server/`, run:

```
./gradlew run
```

## Firefox browser with PQC authenticator

Follow the instructions [here](docs/firefox.md) (make sure you have [`Mercurial`](https://pypi.org/project/mercurial/) installed first) to embed the modified PQC authenticator into Firefox.


## Hardware Authenticator

The [LPC55 Quickstart Guide](https://github.com/Nitrokey/nitrokey-3-firmware/blob/main/docs/lpc55-quickstart.md) explains how to compile and flash the firmware on a LPC55 Nitrokey 3 Hacker device or on a LPCXpresso55S69 development board.

### Nitrokey 3 Hacker

Once the device is reset and provisioned following the [LPC55 Quickstart Guide](https://github.com/Nitrokey/nitrokey-3-firmware/blob/main/docs/lpc55-quickstart.md), the firmware can be flashed using the following command line:

```console
make -C utils/lpc55-builder flash FEATURES=develop
```

### LPCXpresso55S69

Hardware setup:

- [LPCXpresso55S69](https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools-/lpcxpresso-boards/lpcxpresso55s69-development-board:LPC55S69-EVK) development board and 2 USB cables.

- Connect 2 USB cables to two ports: P9 High-Speed ("High Spd" label) and P6 Debug Probe ("Debug Link" label).

- Install the [SEGGER J-Link software bundle](https://www.segger.com/downloads/jlink/#J-LinkSoftwareAndDocumentationPack)

- Open the SEGGER J-Link Configuration program and make sure the J-Link protocol is being used
>  The board should appear in the list of devices connected via USB. We will be using the J-Link communication protocol. Therefore, in case the board appears as using the CMSIS-DAP protocol (the default in a new board), we will need to install the J-Link firmware by following [these steps](https://www.segger.com/products/debug-probes/j-link/models/other-j-links/lpc-link-2/). After restart, the board should show up in the configuration tool as a J-Link device. 

![SEGGER J-Link](images/jlink.png)

Go to the `nitrokey-3-firmware/` directory and from 2 terminals:

- Terminal 1: `make -C utils/lpc55-builder/ jlink`
- Terminal 2: `make -C utils/lpc55-builder/ run FEATURES=develop-no-press`

To verify that the board is working as a hardware authenticator, we use the `fido2-token` tool from `libfido2`:

```
❯ fido2-token -L
ioreg://4295801683: vendor=0x20a0, product=0x42b2 (Nitrokey Nitrokey 3)
```


## Test

1. Visit https://localhost:8443 in the Firefox browser and click on the "Create account with passkey" button. This will initiate the registration process using a resident key.

![Alt text](images/create_account.png)

Select "Proceed" and, if prompted for it, click the **USER** button on the board / tap the Nitrokey 3 Hacker board.

Output of the successful registration.

![Alt text](images/create_account_success_di3.png)

In our experiment, Dilithium3 ID is `-20`, you can see it in our [patched COSEY module](https://github.com/sandbox-quantum/cosey_fork/blob/pqc_kyber768_dilithium3/src/lib.rs#L76)

The string `DIL3`, `Dil` or `Dilithium` should appear in the server logs.


2. For authentication click "Authenticate with passkey" or "Authenticate with username" to authenticate with the corresponding resident key.

Select "Proceed" and click the **USER** button on the board  / tap the Nitrokey 3 Hacker board.

![Alt text](images/auth.png)

Output of the successful authentication. Again, the string `DIL3`, `Dil` or `Dilithium` should appear in the server logs.

![Alt text](images/auth_success.png)

### Troubleshooting

* MACOS:  In case of errors related to `liboqs` in the server, make sure `OpenJDK@17` is used (or any other version supporting `aarc64`) and add a symlink for `liboqs.5.dylib`:  

```console
ln -s /usr/local/lib/liboqs.5.dylib /opt/homebrew/Cellar/openjdk@17/17.0.9/libexec/openjdk.jdk/Contents/Home/lib/server/liboqs.5.dylib
```

### Application logs
You can enable RTT logs by running `make -C utils/lpc55-builder flash FEATURES=develop,log-rtt` (Nitrokey 3 Hacker device)/ `make -C utils/lpc55-builder/ run FEATURES=develop-no-press,log-rtt` (LPCXpresso55S69 development board). 

Logs will appear in port 19021 (you can run `nc -nv 127.0.0.1 19021`).


In order to additionally view application logs, you will need to add the `log-all` feature to the component for which you would like to view them. For example, for enabling application logs in the `fido-authenticator` component, we edit the `#apps` section of `nitrokey-3-firmware/components/apps/Cargo.toml` to add the `log-all` feature:
`fido-authenticator = { version = "0.1.1", features = ["dispatch","log-all"], optional = true }` .


> Troubleshooting: we had to remove another feature for this to work: in `nitrokey-3-firmware/components/apps/Cargo.toml`, `[features]` section, replace `-default = ["fido-authenticator", "ndef-app", "secrets-app", "opcard", "trussed/clients-4"]` with `default = ["fido-authenticator", "ndef-app", "secrets-app", "trussed/clients-4"]` .


## List of forked projects

| Project | Branch | Required by |
| ------- | ------ |---------|
| [java-webauthn-server](https://github.com/sandbox-quantum/java-webauthn-server_fork) | `add_Kyber768_Dilithium3_and_liboqs` | |
| [liboqs-java](https://github.com/sandbox-quantum/liboqs-java_fork) | `update_config_and_fix_error` | `java-webauthn-server` |
| [authenticator-rs](https://github.com/sandbox-quantum/authenticator-rs_fork) | `add_Kyber_and_Dilithium` | Firefox |
| [nitrokey-3-firmware](https://github.com/sandbox-quantum/nitrokey-3-firmware_fork) | `nitrokey-pqc-fido2` | Nitrokey 3 / LPCXpresso |
| [trussed](https://github.com/sandbox-quantum/trussed_fork) | `nitrokey-pqc-fido2` | `nitrokey-3-firmware` |
| [fido-authenticator](https://github.com/sandbox-quantum/fido-authenticator_fork) | `nitrokey-pqc-fido2` | `nitrokey-3-firmware` |
| [ctap-types](https://github.com/sandbox-quantum/ctap-types_fork) | `nitrokey-pqc-fido2` | `nitrokey-3-firmware`|
| [cosey](https://github.com/sandbox-quantum/cosey_fork) | `pqc_kyber768_dilithium3` | `fido-authenticator` |
| [usbd-ctaphid](https://github.com/sandbox-quantum/usbd-ctaphid_fork) | `nitrokey-pqc-fido2` | `nitrokey-3-firmware` |


## License

`PQC FIDO2` project is licensed under both the [Apache License, Version 2.0](LICENSE-APACHE) and [MIT License](LICENSE-MIT).
<br>
<sub>Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in the work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without any additional terms or conditions.</sub>


## Disclaimer
<sup>
The software and documentation are provided "as is" and SandboxAQ hereby disclaims all warranties, whether express, implied, statutory, or otherwise. SandboxAQ specifically disclaims, without limitation, all implied warranties of merchantability, fitness for a particular purpose, title, and non-infringement, and all warranties arising from course of dealing, usage, or trade practice. SandboxAQ makes no warranty of any kind that the software and documentation, or any products or results of the use thereof, will meet any person's requirements, operate without interruption, achieve any intended result, be compatible or work with any software, system or other services, or be secure, accurate, complete, free of harmful code, or error-free.
</sup>