# Born2beRoot

## // INTRO

* Voici le Milestone 1, le projet : Born2beRoot

## /. HELP

* Voir le pdf de Born2beRoot qui explique parfaitement le projet.
* Fini à 125%


## /: MINI GUIDE


* Toujours choisir le meme mot de passe pour tout ( meme le passphrase )
* Si une commande ne marche pas, c'est peut être parce que tu ne l'as pas lancé avec 'sudo'...
* Si tu ne peux changer un fichier passe le en mode : chmod 740 <file.txt> ( n'oublie pas de le repasser en mode par default )
* Si tu ne veux faire les bonus, commence à regarder la section : BONUS
* Tu peux défiler de gauche à droite les commandes.

<command> | less				pouvoir defiler sur la page de la commande.
						'g' pour remonter et 'G' pour aller en bas


==================================== VIRTUAL BOX =====================================


Guest OS Install Setup Name: Born2beRoot
	username: ...
	password: ...


======================================= CODE =======================================


* Durant l'installation de Debian.

HOST NAME :

	username: ...42
	password: ...

USER NAME :

	compte utilisateur: ...
	login: ...
	password: ...
	passphrase: ...


====================================== VM ==========================================


	su -					se connecter en root:
	apt-get update/upgrade			installer les derniers MAJ
	apt install sudo			installer 'sudo'
	usermod -aG sudo <username>		ajoute <username> à <sudo>
	getent group sudo			voir les utilisateurs de sudo
	sudo visudo				ouverture protégé des fichiers sudoers

	Change dedans :
	# User privilege specification
	<username>	ALL=(ALL) ALL


====================================== VIM / GIT =====================================


Installer VIM / GIT :

	apt-get install vim/git			install vim et git.


======================================== SSH =======================================


	apt install openssh-server		installe ssh
	systemctl status ssh			voir le status ssh ( system control )
	
	Changer le port : ( faire gaffe c'est sshd_config avec un 'd' et pas ssh_config )
		vim /etc/ssh/sshd_config
		#Port22 ----> Port 4242
		grep Port /etc/ssh/sshd_config ( -> voir les changements )

	service ssh restart			appliquer les modifs


====================================== UFW =========================================


	apt-get install ufw			installe ufw
	ufw enable				mise en marche
	ufw status				voir le status
	ufw allow 4242				configure les ports
	ufw delete allow 22/tcp			supprimer le port par default de ssh

	Changer les ports actifs depuis Virtual Box :

	Aller sur Network ---> Port Forwarding ---> Add Port
	Changer le Host Port et le Guest Port en celui que tu souhaites (4242/4242)
	systemctl restart ssh			restart ssh


============================== SE CONNECTER AU PORT =================================


	ssh <username>@127.0.0.1 -p 4242	se connecte au port desiré -p ( port )

	127.0.0.1 ==> localhost


=================================== STRONG PASSWORD =================================


	install libpam-pwquality		installe pw-quality
	vim /etc/pam.d/common-password		ouvrir le fichier des règles
	Trouver la ligne :

		password  requisite     pam_pwquality.so  retry=3

	Et ajouter a la suite les uns des autres :
	```
	minlen=10
	ucredit=-1
	lcredit=-1
	dcredit=-1
	maxrepeat=3
	reject_username
	difok=7
	enforce_for_root
	```
	vim /etc/login.defs			ouvre un fichier de règles
	Trouver la ligne :

		PASS_MAX_DAYS 9999 PASS_MIN_DAYS 0 PASS_WARN_AGE 7
	
	Changer les valeurs de celles-ci respectivement en : 30 | 2 | 7

	reboot					applique les modifs.


===================================== CREATE A GROUP =================================


	groupadd <user42>			créer un groupe	user42
	getent group				voir les groupes

	groupadd				creates a new group.
	groupdel				deletes a group.

	gpasswd -a				adds a user to a group.
	gpasswd -d				removes a user from a group.

	groups					displays the groups of a user.


================================== SUDOERS PASSWORD =================================


	cd /var/log/sudo			aller dans var log sudo
	touch sudo.log				creer un fichier sudo.log
	vim /etc/sudoers			ouvrir le fichier et ajouter :

	Defaults	env_reset
	Defaults	mail_badpass
	Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/bin:/sbin:/bin"
	Defaults	badpass_message="Password is wrong, please try again!"
	Defaults	passwd_tries=3
	Defaults	logfile="/var/log/sudo/sudo.log"
	Defaults	log_input, log_output
	Defaults	requiretty


================================= MONITORING.SH =====================================


	install net-tools			installer net-tools
	cd /usr/local/bin/			aller a bin
	touch monitoring.sh			creer un fichier .sh
	chmod 740 monitoring.sh			changer les permissions
	Coller : (connecte toi en SSH depuis ton propre terminal et pas 
		sur la VM)

```
#!/bin/bash
arc=$(uname -a)
pcpu=$(grep "physical id" /proc/cpuinfo | sort | uniq | wc -l) 
vcpu=$(grep "^processor" /proc/cpuinfo | wc -l)
fram=$(free -m | awk '$1 == "Mem:" {print $2}')
uram=$(free -m | awk '$1 == "Mem:" {print $3}')
pram=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')
fdisk=$(df -BG | grep '^/dev/' | grep -v '/boot$' | awk '{ft += $2} END {print ft}')
udisk=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} END {print ut}')
pdisk=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} {ft+= $2} END {printf("%d"), ut/ft*100}')
cpul=$(top -bn1 | grep '^%Cpu' | cut -c 9- | xargs | awk '{printf("%.1f%%"), $1 + $3}')
lb=$(who -b | awk '$1 == "system" {print $3 " " $4}')
lvmu=$(if [ $(lsblk | grep "lvm" | wc -l) -eq 0 ]; then echo no; else echo yes; fi)
ctcp=$(ss -Ht state established | wc -l)
ulog=$(users | wc -w)
ip=$(hostname -I)
mac=$(ip link show | grep "ether" | awk '{print $2}')
cmds=$(journalctl _COMM=sudo | grep COMMAND | wc -l)
wall "	#Architecture: $arc
	#CPU physical: $pcpu
	#vCPU: $vcpu
	#Memory Usage: $uram/${fram}MB ($pram%)
	#Disk Usage: $udisk/${fdisk}Gb ($pdisk%)
	#CPU load: $cpul
	#Last boot: $lb
	#LVM use: $lvmu
	#Connections TCP: $ctcp ESTABLISHED
	#User log: $ulog
	#Network: IP $ip ($mac)
	#Sudo: $cmds cmd"
```

	cd /etc/sudoers				aller dans les fichiers sudoers
	sudo visudo				ouvrir sudoers
	sudo crontab -u root -e			open the crontab
	systemctl stop cron


================================ SIGNATURE ==========================================


==> ```shasum <VM'sname>.vdi```
* Faites pour récuperer la signature de ta VM. Elle est unique à chaque VM.


==================================== EVAL ===========================================


Les nouvelles commandes à apprendre pour l'EVAL :

	sudo chage -l username				examine les règles de mot de passe
	hostnamectl set-hostname <new>			changer le hostname courant.
	Redémarre la VM.

	sudo nano /etc/hosts				changer le nom du hostname
	lsblk						display partitions
	dpkg -l | grep sudo				montre que le sudo est installé


==================================== BONUS ==========================================


https://github.com/mcombeau/Born2beroot/blob/main/guide/bonus_debian.md

Le site WEB devra utiliser : PHP, MariaDB et Lighttpd.
Pour changer les partitions, il va falloir reconfigurer une VM et le faire manuellement.
( c'est très long et fastidieux. )

##	/:. EOF

Modifié le 9 Novembre 2024 par Juste.<br>
