# Lab Setup

## Environment

- Oracle VirtualBox
- Windows 10 VM (target)
- Kali Linux VM (test client)
- Windows Event Viewer (log investigation)
- Host-only network for VM-to-VM traffic

## Observed network configuration

| VM | IPv4 address |
|---|---|
| Kali Linux | `192.168.56.106` |
| Windows 10 | `192.168.56.109` |

Both addresses were observed with `ip addr` on Kali and `ipconfig` on Windows.

## Accounts and services

- `LabAdmin`: Windows administrator account used for system configuration.
- `LabUser`: standard local test account; membership was verified as `Users`.
- Windows `LanmanServer` service: confirmed Running.
- Windows SMB TCP port 445: confirmed reachable from Kali after firewall configuration.

## Audit policy

Failed logon auditing was enabled from an elevated Command Prompt:

```cmd
auditpol /set /subcategory:"Logon" /failure:enable
```

Windows returned: `The command was successfully executed.`

## Connectivity checks

From Kali:

```bash
ping -c 4 192.168.56.109
nc -nvz -w 3 192.168.56.109 445
```

The ping test received 4 replies with 0% packet loss. The Netcat test later reported port 445 as `open`.

## SMB firewall configuration

A Windows inbound TCP 445 allow rule was configured for the lab. For a safer setup, scope the rule's remote address to the Kali VM (`192.168.56.106`) and keep the VMs on the Host-only network.

Do not leave SMB exposed to untrusted networks. Remove the temporary lab rule when the lab is no longer needed.

## Troubleshooting notes

- The first ping attempts failed because Windows Firewall blocked ICMP echo requests. After enabling the relevant Echo Request rule, ping succeeded.
- TCP 445 initially timed out. The Windows SMB service was confirmed Running and a firewall rule was reviewed/created. The final Kali Netcat output showed TCP 445 open.
- Kali initially lacked `smbclient`; package installation succeeded after NAT internet access was enabled alongside the Host-only adapter.
- A reverse-DNS lookup warning from Netcat was not itself proof of a TCP failure. The meaningful final result was `445 (microsoft-ds) open`.
