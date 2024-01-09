# Firefox with PQC authenticator

Note that these instructions are specific to MacOS.


## 1. Clone Firefox

```console
$ curl https://hg.mozilla.org/mozilla-central/raw-file/default/python/mozboot/bin/bootstrap.py -O
$ python3 bootstrap.py
```

Select `2. Firefox for Desktop` as the build option. (Don't select `1. Firefox for Desktop Artifact Mode [default]`, otherwise changes to Rust code will be ignored.)

Update to revision `752936` because breaking changes were introduced after that:

```console
$ cd mozilla-unified
$ hg update 752936 --clean
```


## 2. Apply patch

Use the appropriate path to the patch and apply it:

```console
$ patch -p1 < pqc-fido2-impl/docs/firefox.diff
```

Update `Cargo.lock` using our `authenticate-rs` version:

```console
$ cargo update -p authrs_bridge
```


## 3. Build Firefox

Prefer Firefox toolchain clang over the system clang:

```console
$ CC=$HOME/.mozbuild/clang/bin/clang ./mach configure
```

Build Firefox

```console
$ ./mach build
$ ./mach run
```


## Additional notes

### pqc_kyber and nss-gk-api versions


If the following errors are encountered during the build:

```
 0:38.46 error: no matching package named `pqc_kyber` found
 0:38.46 location searched: registry `crates-io`
 0:38.46 required by package `authenticator v0.4.0-alpha.18 (https://github.com/sandbox-quantum/authenticator-rs_fork.git?rev=70bf457918213d17afbbeb0c5570a495bf41b5f2#70bf4579)`
 0:38.47     ... which satisfies git dependency `authenticator` (locked to 0.4.0-alpha.18) of package `authrs_bridge v0.1.0 (/Users/user/Code/firefox/mozilla-unified/dom/webauthn/authrs_bridge)`
 0:38.47     ... which satisfies path dependency `authrs_bridge` (locked to 0.1.0) of package `gkrust-shared v0.1.0 (/Users/user/Code/firefox/mozilla-unified/toolkit/library/rust/shared)`
 0:38.47     ... which satisfies path dependency `gkrust-shared` (locked to 0.1.0) of package `gkrust-gtest v0.1.0 (/Users/user/Code/firefox/mozilla-unified/toolkit/library/gtest/rust)`
 ```

Add the following lines to the `[patch.crates-io]` section of `Cargo.toml`:

```
pqc_kyber = { git="https://github.com/Argyle-Software/kyber/", rev="476e22c1a1ed579f3030e1ae46077036dc384d7f" }
nss-gk-api = { git="https://github.com/mozilla/nss-gk-api", rev="bec1dd413b499a33ace5e61d3eaf701150b9d6ff" }
```

and update Cargo packages:

```console
$ cargo update -p authenticator
```

### libclang_rt.builtins-wasm32.a

`clang` misses `libclang_rt.builtins-wasm32.a` on MacOS. Download `https://github.com/jedisct1/libclang_rt.builtins-wasm32.a` and move it to `/opt/homebrew/Cellar/llvm/17.0.6/lib/clang/17/lib/wasi/`:

```
$ mkdir -p /opt/homebrew/Cellar/llvm/17.0.6/lib/clang/17/lib/wasi/
$ mv ~/Downloads/libclang_rt.builtins-wasm32.a /opt/homebrew/Cellar/llvm/17.0.6/lib/clang/17/lib/wasi/
```

### Linker errors

I'm not sure if XCode has to be installed. I encountered errors with the loader `--version` argument not being recognized and ended up patching a toolchain configuration file:

```diff
diff -r b2c0698cf97e build/moz.configure/toolchain.configure
--- a/build/moz.configure/toolchain.configure   Tue Jan 02 06:47:16 2024 +0000
+++ b/build/moz.configure/toolchain.configure   Wed Jan 03 16:51:25 2024 +0100
@@ -1752,7 +1752,7 @@
             die("Unsupported linker " + linker)

         # Check the kind of linker
-        version_check = ["-Wl,--version"]
+        version_check = ["-Wl,-v"]
         cmd_base = c_compiler.wrapper + [c_compiler.compiler] + c_compiler.flags

         def try_linker(linker):
@@ -1782,7 +1782,7 @@
             if retcode == 1 and "Logging ld64 options" in stderr:
                 kind = "ld64"

-            elif retcode != 0:
+            elif retcode != 0 and False:
                 return None

             elif "mold" in stdout:
```
