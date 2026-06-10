# WSO2 MI SAP JCo Connector

A [WSO2 Micro Integrator](https://wso2.com/micro-integrator/) connector for SAP systems via the SAP Java Connector (JCo) library. Built from the [Ballerina SAP JCo module](https://github.com/ballerina-platform/module-ballerinax-sap.jco) and packaged for MI using the standard Ballerina-to-MI connector pipeline.

---

## Overview

This connector enables WSO2 MI integration flows to communicate with SAP systems through:

- **RFC** — Remote Function Call execution via `execute`
- **IDoc** — Intermediate Document dispatch via `sendIDoc`

The SAP JCo (`sapjco3.jar`) and SAP IDoc (`sapidoc3.jar`) libraries are shaded into the connector bundle with package relocation to prevent OSGi classpath conflicts in the MI runtime.

---

## Prerequisites

| Requirement | Version |
|---|---|
| Java | 21+ |
| WSO2 Micro Integrator | 4.6.0 |
| SAP JCo native library | 3.1.x |

### SAP Middleware Libraries

The SAP JCo and IDoc JARs **cannot be distributed publicly** (SAP license). You must obtain them from the [SAP Support Portal](https://support.sap.com):

| File | Maven Coordinate | Version |
|---|---|---|
| `sapjco3.jar` | `com.sap:com.sap.conn.jco` | `3.1.9` |
| `sapidoc3.jar` | `com.sap:com.sap.conn.idoc` | `3.1.3` |

Once downloaded, replace the jars in the project-local repo:

```
repo/
  com/sap/com.sap.conn.jco/3.1.9/com.sap.conn.jco-3.1.9.jar
  com/sap/com.sap.conn.idoc/3.1.3/com.sap.conn.idoc-3.1.3.jar
```

Regenerate the SHA1 checksums after replacing:

```bash
sha1sum repo/com/sap/com.sap.conn.jco/3.1.9/com.sap.conn.jco-3.1.9.jar \
  | awk '{print $1}' > repo/com/sap/com.sap.conn.jco/3.1.9/com.sap.conn.jco-3.1.9.jar.sha1

sha1sum repo/com/sap/com.sap.conn.idoc/3.1.3/com.sap.conn.idoc-3.1.3.jar \
  | awk '{print $1}' > repo/com/sap/com.sap.conn.idoc/3.1.3/com.sap.conn.idoc-3.1.3.jar.sha1
```

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
      ballerinax-sap.jco-1.0.1.jar     ← shaded fat-jar (SAP + Ballerina)
```

### What the build does

1. **Shade** — `maven-shade-plugin` bundles `ballerinax-sap.jco`, `com.sap.conn.jco`, and `com.sap.conn.idoc` into a single fat-jar. The `com.sap.*` packages are relocated to `org.wso2.integration.connector.sap.shaded.com.sap.*` to avoid classpath conflicts.
2. **Assemble** — `maven-assembly-plugin` packages the connector zip containing the shaded jar under `lib/` and the connector XML configuration at the archive root.

---

## Deployment

1. Place the SAP native library on the server's library path:

   | OS | File | Location |
   |---|---|---|
   | Linux | `libsapjco3.so` | `/usr/lib` or `LD_LIBRARY_PATH` |
   | Windows | `sapjco3.dll` | `%JAVA_HOME%\bin` or `PATH` |
   | macOS | `libsapjco3.dylib` | `/usr/local/lib` or `DYLD_LIBRARY_PATH` |

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
