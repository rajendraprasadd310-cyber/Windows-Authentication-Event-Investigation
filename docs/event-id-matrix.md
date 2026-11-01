# Event ID Investigation Matrix

This project uses Windows Event Viewer only for log collection and investigation. Kali is used as a test client to generate selected authentication activity.

## 4624 — Successful logon

**Goal:** Identify a successful sign-in and determine how it occurred.

Investigate:
- Target account and domain
- Logon Type
- Source network address and workstation (when populated)
- Authentication package
- Timestamp
- Logon ID for correlation

Practical activity:
- Sign in interactively to Windows using the lab account.
- Find Event ID 4624 in Security.
- For the local console sign-in, expect an interactive logon (typically Type 2).
- For SMB network authentication, expect Type 3.

Evidence required:
- Event Viewer General tab
- Details showing account, Logon Type, Logon ID and source fields where present

## 4625 — Failed logon

**Status: Completed for one SMB network authentication attempt.**

Practical activity:
- From Kali, connect to the Windows IPC$ share using `LabUser`.
- Enter one intentionally incorrect password.
- Inspect the matching Event ID 4625.

Investigate:
- Target account and domain
- Logon Type
- Status and SubStatus
- Failure Reason
- Source IP and workstation
- Timestamp

Observed in the supplied evidence:
- TargetUserName: `LabUser`
- TargetDomainName: `WORKGROUP`
- LogonType: `3`
- Status: `0xC000006D`
- SubStatus: `0xC000006A`
- FailureReason: `%%2313`

## 4634 — Account logged off

**Goal:** Establish when a logon session ended.

Practical activity:
- Create a session using a lab account, then sign out or close the relevant session.
- Filter Security for Event ID 4634.

Investigate:
- Target account
- Logon ID
- Logon Type (if available)
- Timestamp

Correlation:
- Match the 4634 Logon ID with the corresponding 4624 Logon ID. A 4634 is a session logoff record; it does not necessarily mean the user shut down Windows.

Evidence required:
- 4634 General and Details tabs
- Related 4624 event with matching Logon ID

## 4648 — Logon attempted using explicit credentials

**Goal:** Identify a process that supplied explicit credentials.

Practical activity:
- On Windows, use the built-in `runas` command with the test account, for example:

  ```cmd
  runas /user:.\LabUser cmd
  ```

- Enter the test account password and inspect Event ID 4648.

Investigate:
- Subject account
- Account whose credentials were used
- Target server
- Process name and process ID (where present)
- Timestamp

Evidence required:
- 4648 General and Details tabs
- Note the process and both account identities; do not assume the subject and credential account are the same.

## 4672 — Special privileges assigned to a new logon

**Goal:** Review privileged logon activity.

Practical activity:
- Sign in with the Windows administrator account.
- Filter Security for Event ID 4672.

Investigate:
- Subject account
- Logon ID
- Privilege list
- Timestamp

Correlation:
- Match the 4672 Logon ID to the corresponding 4624 event. This event is not automatically malicious; determine whether the account and privileges are expected.

Evidence required:
- 4672 Details showing account, Logon ID and privileges
- Matching 4624 event

## 4740 — User account locked out

**Goal:** Investigate an account lockout and its likely cause.

Practical activity:
- Use only the disposable `LabUser` account.
- Configure a low lockout threshold in the local account lockout policy for the lab.
- Make a small, controlled number of incorrect authentication attempts until the test account locks.
- Inspect Event ID 4740.

Investigate:
- Target account
- Caller Computer Name
- Timestamp
- Nearby 4625 events and their source details

Important:
- Local account lockout behavior and event placement can differ from domain account lockout behavior.
- Domain account lockouts are typically investigated on the domain controller.
- Reset the test account and restore the original lockout policy after the exercise.

Evidence required:
- 4740 General and Details tabs
- Related 4625 records and a short timeline

## Standard investigation template

For every scenario, record:

1. Event ID and event description
2. Date/time and time zone
3. Target account and domain
4. Subject account, if present
5. Logon Type, if present
6. Source IP and workstation, if present
7. Status, SubStatus and failure reason, if relevant
8. Logon ID and correlated events
9. Expected or suspicious activity assessment
10. Recommended next steps
11. Screenshot filenames
