c.f. https://yojimbosecurity.ninja/golden-ticket-with-impacket/

# 必要なもの

- ターゲットのNTハッシュ
- Domain SID
- ドメイン名

（今回（HTB-APT）はターゲットのNTハッシュとドメイン名は分かっている前提）

## impacket-lookupsidでDomai SIDをゲット

![domainSID](https://user-images.githubusercontent.com/85237728/159251891-2699be92-0545-4b78-8698-25bec759666a.png)

## imapcket-ticketerでチケットを作る

![ticketer](https://user-images.githubusercontent.com/85237728/159253117-3282bf84-f817-4550-ab35-4da47cbc5b43.png)

## チケットを使ってimapcket-psexec

```
cp henry.vinson.ccache /tmp/krb5cc_0
export KRB5CCNAME=/tmp/krb5cc_0
```
