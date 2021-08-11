# Domain Name System Security Extension `DNSSEC`
Implementation du DNS Security Extension suivant la hierachie DNS

Ce document est la transcription final des effort effectues dans la realisation de rapport de fin de formation en `Audit, Securite des Systemes et Reseaux Informatiques` sur le theme `Implementation d'une infrastructure a cle publique (PKI): Cas du DNSSEC`.

## Systeme D'exploitation
Pour la realisation de ce Projet nous avons installer nos serveurs sur :
* Ubuntu 16.04 LTS, [Download Here](https://releases.ubuntu.com/16.04/)

## Les outils
Pour l'installation des serveurs nous avons utiliser `BIND9`.

Voici comment l'installer :
```bash
sudo apt install bind9 bind9-utils haveged
```

## Architecture
Suivant la hierachie d'un syteme DNS nous avons mis en place 4 serveurs tels que :
* Serveur Root `10.0.0.1`
* Serveur TLD `10.0.0.3`
* Resolveur Recursif `10.0.0.8`
* Serveur de Nom `10.0.0.5`

Et pour finalier nous avons utiliser une machine client `10.0.0.10` pour faire les tests de resolutions de nom. 
L'architecture de l'implementation suis le schema suivant :

![DNSSEC](https://github.com/th3f0r3ign3r/DNSSEC/blob/main/dns-archi.png "Hierachical DNS")

## Configuration
Nous allons passer a la configuration des serveurs.

Pour la mise en place de ce projet nous avons installer ces serveurs sur des machines virtuels sous `VMWARE WORKSTATION` sous linux `Ubuntu 16.04 LTS`.

Dans un premier temps nous allons configurer les serveurs differement et ensuite nous allons passer a la generation des cles public/prive et ensuite a la signature de zones.


## Serveur Root `.`

#### 1 - Modifier le fichier `/etc/bind/named.conf.options` :

Pour qu'il ressemble a ceci :
```conf
options {
	directory "/var/cache/bind";

	dnssec-validation no;
	//dnssec-enable yes;

	auth-nxdomain no;    # conform to RFC1035
	listen-on { any; };

};
```

#### 2 - Modifier le fichier `/etc/bind/named.conf.local` :
```
//include "/etc/bind/zones.rfc1918";

zone "." {
	type master;
	file "/etc/bind/root.srv";
	allow-transfer { any; };
        allow-query { any; };
};

zone "0.0.10.in-addr.arpa" {
	type master;
	file "/etc/bind/root.srv.inv";
	allow-transfer { any; };
        allow-query { any; };
};
```

#### 3 - Configuration des fichier de zone :

* Creer le ficher `root.srv`
```bash
touch root.srv
```

* Modifier le fichier et y inserer la configuration suivante :
```
;
; BIND data file for local loopback interface
;
$TTL	604800
@	IN	SOA	root. srvmaster.root. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	root.
root.	A	10.0.0.1
```

* Creer le fichier `root.srv.inv`
```bash
touch root.srv.inv
```

* modifier le fichier et y inserer la config suivante :
```
;
; BIND reverse data file for local loopback interface
;
$TTL	604800
@	IN	SOA	root. srvmaster.root. (
			      1		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	root.
1	IN	PTR	root.
```

#### 4 - Commenter la zone root "." dans le fichier `/etc/bind/named.conf.default-zones` comme suit :
```
//zone "." {
//	type hint;
//	file "/etc/bind/db.root";
//};
```

#### 5 - Fixer l'addresse IP sur la machime comme suit :
`ens33` ici est mis pour designer mon interface `Ethernet` dans certains cas on trouveras `eth0` 
```
auto ens33
iface ens33 inet static
	address 10.0.0.1
	netmask	255.255.255.0
	broadcast 10.0.0.255
	dns-nameserver 10.0.0.1
```

#### 6 - Vider le fichier `/etc/resolv.conf` et y ajouter cette ligne :
```
nameserver 10.0.0.1
```


## Serveur TLD  `.com`

#### 1 - Modifier le fichier `/etc/bind/named.conf.options` :

Pour qu'il ressemble a ceci :
```conf
options {
	directory "/var/cache/bind";

	dnssec-validation auto;
	dnssec-enable yes;

	auth-nxdomain no;    # conform to RFC1035
	listen-on { any; };

};
```

#### 2 - Modifier le fichier `/etc/bind/named.conf.local` :
```
//include "/etc/bind/zones.rfc1918";

zone "com" {
	type master;
	file "/etc/bind/db.com";
	allow-transfer { any; };
        allow-query { any; };
};

zone "0.0.10.in-addr.arpa" {
	type master;
	file "/etc/bind/db.com.inv";
	allow-transfer { any; };
        allow-query { any; };
};
```

#### 3 - Configuration des fichier de zone :

* Creer le ficher `db.com`
```bash
touch db.com
```

* Modifier le fichier et y inserer la configuration suivante :
```
;
; BIND data file for local loopback interface
;
$TTL	604800
@	IN	SOA	com. root.com. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	com.
com.	A	10.0.0.3
```

* Creer le fichier `db.com.inv`
```bash
touch db.com.inv
```

* modifier le fichier et y inserer la config suivante :
```
;
; BIND reverse data file for local loopback interface
;
$TTL	604800
@	IN	SOA	com. root.com. (
			      1		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	com.
3	IN	PTR	com.
```

#### 4 - Dans le fichier `/etc/bind/named.conf.default-zones` assurer vous que le fichier que la zone root `.` n'est pas commenter ou effacer :
```
zone "." {
	type hint;
	file "/etc/bind/db.root";
};
```
Creer ou modifier le fichier `db.root` comme suit et :
```
;       This file holds the information on root name servers needed to
;       initialize cache of Internet domain name servers
;       (e.g. reference this file in the "cache  .  <file>"
;       configuration file of BIND domain name servers).
;
;       This file is made available by InterNIC
;       under anonymous FTP as
;           file                /domain/named.cache
;           on server           FTP.INTERNIC.NET
;       -OR-                    RS.INTERNIC.NET
;
;       last update:    February 17, 2016
;       related version of root zone:   2016021701
;
; formerly NS.INTERNIC.NET
;
.		3600000		IN	NS	root.
root.		3600000		A	10.0.0.1    
;
; End of file
```

#### 5 - Fixer l'addresse IP sur la machime comme suit :
`ens33` ici est mis pour designer mon interface `Ethernet` dans certains cas on trouveras `eth0` 
```
auto ens33
iface ens33 inet static
	address 10.0.0.3
	netmask	255.255.255.0
	broadcast 10.0.0.255
	dns-nameserver 10.0.0.3
```

#### 6 - Vider le fichier `/etc/resolv.conf` et y ajouter cette ligne :
```
nameserver 10.0.0.3
```

#### 7 - Declaration du TLD `.com`
Dans le fichier de zone du serveur root `root.srv` sautez une ligne et ajoutez la config suivante :
```
$ORIGIN	com.
@	IN	NS	com.
com.	A	10.0.0.3
```


## Serveur de nom `example.com`

#### 1 - Modifier le fichier `/etc/bind/named.conf.options` :

Pour qu'il ressemble a ceci :
```conf
options {
	directory "/var/cache/bind";

	dnssec-validation auto;
	dnssec-enable yes;

	auth-nxdomain no;    # conform to RFC1035
	listen-on { any; };

};
```

#### 2 - Modifier le fichier `/etc/bind/named.conf.local` :
```
//include "/etc/bind/zones.rfc1918";

zone "example.com" {
	type master;
	file "/etc/bind/example.com";
	allow-transfer { any; };
        allow-query { any; };
};

zone "0.0.10.in-addr.arpa" {
	type master;
	file "/etc/bind/example.com.inv";
	allow-transfer { any; };
        allow-query { any; };
};
```

#### 3 - Configuration des fichier de zone :

* Creer le ficher `example.com`
```bash
touch example.com
```

* Modifier le fichier et y inserer la configuration suivante :
```
;
; BIND data file for local loopback interface
;
$TTL	604800
@	IN	SOA	example.com. root.example.com. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	example.com.
example.com.	A	10.0.0.3
```

* Creer le fichier `db.com.inv`
```bash
touch example.com.inv
```

* modifier le fichier et y inserer la config suivante :
```
;
; BIND reverse data file for local loopback interface
;
$TTL	604800
@	IN	SOA	example.com. root.example.com. (
			      1		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	example.com.
3	IN	PTR	example.com.
```

#### 4 - Dans le fichier `/etc/bind/named.conf.default-zones` assurer vous que le fichier que la zone root `.` n'est pas commenter ou effacer :
```
zone "." {
	type hint;
	file "/etc/bind/db.root";
};
```
Creer ou modifier le fichier `db.root` comme suit et :
```
;       This file holds the information on root name servers needed to
;       initialize cache of Internet domain name servers
;       (e.g. reference this file in the "cache  .  <file>"
;       configuration file of BIND domain name servers).
;
;       This file is made available by InterNIC
;       under anonymous FTP as
;           file                /domain/named.cache
;           on server           FTP.INTERNIC.NET
;       -OR-                    RS.INTERNIC.NET
;
;       last update:    February 17, 2016
;       related version of root zone:   2016021701
;
; formerly NS.INTERNIC.NET
;
.		3600000		IN	NS	root.
root.		3600000		A	10.0.0.1    
;
; End of file
```

#### 5 - Fixer l'addresse IP sur la machime comme suit :
`ens33` ici est mis pour designer mon interface `Ethernet` dans certains cas on trouveras `eth0` 
```
auto ens33
iface ens33 inet static
	address 10.0.0.5
	netmask	255.255.255.0
	broadcast 10.0.0.255
	dns-nameserver 10.0.0.5
```

#### 6 - Vider le fichier `/etc/resolv.conf` et y ajouter cette ligne :
```
nameserver 10.0.0.5
```

#### 7 - Declaration du TLD `example.com`
Dans le fichier de zone du serveur TLD `db.com` sautez une ligne et ajoutez la config suivante :
```
$ORIGIN	example.com.
@	IN	NS	example.com.
example.com.	A	10.0.0.5
```

## Serveur Recursif

#### 1 - Modifier le fichier `/etc/bind/named.conf` :

Vider le contenu du fichier afin qu'il ressemble a ceci :
```
options {
	directory "/var/cache/bind";

	auth-nxdomain no;    # conform to RFC1035
	listen-on { any; };
	dnssec-validation auto;
	dnssec-enable yes;
	//dnssec-must-be-secure root yes;
};

zone "." {
	type hint;
	file "/etc/bind/db.root";
};
```

#### 2 - Creer ou modifier le fichier `db.root` comme suit et :
```
;       This file holds the information on root name servers needed to
;       initialize cache of Internet domain name servers
;       (e.g. reference this file in the "cache  .  <file>"
;       configuration file of BIND domain name servers).
;
;       This file is made available by InterNIC
;       under anonymous FTP as
;           file                /domain/named.cache
;           on server           FTP.INTERNIC.NET
;       -OR-                    RS.INTERNIC.NET
;
;       last update:    February 17, 2016
;       related version of root zone:   2016021701
;
; formerly NS.INTERNIC.NET
;
.		3600000		IN	NS	root.
root.		3600000		A	10.0.0.1    
;
; End of file
```

#### 3 - Fixer l'addresse IP sur la machime comme suit :
`ens33` ici est mis pour designer mon interface `Ethernet` dans certains cas on trouveras `eth0` 
```
auto ens33
iface ens33 inet static
	address 10.0.0.8
	netmask	255.255.255.0
	broadcast 10.0.0.255
```

#### 4 - Vider le fichier `/etc/resolv.conf` afin qu'il ne contienne aucun enregistrement, car cela permettra de rediriger toutes les requetes vers le serveur `BIND`.



## Generation des cles et Signature de zone
Comming Soon
