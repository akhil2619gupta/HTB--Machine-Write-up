                                                      HTB TOUCH

                                             KIOSK-042 • Windows • Easy

                                  Machine notes, enumeration record, and solution summary
                  
      
        
<img width="1566" height="792"  style="padding-left: 35px;"  alt="image"  src="https://github.com/user-attachments/assets/a911e57f-21bd-4e47-928e-2f6e1966cebb"/>


The HTB Airways machine was a hands-on Windows penetration-testing exercise focused on application enumeration, MySQL investigation, credential discovery, filesystem permissions, and operating-system command execution through a MySQL User-Defined Function (UDF).

The assessment began with access to a Windows environment under the KIOSK-042\kioskuser account. Initial enumeration identified the HTB Airways application and its supporting files under C:\ProgramData\HTB Airways and C:\Program Files\HTB Airways\Kiosk.

During the investigation, a database maintenance batch file exposed MySQL credentials. This led to further examination of the locally installed MySQL 8.0.42 service, the htb_airways application database, and the server's configuration. The MySQL service was observed listening on 127.0.0.1:3306 and running as LocalSystem.

Further investigation identified the MySQL plugin directory as C:\MySQL\lib\plugin\ and revealed the availability of the sys_exec() and sys_eval() functions. These findings were significant because the functions supported operating-system command execution and output retrieval in the lab environment.

The investigation also involved troubleshooting file transfers between Kali Linux and Windows, distinguishing MySQL authentication errors from SQL syntax and database-selection errors, and converting hexadecimal output into readable text.

The final breakthrough came when the MySQL client command explicitly selected the mysql database and invoked sys_exec() to run a Windows command that redirected the contents of the Administrator's root flag file into the kiosk user's Downloads directory. The recovered hexadecimal representation was then converted into readable text.

Outcome: Both the user and root flags were recovered.

The primary security lesson is that a database service can become a route to operating-system privilege escalation when command-execution functionality is available and operates within a highly privileged service context. The combination of database access, native UDF functionality, service privileges, and filesystem permissions must therefore be assessed as a whole.

1.1 Key findings

The Windows session was associated with KIOSK-042\kioskuser.

The HTB Airways application used a local MySQL database.

A maintenance script exposed MySQL credentials.

The application database was named htb_airways.

The MySQL service listened on 127.0.0.1:3306.

The MySQL service was observed running as LocalSystem.

The configured MySQL plugin directory was C:\MySQL\lib\plugin\.

The sys_exec() and sys_eval() functions were available.

File-transfer troubleshooting exposed errors involving an incorrect IP address, an incorrect HTTP filename, and a reused filename.

The final root-flag retrieval involved sys_exec(), Windows output redirection, and hexadecimal-to-text conversion.

Both flags were recovered.
1.2  Lab Environment

Item	Recorded Value
Target	HTB Airways
Attacker OS	Kali Linux
Target OS	Windows
Target IP	10.129.137.223
Kali VPN Interface	tun0
Kali VPN IP	10.10.15.235
Windows Account	KIOSK-042\kioskuser
Database Version	MySQL 8.0.42, 64-bit
MySQL Service	MySQL80
MySQL Port	3306
Application Database	htb_airways
MySQL Installation	C:\MySQL
Plugin Directory	C:\MySQL\lib\plugin\
Service Identity	LocalSystem
Final Root-Output File	C:\Users\KioskUser\Downloads\flag.txt
Final Status	Both flags recovered

1.3 Scope and authorization

This write-up documents activities performed against the authorized HTB Airways lab. The findings should not be interpreted as authorization to test unrelated systems. Database functions capable of executing operating-system commands can have significant security implications and should be assessed only within an approved scope.
