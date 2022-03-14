# ReadGMSAPassword abuse
![ted_in_blood](https://user-images.githubusercontent.com/85237728/158108616-9fefaa43-d8f1-45dc-ab0e-81198a546b4b.png)

ReadGMSAPassword help:
> The intended use of a GMSA (Group Managed Service Accounts) is to allow certain computer accounts to retrieve the password for the GMSA, then run local services as the GMSA. 
> An attacker with control of an authorized principal may abuse that privilege to impersonate the GMSA.

上の画像では、SVC_INTがGroup Managed Service Accountである。そして、ITSUPPORTグループは、GMSAであるSVC_INTのパスワードをとれる！！

# gMSADump.py

https://github.com/micahvandeusen/gMSADumper

![gmsadump](https://user-images.githubusercontent.com/85237728/158111017-d5ad67c1-505b-4b50-b1b4-659e42402ef0.png)

**（注意）末尾の「$」マークもユーザ名に含まれてる**
