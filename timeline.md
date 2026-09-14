# Timeline

## Lab 76: SAM Database Artifact Investigation

| Time | Activity | Evidence |
|---|---|---|
| 14-Sep-2026 07:11 | Lab environment initialized | `DESKTOP-9MMM37V` |
| 14-Sep-2026 07:11 | PowerShell version recorded | `7.6.6` |
| 14-Sep-2026 07:11 | Lab directory created | `C:\SAMDatabaseLab` |
| 14-Sep-2026 07:11 | Evidence directory created | `C:\SAMDatabaseLab\Evidence` |
| 14-Sep-2026 | SAM hive identified | `C:\Windows\System32\config\SAM` |
| 14-Sep-2026 | SAM metadata collected | 131072 bytes |
| 14-Sep-2026 | Direct SAM hashing attempted | File locked by Windows |
| 14-Sep-2026 07:15:54 | VSS snapshot created | `HarddiskVolumeShadowCopy1` |
| 14-Sep-2026 | SAM evidence copy acquired | `Evidence\SAM` |
| 14-Sep-2026 | SHA-256 recorded | `62F2D387...AC50B43` |
| 14-Sep-2026 | Local account baseline collected | CSV + `net user` output |
| 14-Sep-2026 | Existing accounts reviewed | Local account inventory |
| 14-Sep-2026 | SID/RID information collected | Account identifiers |
| 14-Sep-2026 | `SAMTestUser` created | Controlled test account |
| 14-Sep-2026 | Test account SID identified | RID `1015` |
| 14-Sep-2026 | Group membership investigated | Not an Administrator |
| 14-Sep-2026 | Event ID `4720` searched | No matching event |
| 14-Sep-2026 | Broader `4720` search performed | No matching event |
| 14-Sep-2026 | Wazuh telemetry reviewed | Windows telemetry available |
| 14-Sep-2026 | Wazuh `4720` confirmation assessed | Not observed |
| 14-Sep-2026 | Final account inventory exported | `local-accounts-final.csv` |
| 14-Sep-2026 | Baseline comparison performed | `SAMTestUser` identified |
| 14-Sep-2026 | Account evidence preserved | `sam-test-account.txt` |
| 14-Sep-2026 | `SAMTestUser` removed | Cleanup completed |
| 14-Sep-2026 | Final inventory exported | `local-accounts-after-cleanup.csv` |

## Investigation Sequence

```text
Environment Initialization
        |
        v
SAM Identification
        |
        v
SAM Metadata Collection
        |
        v
Live Hash Attempt
        |
        v
SAM File Lock Identified
        |
        v
VSS Acquisition
        |
        v
Acquired SAM Hash
        |
        v
Account Baseline
        |
        v
Controlled Account Creation
        |
        v
SID / RID Analysis
        |
        v
Group Membership Review
        |
        v
Security Event Investigation
        |
        v
Wazuh Investigation
        |
        v
Baseline Comparison
        |
        v
Evidence Preservation
        |
        v
Account Cleanup
```

## Key Evidence

### SAM

```text
C:\Windows\System32\config\SAM
```

### Acquired SAM

```text
C:\SAMDatabaseLab\Evidence\SAM
```

### SHA-256

```text
62F2D387D3146578A00DDB08797D410666FF46C5FF9359312EC790930AC50B43
```

### Test Account

```text
Name : SAMTestUser
SID  : S-1-5-21-51198790-337801975-3228388354-1015
RID  : 1015
```

### Account Difference

```text
SAMTestUser =>
```

### Security Event 4720

```text
Status: Not observed
```

### Wazuh

```text
Windows telemetry: Available
Specific 4720 confirmation: Not observed
```

### Cleanup

```text
SAMTestUser removed
```

## Evidence Status

```text
Confirmed
---------
SAM hive identified
SAM metadata collected
SAM acquired
SAM hash calculated
Baseline account inventory collected
SAMTestUser created
SAMTestUser SID/RID identified
Account inventory change observed
SAMTestUser removed

Not Observed
------------
Security Event 4720
Wazuh confirmation of 4720

Unknown
-------
Whether account-management auditing was configured
to generate the expected 4720 event
```

## Timeline Assessment

The investigation moved from artifact identification and acquisition to account enumeration, controlled account creation, telemetry investigation, correlation, and cleanup.

The absence of Event ID `4720` is preserved as a telemetry limitation rather than interpreted as evidence that account creation did not occur.

## DFIR Takeaway

```text
SAM Artifact
     +
Account Inventory
     +
SID / RID
     +
Security Events
     +
Group Membership
     +
Authentication Activity
     +
Wazuh
     |
     v
Evidence-Based Account Timeline
```

The investigation demonstrates that a SAM artifact is most useful when correlated with independent Windows and SIEM telemetry.
