# Constrained Delegation abuse

![delegation](https://user-images.githubusercontent.com/85237728/158114156-6e934692-fdf4-4e26-b0b8-93f9e4da7e51.png)

> The constrained delegation privilege allows a principal to authenticate as any user to specific services on the target computer.
> That is, a node with this privilege can impersonate any domain principal (including Domain Admins) to the specific service on the target host.

上の画像で、SVC_INTがドメインコントローラに対してconstrained delegation privilegeを持っている。で、SVC_INTはadminになりすます（impersonate）ことが出来る。

c.f.) https://wadcoms.github.io/wadcoms/Impacket-getST-Creds/

```
sudo ntpdate <target's IP>
```

## impacket-getST.py

[HTB-intelligence](https://youtu.be/Jg_BjkxdtsE?t=2673)

ハッシュとしてgMSADumper.pyで得られたlmハッシュを続けて「09e5c4522742c318011036d6f73a0b86:09e5c4522742c318011036d6f73a0b86」と指定する。

getST.pyに関しては何通りかtrial-and-errorしてみる（SPNのservice名、server名、ユーザ名末尾の"$"の有無）

![getst](https://user-images.githubusercontent.com/85237728/158138674-f8408d91-8149-4100-8cde-196bf0232a86.png)

<br>

そしたら、これを次のようにsecretsdump.pyに使う：
