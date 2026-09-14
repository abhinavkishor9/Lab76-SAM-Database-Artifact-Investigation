# Lab76-SAM-Database-Artifact-Investigation
## Overview
The Security Account Manager (SAM) database is a Windows registry hive that stores information associated with local Windows user accounts.

The primary hive is:

C:\Windows\System32\config\SAM

It contains security-sensitive account information, so the investigation should focus on artifact analysis and account metadata, not extracting or attempting to crack password hashes.

For DFIR, the SAM artifact can help answer questions such as:

Which local accounts exist?
Which accounts are enabled or disabled?
What local account identifiers are present?
When was an account created or modified, where the artifact/parser supports that information?
Are unexpected local accounts present?
Does the account inventory correlate with Windows Security events?

The important point is that SAM should not be investigated in isolation.

A suspicious local account becomes much more meaningful when correlated with events such as:

4720 → account created
4732 → account added to local security-enabled group
4672 → special privileges assigned
4624 → successful logon
4625 → failed logon

The SAM hive provides local-account artifact evidence; Windows Security logs provide the activity context. Correlating both produces a stronger investigation than relying on either artifact alone.

This lab investigates the Windows Security Account Manager (SAM) database as a DFIR artifact for understanding local Windows accounts and account-related changes.

The investigation combines SAM metadata, forensic acquisition, local account enumeration, SID/RID analysis, baseline comparison, Windows Security telemetry, and Wazuh correlation.

A controlled `SAMTestUser` account was created to generate a known account-change scenario. The account was later removed after evidence collection.

The investigation follows an evidence-first approach and does not extract, crack, or modify password hashes.

## Objectives

The objectives of this lab are to:

- Identify the Windows SAM database and understand its relevance to local-account investigations.
- Examine SAM file metadata and document its location and state.
- Attempt protected SAM acquisition and understand why the live hive cannot be directly accessed.
- Use Volume Shadow Copy to investigate an alternative acquisition approach.
- Calculate and preserve a SHA-256 hash of the acquired SAM evidence.
- Establish a baseline of local Windows accounts before making changes.
- Create a controlled test account to generate known account-related evidence.
- Examine the account SID, RID, enabled state, and group membership.
- Search Windows Security logs for account-creation telemetry.
- Compare expected Security Event `4720` evidence with the telemetry actually available.
- Correlate the account investigation with available Wazuh process telemetry.
- Compare account inventories before and after the controlled change.
- Document telemetry limitations and distinguish an evidence gap from evidence of absence.
- Remove the test account and verify the final account state.
- 
## Scenario

A Windows endpoint is being examined for evidence related to local-account activity. The investigation focuses on the SAM database and supporting Windows telemetry to understand how a local account can be identified, documented, and correlated with other evidence. Since the live SAM hive is protected, the lab also demonstrates the challenges involved in acquiring and validating the artifact.

The investigation involves:

- Identifying the SAM database and recording its file metadata.
- Attempting direct access and documenting the protection encountered.
- Creating a Volume Shadow Copy as an alternative acquisition method.
- Preserving the acquired SAM and calculating its SHA-256 hash.
- Establishing a baseline of existing local accounts, including SIDs and RIDs.
- Creating a controlled `SAMTestUser` account to represent a known account change.
- Reviewing its account properties and local group membership.
- Searching for Windows Security Event ID `4720` related to account creation.
- Reviewing available Wazuh telemetry for supporting endpoint evidence.
- Comparing account inventories before and after the controlled change.
- Removing the test account and verifying the final state.

The investigation does not assume that the presence of a local account indicates compromise. Available artifacts and telemetry are evaluated separately, with missing Security event data documented as an evidence gap rather than treated as proof that the activity did not occur.


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

