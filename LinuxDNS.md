# Installation et configuration d'un serveur DNS sur Linux

<br>

# 1. Installation de Bind9

* Se mettre en réseau NAT
* Mettre son IP en automatique (DHCP), déconnecter, reconnecter
* Pour installer Bind9, taper sur le terminal : 

```
sudo apt update
sudo apt install -y bind9 bind9utils bind9-doc dnsutils
```

* Se remettre en réseau interne
* Remettre son IP en manuel (statique), déconnecter, reconnecter

<br>

# 2. Named.conf.options (Configuration)

```
cd /etc/bind/
sudo nano named.conf.options
```
<img src = https://github.com/Fairskip/LinuxDNS/blob/main/Named.conf.options%201.png width = "700" height = "350">


* Remplacer le 0.0.0.0 par : 

```
forwarders {
172.20.0.254;
};

auth-nxdomain no; # conform to RCF1035
```

* Sortir et enregistrer les changements

<br>

# 3. Zone de recherche directe (Création & Configuration)

1. Se déplacer dans le dossier Bind


```
cd /etc/bind
```

<br>

2. Par précaution, faire un backup du fichier **named.conf.local**

```
sudo cp named.conf.local named.conf.local.backup
```

<br>

3. Ouvrir le fichier **named.conf.local** **:

```
sudo nano named.conf.local
```

<img src = https://github.com/Fairskip/LinuxDNS/blob/main/Zone%20principale%201.png width = "600" height = "250">

 * Ajouter la **zone de recherche directe**  : 
 
 Nom de la zone : wilders.lan
 Type de domaine : master
 Chemin & Nom du Fichier où se trouvera toutes les hôtes de la zone de recherche directe : /etc/bind/db.wilders.lan

```
zone "wilders.lan" IN {
	type master;
	file "/etc/bind/db.wilders.lan";
	allow-update { none; };
};
```
Note: Faire attention à la police du "". Il faut que ce soit "" et non « »

<br>

<img src = https://github.com/Fairskip/LinuxDNS/blob/main/3-named.config.local%20a.png width = "600" height = "250">


* Sortir sans oublier d'enregistrer les modifications*

<br>

4. Lister tous les fichiers de bind

<img src = https://github.com/Fairskip/LinuxDNS/blob/main/Zone%20principale%203%20ls.png width = "700" height = "200">

<br>

5. Faire une copy du fichier **db.0** et le nommer **db.wilders.lan**

```
sudo cp db.0 db.wilders.lan
```

<br>

6. Ouvrir le fichier **db.wilders.lan**

```
sudo nano db.wilders.lan
```

<img src = https://github.com/Fairskip/LinuxDNS/blob/main/Zone%20principale%204%20db%20wilders.lan.png width = "600" height = "250">

<br>

* Le modifier et le remplir :

```
@ IN SOA wilders.lan. root.wilders.lan. ( 
	2023051401 ; serial 
	21600 ; refresh after 6 hours 
	3600 ; retry after 1 hour 
	604800 ; expires after 1 week 
	86400 ) ; minimum TTL of 1 day
;	 
@ IN NS wilders.lan. 
server IN A 172.20.0.254 
windy IN A 172.20.0.10 
dns IN CNAME server.wilders.lan
```

<img src = https://github.com/Fairskip/LinuxDNS/blob/main/db.wilders.lan.png width = "600" height = "250">

* Sortir et enregistrer les modifications.

<br>

# 4. Zone de recherche inversée (Création & Configuration)

1. Retourner dans le fichier **named.conf.local** **:

```
sudo nano named.conf.local
```

<br>

* Ajouter la zone de recherche inversée

```
zone "0.20.172.in-addr.arpa" {
	type master;
	file "db.wilders.lan.reverse"; 
};
```

<img src = https://github.com/Fairskip/LinuxDNS/blob/main/3-named.config.local%20b.png width = "600" height = "250">

<br>

* Sortir dans oublier d'enregistrer les modification

<br>

2. Faire une copy du fichier **db.127** et le nommer **db.wilders.lan.reverse**

```
sudo cp db.127 db.wilders.lan.reverse
```

<br>

3. Ouvrir le fichier **db.wilders.lan**

```
sudo nano db.wilders.lan.reverse
```

<img src = https://github.com/Fairskip/LinuxDNS/blob/main/Zone%20reverse%202.png width = "600" height = "250">
 
<br>

* Le modifier et le remplir. 

```
@       IN      SOA     wilders.lan. root.wilders.lan. (
  1         ; Serial
  604800         ; Refresh
  86400         ; Retry
  2419200         ; Expire
  604800 )       ; Negative Cache TTL
;
@     IN      NS      wilders.lan.
254   IN      PTR     server.wilder.lan.
10    IN      PTR     Windy
```

<img src = https://github.com/Fairskip/LinuxDNS/blob/main/db.wilders.lan.reverse.png width = "600" height = "250">


* Sortir et enregistrer les modifications.

<br>

## 5. Configuration à tester et démarrer

<br>

1. Tester si les fichiers de configurations sont correctement ajoutés avec : 

> sudo named-checkzone nom-de-la-zone-enregistré-dans-named.conf.local path-du-fichier-de-la-zone

<br>

```
sudo named-checkzone wilders.lan /etc/bind/db.wilders.lan

sudo named-checkzone wilders.lan /etc/bind/db.wilders.lan.reverse
```

<img src = https://github.com/Fairskip/LinuxDNS/blob/main/checkzone.png width = "600" height = "250">

<br>

2. Start le service Bind

```
sudo service bind9 start
```

<br>

3. Voir le status de Bind

```
sudo service bind9 status
```

<img src = https://github.com/Fairskip/LinuxDNS/blob/main/4-bindstart_status.png width = "700" height = "350">

<br>

# 5. Test
### 1 - Test depuis Server  

* Résolution de Nom 
* Résolution inverse

<img src = https://github.com/Fairskip/LinuxDNS/blob/main/Test%20server.png width = "450" height = "600">

<br>

### 2 - Test depuis Windy (host windows) 

* Résolution de Nom 
* Résolution inverse

<img src = https://github.com/Fairskip/LinuxDNS/blob/main/Test%20Windy.png width = "450" height = "600">


