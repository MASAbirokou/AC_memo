# そもそもレジストリとは？

- Windows全ての設定が格納がされているデータベース
- Key と Value から構成される（Key: フォルダ、Value: ファイル）
- レジストリには親となる「ルートキー」がある：
  - HKEY_CLASSES_ROOT (HKCR): ファイルの拡張子と関連付けについての情報
  - HKEY_CURRENT_USER (HKCU): ログイン中のユーザーの設定
  - HKEY_LOCAL_MACHINE (HKLM): システムについて全ての設定
  - HKEY_USERS (HKU): ユーザー別とデフォルトの設定を含む設定
  - HKEY_CURRENT_CONFIG: 使用中のプリンタとディスプレイの機器についての情報

c.f. https://soma-engineering.com/desktop/windows10/what-is-registry/2018/10/27/

![wi-regcommandfig11](https://user-images.githubusercontent.com/85237728/159463179-4d183811-d171-4353-bf37-4b0fc156ebb1.png)

c.f. https://atmarkit.itmedia.co.jp/ait/articles/0402/21/news005.html

# impacket-reg.py

-> Remote registry manipulation tool.

（nessusもリモートレジストリのenumをする）
  
```
usage: reg.py target query [-h] -keyName KEYNAME [-v VALUENAME] [-ve] [-s]

optional arguments:
  -h, --help        show this help message and exit
  -keyName KEYNAME  Specifies the full path of the subkey. The keyName must include a valid root key. Valid root keys for the local computer are: HKLM, HKU, HKCR.
  -v VALUENAME      Specifies the registry value name that is to be queried. If omitted, all value names for keyName are returned.
  -ve               Queries for the default value or empty value name
  -s                Specifies to query all subkeys and value names recursively.
```
<br>

- [about registry (HackTricks)](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries#registry)
- [ippsec (APT)](https://youtu.be/eRnqtXwCZVs?t=4051)

APTやったときに、ハッシュが分かってるのに、pass the hashもpass the ticketもgolden ticketもover pass the hashも出来なかったとき、このリモート
レジストリが希望となった。

まだログインセッションがないから、HKCUでなくHKUをenum（HKCUだと[-] Invalid root key HKCUとなる）:

![hkureg](https://user-images.githubusercontent.com/85237728/159431871-09c8ad6b-70c7-4c84-bd01-b2dacfa12a5b.png)

（＊）たとえ一回やってエラー出ても、続けてもう一回やればきちんと動く可能性あるから、一度のエラーで諦めるな
