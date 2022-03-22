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

- /v <Valuename>: 照会するレジストリ値の名前を指定する（省略するとkeyname のすべての値の名前が返される）
- /ve: 値の名前が空のクエリを実行
- /s: すべてのサブキーと値の名前を再帰的にクエリ
  
-> imapcket-regでは、`-keyName <key> -v | -ve | -s`、つまり`/`じゃなくて`-`
  
# impacket-reg.py

-> Remote registry manipulation tool.

（nessusもリモートレジストリのenumをする）

<br>

APTやったときに、ハッシュが分かってるのに、pass the hashもpass the ticketもgolden ticketもover pass the hashも出来なかったとき、このリモート
レジストリが希望となった。

ippsec (APT): https://youtu.be/eRnqtXwCZVs?t=4051

まだログインセッションがないから、HKCUでなくHKUをenum（HKCUだと[-] Invalid root key HKCUとなる）:

![hkureg](https://user-images.githubusercontent.com/85237728/159431871-09c8ad6b-70c7-4c84-bd01-b2dacfa12a5b.png)

（＊）たとえ一回やってエラー出ても、続けてもう一回やればきちんと動く可能性あるから、一度のエラーで諦めるな
