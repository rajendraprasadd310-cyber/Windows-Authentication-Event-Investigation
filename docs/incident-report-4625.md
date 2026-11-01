# Incident Investigation: Event ID 4625 — Failed SMB Logon

## 1. Summary

A controlled SMB authentication attempt was initiated from the Kali Linux VM to the Windows 10 VM using the standard test account `LabUser`. An intentionally incorrect password was supplied. Kali returned `NT_STATUS_LOGON_FAILURE`, and Windows Security recorded Event ID 4625.

**Assessment:** The observed event is consistent with the authorized lab test. It demonstrates a failed network logon, not evidence of a real-world compromise.

## 2. Lab context

| Item | Value |
|---|---|
| Source test VM | Kali Linux |
| Kali IPv4 address | `192.168.56.106` |
| Target VM | Windows 10 |
| Windows IPv4 address | `192.168.56.109` |
| Protocol/service | SMB over TCP 445 |
| Target account | `LabUser` |
| Target domain | `WORKGROUP` |
| Share used | `IPC$` |

## 3. Generation procedure

Kali command:

```bash
smbclient //192.168.56.109/IPC$ -U LabUser
```

The test operator entered an intentionally incorrect password.

Kali response:

```text
session setup failed: NT_STATUS_LOGON_FAILURE
```

Windows failed-logon auditing had been enabled using:

```cmd
auditpol /set /subcategory:"Logon" /failure:enable
```

## 4. Event evidence and observed fields

The matching Event Viewer record showed:

| Windows event field | Observed value | Interpretation |
|---|---|---|
| Event ID | `4625` | An account failed to log on |
| TargetUserName | `LabUser` | Account whose authentication failed |
| TargetDomainName | `WORKGROUP` | Local workgroup context |
| LogonType | `3` | Network logon |
| Status | `0xC000006D` | Generic logon failure |
| SubStatus | `0xC000006A` | Incorrect password |
| FailureReason | `%%2313` | Unknown user name or bad password message |
| SubjectUserSid | `S-1-0-0` | NULL SID |
| SubjectUserName | `-` | No named subject account in this record |
| SubjectLogonId | `0x0` | No subject logon session ID |

### Interpretation

- **Logon Type 3:** The authentication was performed over a network. This is consistent with SMB.
- **Status `0xC000006D`:** A general authentication failure.
- **SubStatus `0xC000006A`:** The supplied password was incorrect.
- **FailureReason `%%2313`:** Windows' rendered message indicates an unknown username or bad password. The SubStatus provides the more specific reason in this record.
- **Target account:** `LabUser` is the account being authenticated.
- **Subject identity:** The Subject fields identify the security context associated with the event; they are not the same as the target account.

### Source address limitation

The supplied Event Viewer Details screenshot is scrolled to the upper EventData fields and does not show `IpAddress` or `WorkstationName`. The lab's Kali IP is `192.168.56.106`, but the event's own source IP must be verified by scrolling farther down in the Details view or using the XML view. Do not claim the source IP is confirmed by this screenshot.

## 5. SOC investigation conclusion

**Finding:** One controlled failed network authentication for `LabUser` was observed. The event's Logon Type 3 is consistent with SMB authentication. The SubStatus indicates an incorrect password.

**Disposition:** Expected lab activity; no incident escalation is required for this isolated test.

**Recommended SOC follow-up in a real environment:**
1. Check whether other 4625 events target the same account.
2. Compare source IPs and workstation names across failures.
3. Check whether a 4624 success follows the failed attempts.
4. Correlate timestamps and Logon IDs where available.
5. Determine whether the account is expected to authenticate from that source.
6. Escalate if failures are frequent, distributed, target privileged accounts, or are followed by suspicious successful access.

## 6. Evidence

- `evidence/02-event-4625-failed-logon/01-kali-smb-authentication-failed.png`
- `evidence/02-event-4625-failed-logon/02-event-viewer-4625-details.png`
- `evidence/01-lab-setup/01-audit-logon-failure-enabled.png`

## 7. What remains to capture

- Event Viewer General tab for the same event
- Lower Details fields showing `IpAddress` and `WorkstationName`
- A screenshot showing the matching event selected by timestamp
