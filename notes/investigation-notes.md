# Investigation Notes

## Lab 76: SAM Database Artifact Investigation

## Environment

```text
Hostname   : DESKTOP-9MMM37V
User       : desktop-9mmm37v\dell
PowerShell : 7.6.6
Date       : 14 September 2026
Lab Path   : C:\SAMDatabaseLab
```

## SAM Artifact

The SAM hive was located at:

```text
C:\Windows\System32\config\SAM
```

Recorded metadata:

```text
Length        : 131072 bytes
CreationTime  : 01-04-2024 12:51:16
LastWriteTime : 09-09-2026 16:36:27
```

## SAM Acquisition

Direct hashing of the live SAM failed because the file was being used by Windows.

The investigation therefore used Volume Shadow Copy acquisition.

The created shadow copy was identified as:

```text
\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1
```

The acquired SAM was stored as:

```text
C:\SAMDatabaseLab\Evidence\SAM
```

SHA-256:

```text
62F2D387D3146578A00DDB08797D410666FF46C5FF9359312EC790930AC50B43
```

## Baseline Account Inventory

The initial account inventory included:

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

The baseline was exported to:

```text
local-account-baseline.csv
```

A `net user` baseline was also collected:

```text
net-user-baseline.txt
```

## Account State Review

The account inventory was reviewed for:

- Account name
- SID
- Enabled state
- Description
- Last logon
- Password requirements

Example account states included:

```text
Administrator      Disabled
CompromiseTest     Enabled
DefaultAccount     Disabled
Dell               Enabled
DormantUser        Enabled
Guest              Disabled
lab-reset-user     Enabled
TempSupport        Enabled
WDAGUtilityAccount Disabled
```

Existing account names were not treated as malicious by themselves.

Additional evidence would be required to determine whether an account is suspicious.

## SID and RID Analysis

Local account SIDs were collected using:

```powershell
Get-LocalUser |
    Select-Object Name, SID
```

The controlled test account had:

```text
SID:
S-1-5-21-51198790-337801975-3228388354-1015

RID:
1015
```

The RID was used as an account identifier for correlation.

A RID alone does not establish malicious activity.

## Controlled Account

A dedicated laboratory account was created:

```text
Name        : SAMTestUser
SID         : S-1-5-21-51198790-337801975-3228388354-1015
Enabled     : True
Description : SAM database DFIR laboratory account
```

The account was created specifically to provide a controlled account-change scenario.

## Group Membership

Local groups were searched for `SAMTestUser`.

The account was not present in the Administrators group.

The observed Administrators members were:

```text
DESKTOP-9MMM37V\Administrator
DESKTOP-9MMM37V\Dell
DESKTOP-9MMM37V\TempSupport
```

This provided useful privilege context without introducing an administrative test account.

## Security Event Investigation

Event ID `4720` was searched for the controlled account.

The search returned:

```text
Get-WinEvent: No events were found that match the specified selection criteria.
```

A broader search for Event ID `4720` also returned no results.

Therefore:

```text
4720 Status: Not observed
```

This does not prove that account creation did not occur.

It indicates that the expected account-creation telemetry was not available in the Security log during the investigation.

## Authentication Investigation

The investigation considered:

```text
4624 - Successful logon
4625 - Failed logon
```

These events provide authentication context.

They should be treated separately from the SAM artifact itself.

The presence of an account and the occurrence of an authentication event are different evidence points that should be correlated when available.

## Wazuh Investigation

Wazuh telemetry from the workstation was reviewed.

Available telemetry included Windows PowerShell process activity.

Example observed fields included:

```text
agent.id   : 001
agent.name : DESKTOP-9MMM37V
image      : C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

This confirmed Windows activity was reaching Wazuh.

However, the available data did not independently confirm Event ID `4720` for `SAMTestUser`.

Therefore:

```text
Windows telemetry visible      : Yes
Specific 4720 confirmation     : No
SAM artifact ingestion proven  : No
```

## Account Inventory Comparison

The baseline and final account inventories were compared using:

```powershell
Compare-Object `
    (Import-Csv "$Lab\Evidence\local-account-baseline.csv").Name `
    (Import-Csv "$Lab\Evidence\local-accounts-final.csv").Name
```

The result was:

```text
InputObject    SideIndicator
-----------    -------------
SAMTestUser    =>
```

This matched the controlled account creation performed during the lab.

## Evidence Interpretation

The investigation established:

```text
Confirmed:
- SAM hive identified
- SAM metadata recorded
- VSS acquisition completed
- Acquired SAM hashed
- Local account baseline established
- SAMTestUser created
- SAMTestUser SID/RID identified
- Account inventory change observed
- Test account removed

Not observed:
- Security Event 4720 for SAMTestUser
- Wazuh confirmation of Event 4720

Unknown:
- Whether account-management auditing was configured to generate the expected event
- Whether unavailable Security telemetry existed outside the retained log data
```

## Cleanup

The controlled account was removed after evidence collection:

```powershell
Remove-LocalUser -Name "SAMTestUser"
```

The account was then checked:

```powershell
Get-LocalUser "SAMTestUser" -ErrorAction SilentlyContinue
```

No result was returned.

The final account state was exported to:

```text
local-accounts-after-cleanup.csv
```

## Investigation Conclusion

The lab demonstrated how the Windows SAM can support local-account investigations when combined with other evidence.

The investigation did not treat the SAM artifact as proof of compromise.

The strongest correlation path is:

```text
SAM
  |
  +-- Account inventory
  |
  +-- SID / RID
  |
  +-- Account creation events
  |
  +-- Group membership
  |
  +-- Authentication events
  |
  +-- Wazuh telemetry
  |
  v
Account timeline
```
