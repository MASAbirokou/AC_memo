c.f.)
- [HackTheBox - Fuse](https://youtu.be/VxbC03xmS60?t=1087)
- [Infrastructure PenTest Series](https://bitvijays.github.io/LFF-IPS-P3-Exploitation.html)

### `rpcclient`

`rpcclient` wiht no password:

```
┌──(shoebill㉿shoebill)-[~]
└─$ rpcclient -U "" -N <target IP>
rpcclient $> setuserinfo2 <username> 23 <new password>
```
"NT_STATUS_NO_USER_SESSION_KEY"のエラーはログインしてない場合はできないということ。

### `smbpasswd`

```
┌──(shoebill㉿shoebill)-[~]
└─$ smbpasswd -U <username> -r <target IP>
Old SMB password:
New SMB password:
Retype new SMB password:
Password changed for user tlavel
```
