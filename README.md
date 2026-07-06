# WSO2 MI SAP JCo Connector

A [WSO2 Micro Integrator](https://wso2.com/micro-integrator/) connector for SAP systems via the SAP Java Connector (JCo) library. Built from the [Ballerina SAP JCo module](https://github.com/ballerina-platform/module-ballerinax-sap.jco) and packaged for MI using the standard Ballerina-to-MI connector pipeline.

---

## Overview

This connector enables WSO2 MI integration flows to communicate with SAP systems through:

- **RFC** — Remote Function Call execution via `execute`
- **IDoc** — Intermediate Document dispatch via `sendIDoc`

The SAP JCo (`sapjco3.jar`) and SAP IDoc (`sapidoc3.jar`) libraries are **not** bundled in the connector — they are proprietary and cannot be redistributed. You must obtain them from SAP and add them to the MI runtime yourself (see below).

---

## Prerequisites

| Requirement | Version |
|---|---|
| Java | 21+ |
| WSO2 Micro Integrator | 4.6.0 |
| SAP JCo native library | 3.1.x |

### SAP Middleware Libraries

The SAP JCo and IDoc JARs **cannot be distributed publicly** (SAP license), so they are **not** included in this connector or resolved at build time. You must obtain them from the [SAP Support Portal](https://support.sap.com):

| File | Version |
|---|---|
| `sapjco3.jar` | `3.1.9` |
| `sapidoc3.jar` | `3.1.3` |

These jars, together with the SAP native library, are added to the MI runtime at deployment time — see [Deployment](#deployment) below. They are not needed to build the connector.

---

## Build

```bash
mvn clean package
```

The build produces:

```
target/
  sap_jco-connector-1.0.1.zip          ← deployable connector archive
  connector/
    dependencies/
      ballerinax-sap.jco-2.0.0.jar     ← Ballerina connector jar (SAP classes not included)
```

---

## Deployment

### Installing the SAP native library

SAP JCo ships in **two parts** that are resolved by two different mechanisms:

- `sapjco3.jar` — Java classes, found via the **classpath**. Not bundled in the connector; place it in MI's `lib/` folder.
- `libsapjco3.dylib` / `.so` / `.dll` — the **native** library, found via the JVM's **`java.library.path`**, *not* the classpath.

The native library is not distributed with this connector — download it from the [SAP Support Portal](https://support.sap.com) as part of the SAP JCo package. Simply placing it on the classpath (e.g. in MI's `lib/` folder) is **not** enough — that folder is on the classpath but not on `java.library.path`. If the native library is missing from `java.library.path`, initialization fails with:

```
java.lang.UnsatisfiedLinkError: no sapjco3 in java.library.path: ...
```

In the instructions below, `<NATIVE_LIB_DIR>` is the directory where you extracted the downloaded native library. Pick **one** option per OS. The native library architecture (e.g. arm64 vs x86_64) must match the JVM's architecture. After installing, **fully restart** the MI server — native libraries load only once at JVM startup.

#### Linux — `libsapjco3.so`

| Option | How |
|---|---|
| Env var (no root) | `export LD_LIBRARY_PATH=<NATIVE_LIB_DIR>:$LD_LIBRARY_PATH` before starting MI |
| System dir (root) | `sudo cp <NATIVE_LIB_DIR>/libsapjco3.so /usr/lib/` then `sudo ldconfig` |

#### macOS — `libsapjco3.dylib`

| Option | How |
|---|---|
| User extensions dir (recommended) | `ln -sf <NATIVE_LIB_DIR>/libsapjco3.dylib ~/Library/Java/Extensions/libsapjco3.dylib` — this folder is on the default `java.library.path` and is user-writable (no root). |

> `DYLD_LIBRARY_PATH` is unreliable on macOS — it is stripped for many system-launched processes under System Integrity Protection (SIP). Prefer the symlink above.
>
> If the `.dylib` was downloaded, macOS Gatekeeper may quarantine it, producing a *different* error ("cannot be opened because the developer cannot be verified"). Clear it with:
> ```bash
> xattr -d com.apple.quarantine <NATIVE_LIB_DIR>/libsapjco3.dylib
> ```

#### Windows — `sapjco3.dll`

| Option | How |
|---|---|
| Folder on `PATH` | Copy `<NATIVE_LIB_DIR>\sapjco3.dll` into any directory listed in the `PATH` environment variable (e.g. `%JAVA_HOME%\bin`). |

#### Portable option (all OSes) — JVM flag

Setting `java.library.path` directly avoids the per-OS env-var differences. MI's launch script passes `$JAVA_OPTS` to the JVM, so you can set it without editing the script:

```bash
# macOS / Linux
export JAVA_OPTS="-Djava.library.path=<NATIVE_LIB_DIR>"
# Windows (cmd):  set JAVA_OPTS=-Djava.library.path=<NATIVE_LIB_DIR>
```

> `-Djava.library.path` **replaces** the default native path rather than appending to it, so include every folder you need. Use `:` as the separator on Unix and `;` on Windows.

---

## Connector Operations

### `init` — Initialize Connection

Establishes a connection to an SAP system. Called implicitly when the connector is used in a sequence.

**Parameters**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | Unique name for this connection |
| `connectionType` | string | Yes | Connection type identifier |
| `configurations_ashost` | string | Yes | SAP application server hostname |
| `configurations_sysnr` | string | Yes | SAP system number (e.g., `00`) |
| `configurations_jcoClient` | string | Yes | SAP client number (e.g., `100`) |
| `configurations_user` | string | Yes | SAP username |
| `configurations_passwd` | string | Yes | SAP password |
| `configurations_lang` | string | No | Logon language (e.g., `EN`) |
| `configurations_group` | string | No | Logon group for load balancing |
| `configurationsMap` | string | No | Advanced JCo properties as a map |

### `execute` — Execute an RFC Function Module

Executes a Remote Function Call (RFC) on the connected SAP system.

### `sendIDoc` — Send an IDoc

Sends an Intermediate Document (IDoc) to the SAP system in XML format.

---

## License

Licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).

**SAP JCo and SAP IDoc libraries** are proprietary software owned by SAP SE and distributed under SAP's own license terms. They are not included in this repository and must be obtained separately from the [SAP Support Portal](https://support.sap.com).
