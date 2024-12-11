# z/OS REXX Utilities

A collection of REXX scripts for z/OS system administration, cryptographic operations, and daily tasks.

## Table of Contents

- [Script Overview](#script-overview)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Scripts](#scripts)
  - [GENECC](#genecc)
  - [GENLI2](#genli2)
  - [JOBCARD](#jobcard)
  - [SMSVJSON](#smsvjson)
  - [Z](#z)

## Prerequisites

- z/OS operating system
- REXX support
- ICSF access (for cryptographic operations)
- SDSF access
- ISPF access

## Installation

1. Upload scripts to your z/OS system
2. Set appropriate file attributes:
   ```
   RECFM=FB
   LRECL=80
   DSORG=PS
   ```
3. Ensure execute permissions are set

## Scripts

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

### JOBCARD
ISPF edit macro for JCL job card generation.

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

### Z
Quick logoff utility.

Usage:
```
TSO %Z
```
