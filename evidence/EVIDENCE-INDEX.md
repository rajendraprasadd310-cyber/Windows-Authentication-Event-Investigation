# Evidence Index

## Lab setup
- `01-lab-setup/01-audit-logon-failure-enabled.png` — `auditpol` confirms failed logon auditing was enabled.
- `01-lab-setup/02-kali-and-windows-vm-network-check.png` — Kali and Windows VM network troubleshooting view.
- `01-lab-setup/03-labadmin-account-verification.png` — Windows administrator account verification.
- `01-lab-setup/04-smb-server-service-running.png` — `LanmanServer` service shown as Running.
- `01-lab-setup/05-labuser-standard-account-created.png` — `LabUser` created and shown in the Users group.
- `01-lab-setup/06-windows-firewall-smb-in-rule.png` — Windows inbound SMB rule configuration.
- `01-lab-setup/07-host-only-network-ping-success.png` — Successful Kali-to-Windows ping.
- `01-lab-setup/08-smb-firewall-and-connectivity-troubleshooting.png` — Combined troubleshooting evidence.

## Event ID 4625
- `02-event-4625-failed-logon/01-kali-smb-authentication-failed.png` — Kali SMB authentication failure (`NT_STATUS_LOGON_FAILURE`).
- `02-event-4625-failed-logon/02-event-viewer-4625-details.png` — Windows Security Event ID 4625 Details.

## Pending evidence
Add evidence for Event IDs 4624, 4634, 4648, 4672 and 4740 as each exercise is completed. Add the missing 4625 General tab and lower Details fields when captured.
