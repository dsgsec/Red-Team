# WHOIS

On peut considérer le WHOIS comme les "pages blanches" des noms de domaine. Il s'agit d'un protocole de requête/réponse orienté transaction basé sur TCP qui écoute sur le port TCP 43 par défaut. Nous pouvons l'utiliser pour interroger des bases de données contenant des noms de domaine, des adresses IP ou des systèmes autonomes et fournir des services d'information aux internautes. Le protocole est défini dans la RFC 3912. Le premier répertoire WHOIS a été créé au début des années 1970 par Elizabeth Feinler et son équipe travaillant au Network Information Center (NIC) de l'Université de Stanford. Avec son équipe, ils ont créé des domaines divisés en catégories basées sur l'adresse physique d'un ordinateur. Nous pouvons en savoir plus sur l'histoire fascinante du WHOIS ici.

Les recherches de domaine WHOIS nous permettent de récupérer des informations sur le nom de domaine d'un domaine déjà enregistré. L'Internet Corporation of Assigned Names and Numbers (ICANN) exige que les bureaux d'enregistrement accrédités saisissent les coordonnées du titulaire, les dates de création et d'expiration du domaine et d'autres informations dans la base de données Whois immédiatement après l'enregistrement d'un domaine. En termes simples, la base de données Whois est une liste consultable de tous les domaines actuellement enregistrés dans le monde.

Les recherches WHOIS étaient initialement effectuées à l'aide d'outils de ligne de commande. De nos jours, de nombreux outils Web existent, mais les options de ligne de commande nous donnent souvent le plus de contrôle sur nos requêtes et aident à filtrer et trier la sortie résultante. L'utilitaire de ligne de commande WHOIS pour Windows ou Linux de Sysinternals WHOIS sont nos outils préférés pour la collecte d'informations. Cependant, il existe certaines versions en ligne comme whois.domaintools.com que nous pouvons également utiliser.

Nous obtiendrions la réponse suivante de la commande précédente pour exécuter une recherche whois sur le domaine facebook.com. Un exemple de cette commande whois est :

```
dsgsec@htb[/htb]$ export TARGET="facebook.com" # Assign our target to an environment variable
dsgsec@htb[/htb]$ whois $TARGET

Domain Name: FACEBOOK.COM
Registry Domain ID: 2320948_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.registrarsafe.com
Registrar URL: https://www.registrarsafe.com
Updated Date: 2021-09-22T19:33:41Z
Creation Date: 1997-03-29T05:00:00Z
Registrar Registration Expiration Date: 2030-03-30T04:00:00Z
Registrar: RegistrarSafe, LLC
Registrar IANA ID: 3237
Registrar Abuse Contact Email: abusecomplaints@registrarsafe.com
Registrar Abuse Contact Phone: +1.6503087004
Domain Status: clientDeleteProhibited https://www.icann.org/epp#clientDeleteProhibited
Domain Status: clientTransferProhibited https://www.icann.org/epp#clientTransferProhibited
Domain Status: clientUpdateProhibited https://www.icann.org/epp#clientUpdateProhibited
Domain Status: serverDeleteProhibited https://www.icann.org/epp#serverDeleteProhibited
Domain Status: serverTransferProhibited https://www.icann.org/epp#serverTransferProhibited
Domain Status: serverUpdateProhibited https://www.icann.org/epp#serverUpdateProhibited
Registry Registrant ID:
Registrant Name: Domain Admin
Registrant Organization: Facebook, Inc.
Registrant Street: 1601 Willow Rd
Registrant City: Menlo Park
Registrant State/Province: CA
Registrant Postal Code: 94025
Registrant Country: US
Registrant Phone: +1.6505434800
Registrant Phone Ext:
Registrant Fax: +1.6505434800
Registrant Fax Ext:
Registrant Email: domain@fb.com
Registry Admin ID:
Admin Name: Domain Admin
Admin Organization: Facebook, Inc.
Admin Street: 1601 Willow Rd
Admin City: Menlo Park
Admin State/Province: CA
Admin Postal Code: 94025
Admin Country: US
Admin Phone: +1.6505434800
Admin Phone Ext:
Admin Fax: +1.6505434800
Admin Fax Ext:
Admin Email: domain@fb.com
Registry Tech ID:
Tech Name: Domain Admin
Tech Organization: Facebook, Inc.
Tech Street: 1601 Willow Rd
Tech City: Menlo Park
Tech State/Province: CA
Tech Postal Code: 94025
Tech Country: US
Tech Phone: +1.6505434800
Tech Phone Ext:
Tech Fax: +1.6505434800
Tech Fax Ext:
Tech Email: domain@fb.com
Name Server: C.NS.FACEBOOK.COM
Name Server: B.NS.FACEBOOK.COM
Name Server: A.NS.FACEBOOK.COM
Name Server: D.NS.FACEBOOK.COM
DNSSEC: unsigned

<SNIP>
```
À partir de cette sortie, nous avons recueilli les informations suivantes:

| | |
| --- | --- |
| Organisation | Facebook, Inc. |
| Emplacements | États-Unis, 94025 Menlo Park, Californie, 1601 Willo Rd |
| Adresse e-mail de domaine | domaine@fb.com |
| Adresse e-mail du registraire | abusecomplaints@registrarsafe.com |
| Numéro de téléphone | +1.6505434800 |
| Langue | Anglais (États-Unis) |
| Registraire | RegistrarSafe, LLC |
| Nouveau domaine | fb.com |
| [DNSSEC](https://en.wikipedia.org/wiki/Domain_Name_System_Security_Extensions) | [non signé](https://aws.amazon.com/blogs/networking-and-content-delivery/configuring-dnssec-signing-and-validation-with-amazon-route-53) |
| Serveurs de noms | A.NS.FACEBOOK.COM |