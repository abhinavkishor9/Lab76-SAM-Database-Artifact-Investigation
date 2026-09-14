# Troubleshooting Notes

## Issue 1 — Live SAM Could Not Be Hashed

### Command

```powershell
Get-FileHash $SAM -Algorithm SHA256
```

### Error

```text
The process cannot access the file
'C:\WINDOWS\System32\config\SAM'
because it is being used by another process.
```

### Cause

The live SAM hive is actively used by Windows.

It is not a normal unlocked file that can always be read directly while the operating system is running.

### Resolution

Use a forensic acquisition method such as Volume Shadow Copy and hash the acquired copy.

---

## Issue 2 — `vssadmin create shadow` Was Not Supported

### Command

```powershell
vssadmin create shadow /for=C:
```

### Result

```text
Error: Invalid command.
```

The installed `vssadmin` interface only exposed commands such as:

```text
Delete Shadows
List Providers
List Shadows
List ShadowStorage
List Volumes
List Writers
Resize ShadowStorage
```

### Resolution

The shadow copy was created through the Windows `Win32_ShadowCopy` interface:

```powershell
$Shadow = Invoke-CimMethod `
    -ClassName Win32_ShadowCopy `
    -MethodName Create `
    -Arguments @{
        Volume = "C:\"
        Context = "ClientAccessible"
    }
```

The resulting shadow copy was identified as:

```text
\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1
```

---

## Issue 3 — `Test-Path` Returned False

### Command

```powershell
$ShadowSAM = "\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SAM"

Test-Path $ShadowSAM
```

### Result

```text
False
```

The VSS object itself was confirmed through:

```powershell
Get-CimInstance Win32_ShadowCopy |
    Select-Object ID, DeviceObject, VolumeName, InstallDate
```

The shadow copy showed:

```text
DeviceObject:
\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1
```

Attempts to access the raw path through `cmd /c dir` also produced:

```text
The filename, directory name, or volume label syntax is incorrect.
```

### Lesson

The existence of a VSS snapshot does not automatically guarantee that a particular raw device path will work with every Windows command.

The shadow-copy path must be validated before attempting evidence acquisition.

---

## Issue 4 — Hashing the Wrong File

The original variable:

```powershell
$SAM
```

referred to:

```text
C:\Windows\System32\config\SAM
```

This is the live locked SAM.

The acquired evidence copy was instead referenced as:

```powershell
$AcquiredSAM = "$Lab\Evidence\SAM"
```

The correct hashing command was:

```powershell
Get-FileHash $AcquiredSAM -Algorithm SHA256
```

Successful SHA-256:

```text
62F2D387D3146578A00DDB08797D410666FF46C5FF9359312EC790930AC50B43
```

---

## Issue 5 — Hash Was Not Visible in PowerShell

### Command

```powershell
Get-FileHash $AcquiredSAM -Algorithm SHA256 |
    Out-File "$Lab\Evidence\sam-hash.txt"
```

### Observation

No hash appeared in the PowerShell console.

### Cause

`Out-File` redirects the output into a file.

### Resolution

Display the hash:

```powershell
Get-FileHash $AcquiredSAM -Algorithm SHA256
```

Then save it:

```powershell
Get-FileHash $AcquiredSAM -Algorithm SHA256 |
    Out-File "$Lab\Evidence\sam-hash.txt"
```

Verify:

```powershell
Get-Content "$Lab\Evidence\sam-hash.txt"
```

---

## Issue 6 — Event ID 4720 Was Not Found

### Command

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = "Security"
    Id = 4720
} -MaxEvents 50
```

### Result

```text
Get-WinEvent: No events were found that match the specified selection criteria.
```

A broader search for Event ID `4720` also returned no results.

### Interpretation

The controlled account was successfully created, but the expected account-creation event was not available in the Security log.

Possible reasons include:

- Account-management auditing was not enabled
- The event was not retained
- Security logging configuration differed from the expected configuration
- Relevant telemetry was unavailable

### Correct DFIR wording

Use:

```text
Event 4720: Not observed
```

Do not write:

```text
Event 4720 proves that the account was not created.
```

Missing telemetry is an evidence gap.

---

## Issue 7 — Wazuh Did Not Confirm Account Creation

Wazuh contained Windows telemetry from the workstation.

Observed telemetry included PowerShell activity:

```text
agent.id   : 001
agent.name : DESKTOP-9MMM37V
image      : C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
```

However, the available data did not independently confirm a `4720` event for `SAMTestUser`.

### Correct interpretation

```text
Windows telemetry available
        |
        v
PowerShell activity visible
        |
        v
Specific 4720 event not confirmed
```

Do not claim:

```text
Wazuh detected the SAM account creation.
```

Instead:

```text
Wazuh provided Windows telemetry that could be reviewed for account-related activity.
```

---

## Issue 8 — Existing Accounts Looked Suspicious

The baseline included accounts such as:

```text
CompromiseTest
DormantUser
TempSupport
```

These names may appear suspicious, but names alone are insufficient evidence of compromise.

A real investigation should correlate:

```text
Account
  +
SID / RID
  +
Enabled state
  +
Account creation evidence
  +
Group membership
  +
Last logon
  +
4624 / 4625
  +
Process activity
  +
Network activity
  +
Business justification
```

The lab therefore treated these as accounts requiring context rather than automatically malicious accounts.

---

## Issue 9 — Test Account Was Not an Administrator

The Administrators group contained:

```text
DESKTOP-9MMM37V\Administrator
DESKTOP-9MMM37V\Dell
DESKTOP-9MMM37V\TempSupport
```

`SAMTestUser` was not present.

This was intentional.

The lab focused on SAM and account investigation rather than privilege escalation.

---

## Issue 10 — Baseline Comparison

The baseline and final account lists were compared with:

```powershell
Compare-Object `
    (Import-Csv "$Lab\Evidence\local-account-baseline.csv").Name `
    (Import-Csv "$Lab\Evidence\local-accounts-final.csv").Name
```

Result:

```text
InputObject    SideIndicator
-----------    -------------
SAMTestUser    =>
```

This matched the controlled account creation.

---

## Issue 11 — Cleanup Verification

The controlled account was removed with:

```powershell
Remove-LocalUser -Name "SAMTestUser"
```

The account was then checked:

```powershell
Get-LocalUser "SAMTestUser" -ErrorAction SilentlyContinue
```

No result was returned.

A final inventory was exported to:

```text
local-accounts-after-cleanup.csv
```

