# CVE-2020-1472 POC
Requires the latest impacket from [GitHub](https://github.com/SecureAuthCorp/impacket) with added netlogon structures. 

Do note that by default this changes the password of the domain controller account. Yes this allows you to DCSync, but it also breaks communication with other domain controllers, so be careful with this!

More info and original research [here](https://www.secura.com/blog/zero-logon)

## Installing
Only works on **Python 3.6 and newer**! If your OS comes installed with impacket (such as Kali), make sure to remove all existing installations first. If you still get import errors after this, run it from a *virtualenv* or *pipenv*. 

## Exploit steps
- Read the blog/whitepaper above so you know what you're doing
- Run `cve-2020-1472-exploit.py` with IP and netbios name of DC
- DCSync with secretsdump, using `-just-dc` and `-no-pass` or empty hashes and the `DCHOSTNAME$` account

## Restore steps
If you install a version of impacket from GitHub that was updated on or after September 15th 2020, secretsdump will automatically dump the plaintext machine password (hex encoded) when dumping the local registry secrets. Note that this is different from the DCSync, as DCSync uses the machine account, whereas dumping the registry requires you to execute secretsdump against the domain controller as a Domain Admin (such as the builtin Administrator account).

Alternatively on slightly older versions of secretsdump you can dump this same password by first extracting the registry hives and then running secretsdump offline (it will then always print the plaintext key because it can't calculate the Kerberos hashes).

With this plaintext password you can run `restorepassword.py` with the `-hexpass` parameter. This will first authenticate with the empty password to the same DC and then set the password back to the original one. Make sure you supply the netbios name and IP again as target, so for example:

```
python restorepassword.py testsegment/s2016dc@s2016dc -target-ip 192.168.222.113 -hexpass e6ad4c4f64e71cf8c8020aa44bbd70ee711b8dce2adecd7e0d7fd1d76d70a848c987450c5be97b230bd144f3c3...etc
```
