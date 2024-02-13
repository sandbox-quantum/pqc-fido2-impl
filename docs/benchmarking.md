## Benchmarking

|      |                      | FIDO2              | FIDO2-Dilithium3    | FIDO2-Dilithium5    |
| ---- | -------------------- | ------------------ | ------------------- | ------------------- |
| 1    | **Network data**     |                    |                     |                     |
| 2    | **Memory**           |                    |                     |                     |
|      | Authenticator-code   |              503kB |               543kB |                     |
|      | Authenticator-keys   |                    |                     |                     |
|      | Server-code          |                    |                     |                     |
|      | Server-keys          |                    |                     |                     |
|      | # supported accounts |                    |                     |                     |
| 3    | **Time**             |                    |                     |                     |
|      | Authenticator-registration |        1.29s |               1.54s |                     |
|      | Authenticator-authentication |      0.14s |               1.07s |                     |
|      | Server-registration  |                    |                     |                     |
|      | Server-authentication |                   |                     |                     |
|      | Overall-registration |                    |                     |                     |
|      | Overall-authentication |                  |                     |                     |
|      | Overall-authenticatorClientPIN |          |                     |                     |


### Notes

The authenticator code size is `runner-lpc55-nk3xn.bin` size, built with `make build FEATURES=develop-no-press`.

The authenticator times result from registration and authentication times through USB (average, 10 runs). Results vary and this is a rough estimate.
