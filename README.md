# Roger_Skyline
Roger Skyline project 42


iptables :
https://geekeries.org/2017/12/configuration-avancee-du-firewall-iptables/?cn-reloaded=1&cn-reloaded=1&cn-reloaded=1

ufw :
https://linuxize.com/post/how-to-setup-a-firewall-with-ufw-on-debian-9/

/etc/rc.local
contient scripts lu au boot

affichage état des ports :
ss -tulwn

Parametrage de Virtualbox pour la VM :
Gestionnaire réseau creation d'un réseau dissocie du serveur DHCP de la machine.
Creation d'un nouveau réseau virtuel "Réseau hôte privé" pour la VM

Packages installés : 
net-tools : Outills de gestion du réseau
iptables-persistent : Outils de gestion pare-feu
fail2ban : gestionnaire de règles de protection spécialisé
sendmail : Gestionnaire mails
apache2 : Gestionnaire serveur web

/*
- Vous devez changer le port par defaut du service SSH par celui de votre choix.
L’accès SSH DOIT se faire avec des publickeys. L’utilisateur root ne doit pas
pouvoir se connecter en SSH.
*/

parametrage SSH :
-> /etc/ssh/sshd_config
port -> 2222
PermitRootLogin -> no
PublicKeyAuthentification -> no
PasswordAuthentification -> no (apres generation d'une clef public)
ssh-keygen pour générer une clef public ssh sur la vraie machine a copier sur la vm dans .ssh/authorized_key

/*
- Nous ne voulons pas que vous utilisiez le service DHCP de votre machine. A vous
donc de la configurer afin qu’elle ait une IP fixe et un Netmask en /30.
*/

parametrage IP :
-> /etc/network/interfaces
allow-hotplug enp0s8 : second réseau créé precedement
iface enp0s8 inet static : IP static
address 192.168.56.3
netmask 255.255.255.252 : mask en /30 donc 30 bits sur 32 -> 3 addresses ip resultantes

/*
Vous devez créer un utilisateur non root pour vous connecter et travailler.
*/

adduser USER -> creation d'un nouvel utilisateur

/*
- Utilisez sudo pour pouvoir, depuis cet utilisateur, effectuer les operations demandant des droits speciaux.
*/

adduser USER sudo -> ajout de USER au sudoers
reboot pour confirmer

/*
Vous devez mettre en place des règles de pare-feu (firewall) sur le serveur avec
uniquement les services utilisés accessible en dehors de la VM.
*/

parametrage de /etc/interfaces/if-pre-up.d/iptables
instructions shell ->
sudo sudo iptables -F
sudo iptables -X
sudo iptables -t nat -F
sudo iptables -t nat -X
sudo iptables -t mangle -F
sudo iptables -t mangle -X
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp -i enp0s8 --dport 2222 -j ACCEPT
sudo iptables -A INPUT -p tcp -i enp0s8 --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp -i enp0s8 --dport 443 -j ACCEPT
sudo iptables -A OUTPUT -m conntrack ! --ctstate INVALID -j ACCEPT
sudo iptables -I INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -j LOG
sudo iptables -A FORWARD -j LOG
sudo iptables -I INPUT -p tcp --dport 80 -m connlimit --connlimit-above 10 --connlimit-mask 20 -j DROP
