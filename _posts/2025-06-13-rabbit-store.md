# Rabbit Store

Demonstrate your web application testing skills and the basics of Linux to escalate your privileges.

Difficulty: Medium

## Enumeration

### Nmap

```bash
└─$ rustscan -a rabbit.thm -- -A
PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3f:da:55:0b:b3:a9:3b:09:5f:b1:db:53:5e:0b:ef:e2 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBXuyWp8m+y9taS8DGHe95YNOsKZ1/LCOjNlkzNjrnqGS1sZuQV7XQT9WbK/yWAgxZNtBHdnUT6uSEZPbfEUjUw=
|   256 b7:d3:2e:a7:08:91:66:6b:30:d2:0c:f7:90:cf:9a:f4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILcGp6ztslpYtKYBl8IrBPBbvf3doadnd5CBsO+HFg5M
80/tcp    open  http    syn-ack ttl 63 Apache httpd 2.4.52
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://cloudsite.thm/
4369/tcp  open  epmd    syn-ack ttl 63 Erlang Port Mapper Daemon
| epmd-info: 
|   epmd_port: 4369
|   nodes: 
|_    rabbit: 25672
25672/tcp open  unknown syn-ack ttl 63
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
TCP/IP fingerprint:

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   62.41 ms 10.11.0.1
2   62.60 ms rabbit.thm (10.10.255.200)
```

Right from the get go, I can see port 80 redirects to [`http://cloudsite.thm/`](http://cloudsite.thm/) so I add it to `/etc/hosts` and start browsing. Here, I discover the website has a login functionality, so I create an account and login, to see that I am assigned a jwt cookie that when decoded has a `"subscription": "inactive"` field. I can’t just modify it, because I don’t have the signature yet. But, for now, I’ll try some more numeration. Both vhosts and directories.

### Dirsearch

```bash
└─$ ffuf -u 'http://storage.cloudsite.thm/FUZZ' -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt -mc all -t 100 -ic -fc 404 
 :: Method           : GET
 :: URL              : http://storage.cloudsite.thm/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 100
 :: Matcher          : Response status: all
 :: Filter           : Response status: 404
________________________________________________

images                  [Status: 301, Size: 331, Words: 20, Lines: 10, Duration: 70ms]
css                     [Status: 301, Size: 328, Words: 20, Lines: 10, Duration: 68ms]
js                      [Status: 301, Size: 327, Words: 20, Lines: 10, Duration: 68ms]
assets                  [Status: 301, Size: 331, Words: 20, Lines: 10, Duration: 68ms]
javascript              [Status: 301, Size: 335, Words: 20, Lines: 10, Duration: 79ms]
fonts                   [Status: 301, Size: 330, Words: 20, Lines: 10, Duration: 65ms]
server-status           [Status: 403, Size: 286, Words: 20, Lines: 10, Duration: 63ms]
:: Progress: [17769/17769] :: Job [1/1] :: 1597 req/sec :: Duration: [0:00:11] :: Errors: 0 ::

└─$ ffuf -u 'http://storage.cloudsite.thm/api/FUZZ' -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt -mc all -t 100 -ic -fc 404
 :: Method           : GET
 :: URL              : http://storage.cloudsite.thm/api/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-directories-lowercase.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 100
 :: Matcher          : Response status: all
 :: Filter           : Response status: 404
________________________________________________

uploads                 [Status: 401, Size: 32, Words: 3, Lines: 1, Duration: 165ms]
login                   [Status: 405, Size: 36, Words: 4, Lines: 1, Duration: 196ms]
register                [Status: 405, Size: 36, Words: 4, Lines: 1, Duration: 184ms]
docs                    [Status: 403, Size: 27, Words: 2, Lines: 1, Duration: 195ms]
:: Progress: [17769/17769] :: Job [1/1] :: 1278 req/sec :: Duration: [0:00:14] :: Errors: 0 ::

```

### Vhost Scanner

```bash
└─$ ~/tools/offensivesecurity/vhost-fuzzer.sh cloudsite.thm /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://cloudsite.thm 280,281,282,283,284,285,286,287,288,289,290,291,292,293,294,295,296,297,298,299,300,301,302,303,304,305,306,307,308,309,310,311,312,313,314,315
:: Method           : GET
 :: URL              : http://cloudsite.thm
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 :: Header           : Host: FUZZ.cloudsite.thm
 :: Header           : User-Agent: PENTEST
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 280,281,282,283,284,285,286,287,288,289,290,291,292,293,294,295,296,297,298,299,300,301,302,303,304,305,306,307,308,309,310,311,312,313,314,315
________________________________________________

storage                 [Status: 200, Size: 9039, Words: 3183, Lines: 263, Duration: 61ms]
Storage                 [Status: 200, Size: 9039, Words: 3183, Lines: 263, Duration: 64ms]
STORAGE                 [Status: 200, Size: 9039, Words: 3183, Lines: 263, Duration: 62ms]

```

Vhost scanner and Dirsearch did not reveal much, but capturing my login and register request using burp, I can see there’s an api called, so fuzzing apis I had more luck. But I was still not able to get anywhere without an active subscription. So let’s try to create an account with a `"subscription": "inactive"` field and see if it works.

![image.png](Rabbit%20Store%2021156e3f2ea5804097c1fe4c7e39b012/image.png)

It looks like we’re allowed to register an active user this way. Let’s see if we can actually log in using this user, and what we’re provided with.

![image.png](Rabbit%20Store%2021156e3f2ea5804097c1fe4c7e39b012/image%201.png)

Not much, but it is a way in. And we do have an active token now. Let’s see what else we can read.