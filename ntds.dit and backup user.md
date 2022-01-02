## ntds.ditとは？
- **ドメインコントローラーの中でも最も重要なデータベースファイル**
- All data in Active Directory is stored in the file **ntds.dit** (by default located in `C:\Windows\NTDS\`) on every domain controller. (c.f.[Ntds.dit Password Extraction(stealthbits)](https://attack.stealthbits.com/ntds-dit-security-active-directory))
- Admin権限 or **backupユーザー** であれば見れるかも
- **SeBackupPrivilege**が読むことを、**SeRestorePrivilege**が書き込むこと、この二つの権限があれば良い。
- ntds.ditの存在場所：**C:\Windows\NTDS\ntds.dit**
- NTDSファイルをdecryptするためにはkeyが必要で、そのkeyは**system registry hive**なるレジストリに含まれている。-> `reg save hklm\system c:\temp\system`


c.f.)
- 複数のやり方が書いてある -> [Tha Hacker Recipies](https://www.thehacker.recipes/ad/movement/credentials/dumping/ntds)
- 実際にうまく動いた -> [Windows PrivEsc with SeBackupPrivilege](https://medium.com/r3d-buck3t/windows-privesc-with-sebackupprivilege-65d2cd1eb960)

**SeBackupPrivilege**がある：

```
*Evil-WinRM* PS C:\Users\svc_backup> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
*Evil-WinRM* PS C:\Users\svc_backup> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                 Type             SID          Attributes
========================================== ================ ============ ==================================================
Everyone                                   Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Backup Operators                   Alias            S-1-5-32-551 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users            Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                       Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level
```
