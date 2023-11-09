---
title:       "ログコマンド"
subtitle:    "Linux調査用コマンド"
description: "Linuxログコマンド"
date:        2023-10-23
author:      "sora"
image:       "img/home-bg-net.png"
tags:        ["tech", "memo"]
categories:  ["Tech"]
---

# 各種ログ調査コマンド

## ログサンプル

```bash
# アクセスログ
110.15.201.15 - - [07/Jun/2021:15:16:25 +0900] "GET /bulletins HTTP/1.1" 504 9345 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"
110.15.201.15 - - [07/Jun/2021:15:17:17 +0900] "GET /bulletins HTTP/1.1" 504 9345 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"
127.0.0.1 - - [07/Jun/2021:15:17:35 +0900] "GET / HTTP/1.0" 200 11150 "-" "check_http/v2.3.3"
127.0.0.1 - - [07/Jun/2021:15:19:35 +0900] "GET / HTTP/1.0" 200 11150 "-" "check_http/v2.3.3"

# スロークエリログ
/usr/sbin/mysqld, Version: 8.0.23 (MySQL Community Server - GPL). started with:
Tcp port: 3306  Unix socket: /var/lib/mysql/mysql.sock
Time                 Id Command    Argument
# Time: 2021-05-24T08:12:56.241844Z
# User@Host: wp-user[wp-user] @  [192.168.0.229]  Id:   289
# Query_time: 10.003509  Lock_time: 0.000085 Rows_sent: 0  Rows_examined: 1
use trng_webdb01;
SET timestamp=1621843966;
UPDATE `wp_options` SET `option_value` = '1621843966.2317020893096923828125' WHERE `option_name` = '_transient_doing_cron';
/usr/sbin/mysqld, Version: 8.0.23 (MySQL Community Server - GPL). started with:
Tcp port: 3306  Unix socket: /var/lib/mysql/mysql.sock
Time                 Id Command    Argument
/usr/sbin/mysqld, Version: 8.0.23 (MySQL Community Server - GPL). started with:
Tcp port: 3306  Unix socket: /var/lib/mysql/mysql.sock
Time                 Id Command    Argument
```

## アクセスログ集計

- `httpd -S`でApacheのルートを見つける
- `grep CustomLog  /etc/httpd/conf*/*`でCustomLogの記述を見つける

### 1分間隔で集計

- `ionice -c2 -n7 nice -n19 cat /var/log/httpd/access_log |awk -F\[ '{print $2}' | cut -c2-17 | sort | uniq -c`

```bash
3 7/Jun/2021:10:01
1 7/Jun/2021:10:03
1 7/Jun/2021:10:05
```

- `ionice -c2 -n7 nice -n19 awk -v start="07/Jun/2021:10:25:35" -v end="07/Jun/2021:11:25:35" '{ gsub(/\[/, "", $4); } $4 > start && $4 < end' /var/log/httpd/access_log |awk -F\[ '{print $2}' | cut -c2-17 | sort | uniq -c`

### １０分間隔で集計

- `ionice -c2 -n7 nice -n19 cat /var/log/httpd/access_log | awk -F\[ '{print $2}' | cut -c2-16 | sort | uniq -c`

```bash
7 7/Jun/2021:10:0
5 7/Jun/2021:10:1
5 7/Jun/2021:10:2
```

### アクセス元IPを集計

- `ionice -c2 -n7 nice -n19 cat access_log.txt | awk '{print $1}'|sort | uniq -c`

```bash
  3 110.15.201.15
242 127.0.0.1
```

- 指定時間から最新まで
  - `ionice -c2 -n7 nice -n19 awk -v date="[07/Jun/2021:17:49:25" '$4 > date' access_log.txt | awk '{print $1}' | sort | uniq -c`
- 指定範囲内
  - `sudo ionice -c2 -n7 nice -n19 awk -v start="[07/Jun/2021:17:49:25" -v end="[07/Jun/2021:19:00:00" '$4 > start && $4 < end' access_log.txt | awk '{print $1}' | sort | uniq -c`
  - `sudo ionice -c2 -n7 nice -n19 awk -v start="07/Jun/2021:10:25:35" -v end="07/Jun/2021:11:25:35" '{ gsub(/\[/, "", $4); } $4 > start && $4 < end' access_log.txt | awk '{print $1}' | sort | uniq -c`

