# Timeline


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

