HTB TOUCH

KIOSK-042 • Windows • Easy

Machine notes, enumeration record, and solution summary



Figure 1. HTB Touch machine page supplied with the request.


1. Machine Overview

Hack The Box identifies Touch as an Easy Windows machine. This document summarizes the recorded investigation of KIOSK-042: reconnaissance, Windows enumeration, MySQL and application-file investigation, and the database-to-operating-system command-execution finding used to retrieve the administrator/root flag.

Use these steps only in the authorized HTB lab or another system you own or are permitted to test.

2. Learning Objectives

Nmap host and service enumeration; saving scan output.

Web application inspection and screenshot evidence.

Windows command-line and PowerShell enumeration.

Users, groups, privileges, services, listeners, and file ACLs.

MySQL 8 client basics, authentication errors, configuration, and logs.

Understanding the risk of database functions that can execute OS commands.

Maintaining a reproducible write-up while redacting credentials and flags.

3. Lab Details

Item

Recorded value

Attacker OS

Kali Linux

VPN interface / address

tun0 / 10.10.15.235/23

Target hostname

KIOSK-042

Target IP

10.129.137.223

Windows user

kiosk-042\kioskuser

MySQL version

8.0.42 for Win64

MySQL listeners

127.0.0.1:3306; 0.0.0.0:33060

These addresses and names are from the recorded session and may change after resetting the HTB machine.

4. VPN and Nmap

The recorded Kali VPN interface was tun0. Verify the interface and route before starting enumeration:

ip addr show tun0
ip route get 10.129.137.223

Suggested scans for an authorized target (replace the IP if the machine instance changes):

nmap -Pn -sC -sV -oN nmap-initial.txt 10.129.137.223
nmap -Pn -p- --min-rate 2000 -oN nmap-all-ports.txt 10.129.137.223

The original Nmap output was not present in the material used to prepare this document. Add exact open ports, versions, commands, and screenshots from the saved terminal output; do not invent scan results.

5. Web Review

The supplied screenshot identifies the HTB machine as Touch, with Easy difficulty and Windows operating system. During target web enumeration, record the confirmed URL, HTTP/HTTPS, page title, redirects, login forms, errors, exposed paths, and technologies supported by scan results. Only document endpoints that were actually observed.

6. Windows Initial Session and Enumeration

Recorded Windows context:

whoami
hostname
whoami /priv
whoami /groups

Identity: kiosk-042\kioskuser

Hostname: KIOSK-042

Working directory: C:\Users\KioskUser\Downloads

SeChangeNotifyPrivilege was enabled in the captured output; SeUndockPrivilege, SeIncreaseWorkingSetPrivilege, and SeTimeZonePrivilege were disabled.

Useful commands for collecting additional evidence:

Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsBuildNumber
Get-Service | Where-Object {$_.Status -eq "Running"}
Get-NetTCPConnection -State Listen
whoami /all

7. MySQL and Application Investigation

Recorded listener information associated PID 3780 with 127.0.0.1:3306 and 0.0.0.0:33060. The MySQL80 service was configured for automatic startup, and the client was C:\MySQL\bin\mysql.exe, version 8.0.42 for Win64.

netstat -ano | findstr ":3306"
netstat -ano | findstr ":33060"
sc.exe qc MySQL80

Application directory:

C:\ProgramData\HTB Airways

File

Recorded detail

Access/result

db-config.ini

286 bytes

Access denied as kioskuser

db-sync-replica.ps1

10,724 bytes

Access denied as kioskuser

refresh-dates.bat

117 bytes

File present

refresh-dates.sql

2,282 bytes

File present

The actual batch filename observed was refresh-dates.bat; refresh.bat was not present. Access denied means the tested account could not read the file through that access path, not that the file was absent.

The MySQL configuration at C:\MySQL\my.ini included basedir=C:/MySQL, datadir=C:/MySQL/data, port=3306, bind-address=127.0.0.1, max_allowed_packet=256M, and innodb_buffer_pool_size=512M.

The error log was C:\MySQL\Data\mysql-error.log. Recorded excerpts mentioned an unknown --install option, a warning involving kiosk_app@localhost, and repeated forced thread closures for kiosk_app. A root login attempt failed with MySQL error 1045 (access denied). These were clues to correlate, not proof that no other account could be used.

8. Privilege Escalation Finding

The recorded solution used MySQL's sys_exec function to execute a Windows command that copied the administrator/root flag into the current user's Downloads directory. This demonstrates how database command-execution functionality can cross the boundary from database access to host-level execution.

SELECT sys_exec('cmd.exe /c type C:\\Users\\Administrator\\Desktop\\root.txt > C:\\Users\\KioskUser\\Downloads\\flag.txt');

The command above is included as a historical lab note. Do not include real passwords in a shared write-up; replace any password with <REDACTED> and keep credentials in private notes.

Security takeaway

Restrict database privileges, disable unnecessary command-execution functions, protect application configuration and scripts, review service-account permissions, and run database services with least privilege.

9. Flags and Evidence

User flag captured: [ ]

Administrator/root flag captured: [ ]

Screenshot evidence saved: [ ]

Sensitive flag values redacted before sharing: [ ]

The exact user-flag acquisition command was not included in the material available for this document. Add it from your original terminal history rather than guessing.

10. Screenshot Checklist

01-machine-page.png — HTB machine page and difficulty

02-vpn-and-route.png — tun0 address and route

03-nmap-initial.png — initial Nmap results

04-nmap-full-ports.png — full-port scan

05-web-enumeration.png — browser findings

06-user-session.png — Windows user and hostname

07-windows-enumeration.png — privileges, services, listeners

08-mysql-enumeration.png — MySQL listener/service

09-application-files.png — file names and permissions

10-mysql-error-log.png — sanitized log clues

11-user-flag-proof.png — user flag proof, value redacted

12-root-flag-proof.png — root flag proof, value redacted

The supplied machine-page image is included in the companion folder as touch-machine-page.png.

11. Lessons Learned

Verify VPN routing before troubleshooting target connectivity.

Save Nmap output and document only confirmed findings.

Use file permissions and access-denied results as enumeration evidence.

Check local-only listeners after gaining a Windows session.

Correlate service configuration, application scripts, accounts, and error logs.

An unsuccessful MySQL root login does not end database enumeration.

Database OS-command execution is a high-impact security boundary.

Redact credentials, flag values, and private lab configuration before publishing.

12. Final Checklist

Add exact Nmap commands and outputs.

Document confirmed web ports and browser findings.

Add the exact user-flag acquisition steps.

Insert sanitized screenshots from each stage.

Redact passwords and flag values.

Verify paths and commands against terminal history.

Keep README.md and the screenshot asset together.

Personal lab documentation prepared from the supplied HTB Touch machine-page screenshot and recorded KIOSK-042 investigation notes.