```bash
  6 127.0.0.1
```

### アクセス先URIを集計

- `ionice -c2 -n7 nice -n19 cat access_log.txt | awk -F\" '{print $2}'  |sort | uniq -c`

```bash
242 GET / HTTP/1.0
  3 GET /bulletins HTTP/1.1
```

- 指定時間から最新まで
  - `ionice -c2 -n7 nice -n19 awk -v date="[07/Jun/2021:17:49:25" '$4 > date' access_log.txt | awk -F\" '{print $2}'  |sort | uniq -c`
- 指定範囲内
  - `sudo ionice -c2 -n7 nice -n19 awk -v start="[07/Jun/2021:17:49:25" -v end="[07/Jun/2021:19:00:00" '$4 > start && $4 < end' access_log.txt | awk -F\" '{print $2}'  |sort | uniq -c`
  - `sudo ionice -c2 -n7 nice -n19 awk -v start="07/Jun/2021:10:25:35" -v end="07/Jun/2021:11:25:35" '{ gsub(/\[/, "", $4); } $4 > start && $4 < end' access_log.txt | awk -F\" '{print $2}'  |sort | uniq -c`

```bash
  6 GET / HTTP/1.0
```

### リファラーを集計

- `ionice -c2 -n7 nice -n19 cat access_log.txt | awk -F\" '{print $4}'|sort | uniq -c`

```bash
245 -
```

- 指定時間から最新まで
  - `ionice -c2 -n7 nice -n19 awk -v date="[07/Jun/2021:17:49:25" '$4 > date' access_log.txt | awk -F\" '{print $4}'|sort | uniq -c`
- 指定範囲内
  - `sudo ionice -c2 -n7 nice -n19 awk -v start="[07/Jun/2021:17:49:25" -v end="[07/Jun/2021:19:00:00" '$4 > start && $4 < end' access_log.txt | awk -F\" '{print $4}'  |sort | uniq -c`
  - `sudo ionice -c2 -n7 nice -n19 awk -v start="07/Jun/2021:10:25:35" -v end="07/Jun/2021:11:25:35" '{ gsub(/\[/, "", $4); } $4 > start && $4 < end' access_log.txt | awk -F\" '{print $4}'  |sort | uniq -c`

```bash
  6 -
```

### User-agentヘッダを集計

- `ionice -c2 -n7 nice -n19 cat access_log.txt | awk -F\" '{print $6}'|sort | uniq -c`

```bash
  3 Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36
242 check_http/v2.3.3
```

- 指定時間から最新まで
  - `ionice -c2 -n7 nice -n19 awk -v date="[07/Jun/2021:17:49:25" '$4 > date' access_log.txt | awk -F\" '{print $6}'|sort | uniq -c`
- 指定範囲内
  - `sudo ionice -c2 -n7 nice -n19 awk -v start="[07/Jun/2021:17:49:25" -v end="[07/Jun/2021:19:00:00" '$4 > start && $4 < end' access_log.txt | awk -F\" '{print $6}'|sort | uniq -c`
  - `sudo ionice -c2 -n7 nice -n19 awk -v start="07/Jun/2021:10:25:35" -v end="07/Jun/2021:11:25:35" '{ gsub(/\[/, "", $4); } $4 > start && $4 < end' access_log.txt | awk -F\" '{print $6}'|sort | uniq -c`

```bash
  6 check_http/v2.3.3
```

## MySQLログ集計

- ログ確認
  - `/etc/my.cnf``/etc/my.cnf.d/mysql-server.cnf`

## おまけ

- よくタイムスタンプの月省略が分からなくなるので

```bash
Jan: January (1月)
Feb: February (2月)
Mar: March (3月)
Apr: April (4月)
May: May (5月)
Jun: June (6月)
Jul: July (7月)
Aug: August (8月)
Sep: September (9月)
Oct: October (10月)
Nov: November (11月)
Dec: December (12月)
```

- Alfredを使っているなら`{isodate:dd/MMM/yyyy:HH:mm:ss}`で出力する事も
