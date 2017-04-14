---
layout: post
title: Metasploit
permalink: metasploit
---

进入命令行：

```bash
msfconsole
```


exploit 都存在 modules/exploits 目录里。

使用时，在命令行里：

```bash
msf > use  exploit/windows/smb/ms09_050_smb2_negotiate_func_index
msf exploit(ms09_050_smb2_negotiate_func_index) > help
...snip...
Exploit Commands
================

    Command       Description
    -------       -----------
    check         Check to see if a target is vulnerable
    exploit       Launch an exploit attempt
    pry           Open a Pry session on the current module
    rcheck        Reloads the module and checks if the target is vulnerable
    reload        Just reloads the module
    rerun         Alias for rexploit
    rexploit      Reloads the module and launches an exploit attempt
    run           Alias for exploit

msf exploit(ms09_050_smb2_negotiate_func_index) >
```



# 参考
- https://www.offensive-security.com/metasploit-unleashed/using-exploits/
