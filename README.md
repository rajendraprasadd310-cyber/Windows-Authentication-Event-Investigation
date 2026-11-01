# Windows-Authentication-Event-Investigation
Practical Windows Security Event Monitoring and Incident Investigation using Event Viewer, Windows 10 and Kali Linux.
 hands-on SOC analyst lab using **Windows 10 Event Viewer** and **Kali Linux** in Oracle VirtualBox. The project focuses on investigating six Windows Security event IDs:

- **4624** — Successful logon
- **4625** — Failed logon
- **4634** — Account logged off
- **4648** — Logon attempted using explicit credentials
- **4672** — Special privileges assigned to a new logon
- **4740** — User account locked out

> **Current project status:** The lab environment is configured and an SMB-based failed network logon has been generated and investigated as Event ID 4625. The other five event IDs are included in the project plan but still require their own practical evidence and investigation notes. Do not present them as completed until the evidence is added.

## Project objectives

1. Configure a small, isolated Windows/Kali lab.
2. Enable Windows logon failure auditing.
3. Generate controlled authentication activity.
4. Investigate event fields, timestamps, accounts, logon types, and failure codes.
5. Correlate the Kali test action with Windows Security logs.
6. Document findings in a repeatable SOC investigation format.

## Lab architecture

| Component | Role | Lab address / details |
|---|---|---|
| Windows 10 VM | Target and event-log source | `192.168.56.109` |
| Kali Linux VM | Test client | `192.168.56.106` |
| VirtualBox network | Host-only network | `192.168.56.0/24` (as observed in the lab) |
| Monitoring tool | Windows Event Viewer | Event Viewer → Windows Logs → Security |
| Test account | Standard local account | `LabUser` |
| Admin account | Windows administrator | `LabAdmin` |

The lab uses a Host-only network so the VMs can communicate without exposing SMB to the public internet. Kali was also given temporary NAT connectivity to install `smbclient`; the Host-only adapter remained in place for lab traffic.

## Repository layout

```text
windows-authentication-event-investigation/
├── README.md
├── LICENSE
├── .gitignore
├── docs/
│   ├── lab-setup.md
│   ├── event-id-matrix.md
│   └── incident-report-4625.md
└── evidence/
    ├── 01-lab-setup/
    ├── 02-event-4625-failed-logon/
    └── pending/
```

## Completed investigation: Event ID 4625

A controlled failed SMB authentication attempt was initiated from Kali against the Windows 10 VM:

```bash
smbclient //192.168.56.109/IPC$ -U LabUser
```

An intentionally incorrect password was entered for the test account. Kali returned:

```text
session setup failed: NT_STATUS_LOGON_FAILURE
```

Windows Event Viewer recorded Event ID **4625**. The selected event's visible fields included:

| Field | Observed value |
|---|---|
| Event ID | `4625` |
| TargetUserName | `LabUser` |
| TargetDomainName | `WORKGROUP` |
| LogonType | `3` |
| Status | `0xC000006D` |
| SubStatus | `0xC000006A` |
| FailureReason | `%%2313` |
| SubjectUserSid | `S-1-0-0` |
| SubjectUserName | `-` |
| SubjectLogonId | `0x0` |

Interpretation:

- Logon Type 3 means a network logon, consistent with SMB authentication.
- Status `0xC000006D` is a general logon failure status.
- SubStatus `0xC000006A` indicates an incorrect password.
- `%%2313` is the Windows message resource for an unknown user name or bad password. The substatus narrows this observed attempt to an incorrect password.
- `S-1-0-0` is the NULL SID. The Subject fields do not identify the attempted target account; the target account is shown separately as `LabUser`.

**Evidence limitation:** The supplied Event Viewer screenshot does not show the lower `IpAddress` and `WorkstationName` fields. The Kali address (`192.168.56.106`) is known from the lab network screenshot, but it should not be claimed as confirmed in this specific event until those fields are captured in Event Viewer.

See [the 4625 incident report](docs/incident-report-4625.md) and screenshots in `evidence/02-event-4625-failed-logon/`.

## Event IDs planned for investigation

| Event ID | Investigation | Status |
|---|---|---|
| 4624 | Successful logon; identify account, logon type, source and time | Previously observed during setup; add dedicated evidence/report |
| 4625 | Failed network logon; investigate account, failure codes and source | **Completed** |
| 4634 | Logoff; correlate Logon ID and session timeline | Pending |
| 4648 | Explicit credentials; inspect process and account context | Pending |
| 4672 | Special privileges; inspect account, privileges and Logon ID | Pending |
| 4740 | Account lockout; identify target account and caller computer | Pending |

The generation and investigation steps for each event are in [the event ID matrix](docs/event-id-matrix.md).

## How to reproduce the completed 4625 exercise

1. Use an isolated Host-only VirtualBox network.
2. Confirm the Windows VM and Kali VM can communicate.
3. Confirm Windows SMB server service is running and TCP 445 is reachable from Kali.
4. Create a standard local test account such as `LabUser`.
5. Enable failed logon auditing from an elevated Windows Command Prompt:

   ```cmd
   auditpol /set /subcategory:"Logon" /failure:enable
   ```

6. From Kali, initiate one controlled SMB authentication attempt:

   ```bash
   smbclient //192.168.56.109/IPC$ -U LabUser
   ```

7. Enter an intentionally incorrect password for the test account.
8. On Windows, open Event Viewer → Windows Logs → Security.
9. Filter for Event ID `4625`, open the matching timestamp, and inspect both **General** and **Details**.
10. Record the event fields and preserve screenshots.

## Safety and scope

- Perform these tests only on VMs and accounts you own or are explicitly authorized to test.
- Keep SMB restricted to the isolated lab network and the Kali VM address.
- Do not expose TCP 445 to a public or untrusted network.
- Use a disposable standard test account; do not use real credentials.
- Do not intentionally lock out an account outside the dedicated 4740 exercise.
- This project uses Event Viewer as the investigation interface; Kali is used only to generate the lab authentication attempt.

## Future improvements

- Add dedicated evidence and incident reports for all six event IDs.
- Capture `IpAddress`, `WorkstationName`, `Logon ID`, and timestamps wherever available.
- Add a consolidated incident timeline showing related 4624, 4625, and 4634 records.
- Add a final lessons-learned section after all scenarios are complete.
