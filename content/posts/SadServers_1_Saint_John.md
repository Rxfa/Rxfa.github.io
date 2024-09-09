+++ 
draft = false
date = 2024-09-05T04:20:20Z
title = "Saint John"
description = ""
slug = ""
authors = ["Rxfa"]
tags = ["Easy", "SadServers"]
categories = ["Writeups"]
externalLink = ""
series = ["SadServers"]
+++

# A developer created a testing program that is continuously writing to a log file `/var/log/bad.log` and filling up disk. You can check for example with tail -f /var/log/bad.log. This program is no longer needed. Find it and terminate it.

So, since we know that this program is continuously writing to a log file we can use the `lsof` command.
This command stands for “list open files” and what it does is that it lists open files opened by processes. so if you type `lsof` in your terminal and press enter, you should have information about every file opened in your system as well as information about the process that opened them. In this case, we only want information about one file and the process that opened it, so the command we should use is `lsof <path_to_the_file>`.

```txt
admin@i-08efd29e8dd873afc:~$ lsof /var/log/bad.log
COMMAND     PID     USER    FD  TYPE    DEVICE  SIZE/OFF    NODE    NAME 
badlog.py   587     admin   3W  REG     259,1   11074       265802  /var/log/bad.log
```

And there it is! `badlog.py` is the only program that has opened our log file, it has to be the process we have to terminate. So we kill it (since we also know its ID) and run `lsof` again to make sure no process is writing to our log file.

```txt
admin@i-08efd29e8dd873afc:~$ kill -9 587
admin@i-08efd29e8dd873afc:~$ lsof /var/log/bad.log

```

As expected, no process has opened our log file. We can now click on "check my solution" to confirm that this is over.
