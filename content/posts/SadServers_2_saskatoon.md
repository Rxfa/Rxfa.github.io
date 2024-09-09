+++ 
draft = false
date = 2024-09-06T04:08:00Z
title = "Saskatoon"
description = ""
slug = ""
authors = ["Rxfa"]
tags = ["Easy", "SadServers"]
categories = ["Writeups"]
externalLink = ""
series = ["SadServers"]
+++

# There's a web server access log file at `/home/admin/access.log`. The file consists of one line per HTTP request, with the requester's IP address at the beginning of each line.

## Find what's the IP address that has the most requests in this file (there's no tie; the IP is unique). Write the solution into a file `/home/admin/highestip.txt`. For example, if your solution is "1.2.3.4", you can do `echo "1.2.3.4" > /home/admin/highestip.txt`

For starters, we can read the log file to learn about the format.

```bash
admin@ip-172-31-27-155:/$ head -n 2 /home/admin/access.log
83.149.9.216 - - [17/May/2015:10:05:03 +0000] "GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1" 200 203023 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36"
83.149.9.216 - - [17/May/2015:10:05:43 +0000] "GET /presentations/logstash-monitorama-2013/images/kibana-dashboard3.png HTTP/1.1" 200 171717 "http://semicomplete.com/presentations/logstash-monitorama-2013/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36"
```

So we know that the IP address sits at the first column of the information we have of each log entry, and since we want to write the most common IP address to `/home/admin/highestip.txt`, this is what we come up with.

> cat /home/admin/access.log | awk '{print $1}' | sort | uniq -c | sort -n | tail -n 1 | awk '{print $2}'

With this command, we should open the log file, count the occurrences of each IP address, and then get the most common. Let's break it down.

- `cat /home/admin/access.log`: Reads the content of the file `/home/admin/access.log` and redirects it to the next command in the pipeline.

- `awk '{print $1}'`: Extracts the first field (IP address) of each HTTP request.

- `sort`: Sorts the IP addresses.

- `uniq -c`: Counts the occurrence of each IP address. returns something in the format `<count> <IP_address>`.

- `sort -n`: Sorts by count.

- `tail -n 1`: Returns the last line of the sorted list, which would be the most common IP address.

- `awk '{print $2}'`: Extracts the second field (IP address) of the output of `tail`.

We run the command, and write the output to `/home/admin/highestip.txt`.

```bash
admin@ip-172-31-27-155:/$ cat /home/admin/access.log | awk '{print $1}' | sort | uniq -c | sort -n | tail -n 1 | awk '{print $2}' > /home/admin/highestip.txt
admin@ip-172-31-27-155:/$ cat /home/admin/highestip.txt
66.249.73.135
```
