# z/OS REXX Utilities

A collection of REXX scripts for z/OS system administration, cryptographic operations, and daily tasks.

## Repository Layout

| Directory | Contents |
|-----------|----------|
| `SYSEXEC` | REXX execs — loaded via TSO SYSPROC/SYSEXEC concatenation |
| `CEXEC`   | Compiled REXX execs |

## Table of Contents

- [DCOLLECT](#dcollect) — DCOLLECT compression report
- [GENECC](#genecc) — PKCS #11 ECC key pair generator
- [GENLI2](#genli2) — Dilithium (post-quantum) key pair generator
- [HELLOW](#hellow) — Hello, world!
- [JOBCARD](#jobcard) — ISPF edit macro for JCL job card insertion
- [RESDDDEF](#resdddef) — SMP/E DDDEF volume/unit updater
- [RESRECAT](#resrecat) — Bulk volume recatalog utility
- [RSAGEN](#rsagen) — RSA key pair generator
- [SMSVJSON](#smsvjson) — SMS volume information extractor (JSON)
- [STDEDIT](#stdedit) — ISPF edit macro for standard edit settings
- [Z](#z) — Quick logoff utility

## Scripts

### DCOLLECT
Scans a DCOLLECT output dataset for `D` (data set) records and reports which data sets are compressed, along with their compression type.

Usage:
```
TSO %DCOLLECT your.dcollect.output.dataset
```

Compression types reported: `NOT COMPRESSED`, `GENERIC`, `TAILORED`, `ZEDC`.

### GENECC
PKCS #11 ECC key pair generator for z/OS.

```jcl
//GENECC   EXEC PGM=IKJEFT01
//SYSTSPRT DD SYSOUT=*
//SYSTSIN  DD *
  %GENECC
/*
```

Features:
- Generates secp521r1 curve key pairs
- Configures standard PKCS #11 attributes
- Compatible with EP11 for secure key operations
- Sets token persistence

### GENLI2
Dilithium (post-quantum) key pair generator.

```jcl
//GENLI2   EXEC PGM=IKJEFT01
//SYSTSPRT DD SYSOUT=*
//SYSTSIN  DD *
  %GENLI2
/*
```

Features:
- Generates quantum-resistant Dilithium keys
- Uses standard parameters (OID 1.3.6.1.4.1.2.267.1.6.5)
- EP11 compatibility for secure keys
- Sets appropriate signing/verification attributes

### HELLOW
Minimal "Hello, world!" exec — useful as a sanity check that REXX is working.

Source: `SYSEXEC/HELLOW` | Compiled: `CEXEC/HELLOW`

```
TSO %HELLOW
```

Output: `Hello, world!`

### JOBCARD
ISPF edit macro that inserts a standard JCL job card at the top of the current member.

Usage in ISPF:
```
Command ===> JOBCARD
```

Generated job card format:
```jcl
//IU***** JOB (FB3),'********',CLASS=A,MSGCLASS=H,
//       NOTIFY=&SYSUID,REGION=0M,TIME=1440
//*      TYPRUN=HOLD
```

### RESDDDEF
Updates SMP/E DDDEF entries in a zone to specify explicit `VOLUME` and `UNIT`, bypassing the catalog. Useful when datasets have been moved to a res-pack volume and catalog resolution is unreliable.

Usage:
```
TSO %RESDDDEF csi-dataset zone-name resvol1 [resvol2 ...]
```

Example:
```
TSO %RESDDDEF SYS1.SMPE.CSI TGZONE RESV01 RESV02
```

What it does:
1. Lists all DDDEFs in the target zone via SMP/E
2. Lists all datasets on the specified RESVOL volumes (IEHLIST)
3. For any DDDEF whose dataset lives on a RESVOL, issues `UCLIN REP DDDEF` to add `UNIT(3390) VOLUME(volser)`
4. Reports any RESVOL datasets that have no matching DDDEF in the zone

### RESRECAT
Recatalogs all non-VSAM datasets on a source volume to a new volume serial. Uses IDCAMS `DELETE NOSCRATCH` + `DEFINE NONVSAM` to update catalog entries without touching the data on disk.

Usage:
```
TSO %RESRECAT oldvol newvol
```

Example:
```
TSO %RESRECAT RESV01 RESV02
```

What it does:
1. Lists all datasets on `oldvol` (IEHLIST LISTVTOC)
2. For each dataset, verifies it is catalogued pointing to `oldvol`
3. Deletes and redefines the catalog entry pointing to `newvol`
4. Reports any datasets that could not be recataloged

### RSAGEN
Generates a 2048-bit RSA key pair using ICSF and stores the result in the PKDS (Public Key Data Set).

```jcl
//RSAGEN   EXEC PGM=IKJEFT01
//SYSTSPRT DD SYSOUT=*
//SYSTSIN  DD *
  %RSAGEN
/*
```

ICSF services used:
- `CSNDPKB` — builds a skeleton RSA token
- `CSNDPKG` — generates the key pair under the MASTER key
- `CSNDKRC` — stores the generated token in the PKDS

Default label: `BEN.RSA.KEY` (edit the source to change).

### SMSVJSON
SMS volume information extractor with JSON output.

Usage:
```
TSO %SMSVJSON > VOLUMES.JSON
```

Output format:
```json
{
  "sms_volumes": [
    {
      "volser": "VOL001",
      "status": "ENABLED",
      "total_mb": 3000,
      "used_mb": 2000,
      "free_mb": 1000,
      "usage_percent": 66,
      "storage_group": "PRIMARY",
      "device_status": "ONLINE"
    }
  ]
}
```

### STDEDIT
ISPF edit macro that applies a standard set of edit preferences. Run once at the start of an edit session or assign to a PF key.

Usage in ISPF:
```
Command ===> STDEDIT
```

Settings applied:
- `BOUNDS` — reset column boundaries to default
- `TABS OFF` — disable tab characters
- `NULLS ON` — trailing nulls on
- `HILITE AUTO` — automatic syntax highlighting
- `RECOVER ON` — enable edit recovery

### Z
Quick logoff utility — issues a TSO `LOGOFF`.

Usage:
```
Command ===> Z
```
