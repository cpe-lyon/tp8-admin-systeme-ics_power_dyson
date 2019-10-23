# TP 8 - Ansible

## *Références* : 
- Documentation d’Ansible : https://docs.ansible.com/ansible/latest/index.html
- Vidéos Hackademy, très simples et bien faites 
- Bonne présentation d’Ansible : https://www.youtube.com/watch?v=jePp5ZP1n14
(jusqu’à 32 min)
- Vidéo d’introduction à Ansible par Grafikart : https://www.youtube.com/watch?v=DwNapBHypE8
- SSH : https://www.ssh.com/ssh/copy-id - Gestion des “sudo-ers” : cours 4

## Questions:
### *Qu’est ce qu’un inventaire ?*

Ansible fonctionne avec plusieurs systèmes de l’infrastructure en même temps. Pour ce faire, il sélectionne des parties des systèmes répertoriés dans l’inventaire d’Ansible, qui sont enregistrées par défaut à l’emplacement /etc/ansible/hosts.

### *Qu’est ce qu’un module ?*

Un module permet de contrôler en général les ressources du systèmes. En d’autres termes, il peut contrôler les services, les packages, les fichiers ou peut même gérer l'exécution des commandes systèmes.

### *Qu’est ce qu’un playbook?*

Un playbook est composé des bases pour une bonne gestion de configuration ou alors pour le déploiement système sur de multiples machines. Il permet notamment d’automatiser des tâches dites ad-hoc telles que des commandes de gestion de fichier, d’authentification...

## *Exercice 1. Configuration nécessaire* 

*Deux machines virtuelles : 
• une machine de supervision / contrôle, sur laquelle est installé Ansible, 
• une machine supervisée ou nœud ou serveur.*

Pour installer Ansible :
<pre>
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
</pre>

*Assurez-vous que les deux machines arrivent à communiquer (supprimez si besoin les configurations DHCP / DNS ou les règles Netfilter des TP précédents)* 

Les machines communiquent bien :
<pre>
12:23-abitbol@serveur:~$ ping 192.168.100.2
PING 192.168.100.2 (192.168.100.2) 56(84) bytes of data.
64 bytes from 192.168.100.2: icmp_seq=1 ttl=64 time=0.329 ms
64 bytes from 192.168.100.2: icmp_seq=2 ttl=64 time=0.678 ms
64 bytes from 192.168.100.2: icmp_seq=3 ttl=64 time=0.678 ms
64 bytes from 192.168.100.2: icmp_seq=4 ttl=64 time=0.640 ms
64 bytes from 192.168.100.2: icmp_seq=5 ttl=64 time=0.726 ms
^C
--- 192.168.100.2 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 76ms
rtt min/avg/max/mdev = 0.329/0.610/0.726/0.144 ms
</pre>

Quelques modifications de configuration sont nécessaires pour la suite : 

- modifiez le fichier /etc/ansible/hosts pour renseigner le nœud à superviser, ainsi que le fichier /etc/hosts pour lui attribuer le nom node1 (plus facile à manipuler qu’une adresse IP) 
Pour entrer dans le fichier à modifier :

>sudo nano /etc/ansible/hosts

Ajout de l’adresse IP du noeud :
<pre>
# This is the default ansible 'hosts' file.
#
# It should live in /etc/ansible/hosts
#
#   - Comments begin with the '#' character
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

# Ex 1: Ungrouped hosts, specify before any group headers.

## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10
192.168.100.2
# Ex 2: A collection of hosts belonging to the 'webservers' group

## [webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110
</pre>

Puis modification du fichier /etc/hosts pour attribuer le nom node1 à l’adresse IP -> 192.168.100.2
<pre>
127.0.0.1 localhost
127.0.1.1 serveur
192.168.100.2 node1 #on ajoute l’ip ainsi que son nom
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
</pre>

- toutes les communications entre la machine de supervision et les nœuds se font par SSH. Pour éviter d’avoir à taper un mot de passe à chaque connexion à un nœud, il est nécessaire de mettre de générer une clé SSH et de la déployer sur les nœuds  

Génération de la clé ssh :
<pre>
13:24-abitbol@serveur:~$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/abitbol/.ssh/id_rsa):
Created directory '/home/abitbol/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/abitbol/.ssh/id_rsa.
Your public key has been saved in /home/abitbol/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:I1qwYBy0zH/l+ZDcbcELeyFT4DseWl1Cu295OwnitNk abitbol@client
The key's randomart image is:
+---[RSA 2048]----+
| .o       ..o    |
| + o     . + .   |
|  B .   . = * .  |
| . o o + + O B   |
|    o + S B B    |
|     + . B B o . |
|    .   . = = = o|
|           + E +.|
|               ..|
+----[SHA256]-----+

</pre>

Copie de la clé publique sur votre serveur distant :

<pre>
ssh-copy-id abitbol@192.168.100.2
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/abitbol/.ssh/id_rsa.pub"
The authenticity of host '192.168.100.2 (192.168.100.2)' can't be established.
ECDSA key fingerprint is SHA256:N3vWU5uqHLnu3IUxkl8o4PEIprBT/mz7Q9LlivnNfA0.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
abitbol@192.168.100.2's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'abitbol@192.168.100.2'"
and check to make sure that only the key(s) you wanted were added.

</pre>

**Il est recommandé de ne pas se connecter en root sur les nœuds**
https://www.hostinger.fr/tutoriels/generer-cle-ssh/

## *Exercice 2. Tâches à réaliser*

1. Si l’on n’est pas connecté à un nœud en tant que root, on aura besoin d’exécuter certaines commandes à l’aide de sudo. Pour cela, modifiez la section [privilege_escalation] du fichier de configuration /etc/ansible/ansible.cfg 

<pre>
[privilege_escalation]
become=True
become_method=sudo
#become_user=root
#become_ask_pass=Falsev
</pre>

2. Si l’on n’est pas connecté à un nœud en tant que root, toutes les commandes sudo demanderont de saisir un mot de passe. Pour éviter ceci, modifiez la configuration sudo pour que l’utilisateur sur le nœud n’ait pas à saisir de mot de passe quand il utilise sudo 
<pre>
abitbol ALL=(ALL:ALL) NOPASSWD:ALL
</pre>
3. Utilisez le module ping d’Ansible pour valider la configuration 

4. Utilisez le module setup pour récupérer la liste des périphériques (disques, cartes réseau…) présents sur les nœuds 

5. Créez un premier playbook qui installe git sur le nœud 

6. Créez un second playbook qui installe tmux et screen en utilisant une liste pour éviter de dupliquer inutilement les commandes 

7. Modifiez ce playbook pour créer un utilisateur ; cet utilisateur devra faire partie des ”sudoers” sur le serveur, et le fichier de règles sudo pour cet utilisateur devra être copié depuis la machine de supervision (cf. tuto de Grafikart) 8. Continuez le tuto de Grafikart (à partir de 21’)
