# Lab 76: SAM Database Artifact Investigation

## Overview

This lab investigates the Windows Security Account Manager (SAM) database as a DFIR artifact for understanding local Windows accounts and account-related changes.

The investigation combines SAM metadata, forensic acquisition, local account enumeration, SID/RID analysis, baseline comparison, Windows Security telemetry, and Wazuh correlation.

A controlled `SAMTestUser` account was created to generate a known account-change scenario. The account was later removed after evidence collection.

The investigation follows an evidence-first approach and does not extract, crack, or modify password hashes.

## Objectives

- Understand the role of the Windows SAM database
- Locate and document the SAM hive
- Record SAM metadata and integrity information
- Acquire the SAM through Volume Shadow Copy
- Enumerate local Windows accounts
- Analyze account SIDs and RIDs
- Establish an account baseline
- Create a controlled test account
- Investigate account creation telemetry
- Review local group membership
- Investigate authentication activity
- Compare account inventories
- Review available Wazuh telemetry
- Preserve investigation evidence
- Build an account-focused timeline

## Scenario

A Windows workstation is being investigated after unexpected local-account activity is identified.

The analyst needs to determine which local accounts exist, understand their identifiers and current state, and establish whether supporting Windows Security and SIEM telemetry is available.

A controlled `SAMTestUser` account is created for the investigation. The account is then correlated with local account inventory, SID/RID information, group membership, Windows Security events, and Wazuh telemetry.

The investigation documents unavailable telemetry as an evidence gap rather than assuming that an event occurred.

## Investigation Flow

```text
SAM Hive
   |
   v
SAM Metadata
   |
   v
VSS Acquisition
   |
   v
Local Account Inventory
   |
   v
SID / RID Analysis
   |
   v
Account Baseline
   |
   v
Controlled Account Creation
   |
   v
Security Event Investigation
   |
   v
Group / Logon Investigation
   |
   v
Wazuh Correlation
   |
   v
Timeline
   |
   v
Cleanup
```

## Primary Artifact

```text
C:\Windows\System32\config\SAM
```

The SAM database is a protected Windows registry hive containing information associated with local Windows accounts.

The investigation focuses on account artifacts and metadata rather than password recovery.

## Evidence Collected

```text
Evidence/
├── SAM
├── sam-hash.txt
├── sam-metadata.txt
├── local-account-baseline.csv
├── net-user-baseline.txt
├── sam-test-account.txt
├── local-accounts-final.csv
└── local-accounts-after-cleanup.csv
```

## SAM Acquisition

The live SAM could not be hashed directly because Windows was using the file.

A Volume Shadow Copy was used to obtain an accessible copy for evidence handling.

The acquired SAM was successfully hashed using SHA-256.

```text
SHA256:
62F2D387D3146578A00DDB08797D410666FF46C5FF9359312EC790930AC50B43
```

The hash is used as an integrity reference for the acquired evidence copy.

## Baseline Accounts

The initial local account inventory included:

```text
Administrator
CompromiseTest
DefaultAccount
Dell
DormantUser
Guest
lab-reset-user
TempSupport
WDAGUtilityAccount
```

Account names, SIDs, enabled state, descriptions, last-logon information, and password requirements were collected where available.

## Controlled Test Account

The laboratory account was created as:

```text
Name        : SAMTestUser
SID         : S-1-5-21-51198790-337801975-3228388354-1015
RID         : 1015
Enabled     : True
Description : SAM database DFIR laboratory account
```

The account was not added to the local Administrators group.

## Security Telemetry

Security Event ID `4720` was investigated as the expected account-creation event.

No matching `4720` event was available in the Security log during the investigation.

This is documented as a telemetry gap rather than evidence that the account was not created.

Authentication events such as `4624` and `4625` were also considered for subsequent account activity.

## Wazuh

Wazuh telemetry from the Windows workstation was reviewed.

Windows activity was visible in Wazuh, including PowerShell process telemetry.

However, the available Wazuh data did not independently confirm a `4720` event for `SAMTestUser`.

The investigation therefore does not claim that Wazuh detected the account creation.

## Account Comparison

The baseline and final inventories were compared.

The controlled account appeared as the expected new account:

```text
SAMTestUser    =>
```

This demonstrated the account change introduced during the laboratory exercise.

## Cleanup

After evidence collection, the controlled account was removed.

```text
SAMTestUser
```

A final local-account inventory was then exported to document the post-cleanup state.

## Investigation Principles

> Follow the evidence, not the assumption.

> Account existence alone is not proof of compromise.

> Missing telemetry is an evidence gap, not evidence of absence.

> SAM evidence should be correlated with account creation, group membership, authentication, and SIEM telemetry.

## Key Takeaway

The SAM database provides useful local-account evidence, but it should not be investigated in isolation.

A stronger investigation combines SAM evidence with local account inventories, SID/RID analysis, Security events, group membership, authentication activity, and SIEM telemetry.
