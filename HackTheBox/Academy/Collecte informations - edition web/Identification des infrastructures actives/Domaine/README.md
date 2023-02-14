# Domaine

Nous pouvons effectuer une énumération active des sous-domaines en sondant l'infrastructure gérée par l'organisation cible ou les serveurs DNS tiers que nous avons précédemment identifiés. Dans ce cas, la quantité de trafic généré peut conduire à la détection de nos activités de reconnaissance.

## ZoneTransfer

Le transfert de zone est la façon dont un serveur DNS secondaire reçoit des informations du serveur DNS principal et les met à jour. L'approche maître-esclave est utilisée pour organiser les serveurs DNS au sein d'un domaine, les esclaves recevant des informations DNS mises à jour du DNS maître. Le serveur DNS maître doit être configuré pour activer les transferts de zone à partir de serveurs DNS secondaires (esclaves), bien que cela puisse être mal configuré.

Par exemple, nous utiliserons le service https://hackertarget.com/zone-transfer/ et le domaine zonetransfer.me pour avoir une idée des informations pouvant être obtenues via cette technique.

### 1. identifier les Nameservers
```
dsgsec@htb[/htb]$ nslookup -type=NS zonetransfer.me

Server:		10.100.0.1
Address:	10.100.0.1#53

Non-authoritative answer:
zonetransfer.me	nameserver = nsztm2.digi.ninja.
zonetransfer.me	nameserver = nsztm1.digi.ninja.
```

2. Effectuer le transfert de zone en utilisant les paramètres -type=any et -query=AXFR

### 2. Test pour le transfert de zone ANY et AXFR
```
dsgsec@htb[/htb]$ nslookup -type=any -query=AXFR zonetransfer.me nsztm1.digi.ninja

Server:		nsztm1.digi.ninja
Address:	81.4.108.41#53

zonetransfer.me
	origin = nsztm1.digi.ninja
	mail addr = robin.digi.ninja
	serial = 2019100801
	refresh = 172800
	retry = 900
	expire = 1209600
	minimum = 3600
zonetransfer.me	hinfo = "Casio fx-700G" "Windows XP"
zonetransfer.me	text = "google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA"
zonetransfer.me	mail exchanger = 0 ASPMX.L.GOOGLE.COM.
zonetransfer.me	mail exchanger = 10 ALT1.ASPMX.L.GOOGLE.COM.
zonetransfer.me	mail exchanger = 10 ALT2.ASPMX.L.GOOGLE.COM.
zonetransfer.me	mail exchanger = 20 ASPMX2.GOOGLEMAIL.COM.
zonetransfer.me	mail exchanger = 20 ASPMX3.GOOGLEMAIL.COM.
zonetransfer.me	mail exchanger = 20 ASPMX4.GOOGLEMAIL.COM.
zonetransfer.me	mail exchanger = 20 ASPMX5.GOOGLEMAIL.COM.
Name:	zonetransfer.me
Address: 5.196.105.14
zonetransfer.me	nameserver = nsztm1.digi.ninja.
zonetransfer.me	nameserver = nsztm2.digi.ninja.
_acme-challenge.zonetransfer.me	text = "6Oa05hbUJ9xSsvYy7pApQvwCUSSGgxvrbdizjePEsZI"
_sip._tcp.zonetransfer.me	service = 0 0 5060 www.zonetransfer.me.
14.105.196.5.IN-ADDR.ARPA.zonetransfer.me	name = www.zonetransfer.me.
asfdbauthdns.zonetransfer.me	afsdb = 1 asfdbbox.zonetransfer.me.
Name:	asfdbbox.zonetransfer.me
Address: 127.0.0.1
asfdbvolume.zonetransfer.me	afsdb = 1 asfdbbox.zonetransfer.me.
Name:	canberra-office.zonetransfer.me
Address: 202.14.81.230
cmdexec.zonetransfer.me	text = "; ls"
contact.zonetransfer.me	text = "Remember to call or email Pippa on +44 123 4567890 or pippa@zonetransfer.me when making DNS changes"
Name:	dc-office.zonetransfer.me
Address: 143.228.181.132
Name:	deadbeef.zonetransfer.me
Address: dead:beaf::
dr.zonetransfer.me	loc = 53 20 56.558 N 1 38 33.526 W 0.00m 1m 10000m 10m
DZC.zonetransfer.me	text = "AbCdEfG"
email.zonetransfer.me	naptr = 1 1 "P" "E2U+email" "" email.zonetransfer.me.zonetransfer.me.
Name:	email.zonetransfer.me
Address: 74.125.206.26
Hello.zonetransfer.me	text = "Hi to Josh and all his class"
Name:	home.zonetransfer.me
Address: 127.0.0.1
Info.zonetransfer.me	text = "ZoneTransfer.me service provided by Robin Wood - robin@digi.ninja. See http://digi.ninja/projects/zonetransferme.php for more information."
internal.zonetransfer.me	nameserver = intns1.zonetransfer.me.
internal.zonetransfer.me	nameserver = intns2.zonetransfer.me.
Name:	intns1.zonetransfer.me
Address: 81.4.108.41
Name:	intns2.zonetransfer.me
Address: 167.88.42.94
Name:	office.zonetransfer.me
Address: 4.23.39.254
Name:	ipv6actnow.org.zonetransfer.me
Address: 2001:67c:2e8:11::c100:1332
Name:	owa.zonetransfer.me
Address: 207.46.197.32
robinwood.zonetransfer.me	text = "Robin Wood"
rp.zonetransfer.me	rp = robin.zonetransfer.me. robinwood.zonetransfer.me.
sip.zonetransfer.me	naptr = 2 3 "P" "E2U+sip" "!^.*$!sip:customer-service@zonetransfer.me!" .
sqli.zonetransfer.me	text = "' or 1=1 --"
sshock.zonetransfer.me	text = "() { :]}; echo ShellShocked"
staging.zonetransfer.me	canonical name = www.sydneyoperahouse.com.
Name:	alltcpportsopen.firewall.test.zonetransfer.me
Address: 127.0.0.1
testing.zonetransfer.me	canonical name = www.zonetransfer.me.
Name:	vpn.zonetransfer.me
Address: 174.36.59.154
Name:	www.zonetransfer.me
Address: 5.196.105.14
xss.zonetransfer.me	text = "'><script>alert('Boo')</script>"
zonetransfer.me
	origin = nsztm1.digi.ninja
	mail addr = robin.digi.ninja
	serial = 2019100801
	refresh = 172800
	retry = 900
	expire = 1209600
	minimum = 3600
```

Si nous parvenons à effectuer un transfert de zone réussi pour un domaine, il n'est pas nécessaire de continuer à énumérer ce domaine particulier car cela extraira toutes les informations disponibles.

## Gobuster

Gobuster est un outil que nous pouvons utiliser pour effectuer une énumération de sous-domaines. Les options de modèles sont particulièrement intéressantes pour nous, car nous avons appris certaines conventions de dénomination à partir de la collecte passive d'informations que nous pouvons utiliser pour découvrir de nouveaux sous-domaines suivant le même modèle.

Nous pouvons utiliser une liste de mots du référentiel Seclists avec gobuster si nous recherchons des mots dans des modèles au lieu de chiffres. N'oubliez pas que lors de nos activités passives d'énumération de sous-domaines, nous avons trouvé un modèle lert-api-shv-{NUMBER}-sin6.facebook.com. Nous pouvons utiliser ce modèle pour découvrir des sous-domaines supplémentaires. La première étape sera de créer un fichier patterns.txt avec les patterns précédemment découverts, par exemple :
```
lert-api-shv-{GOBUSTER}-sin6
atlas-pp-shv-{GOBUSTER}-sin6
```

La prochaine étape consistera à lancer gobuster à l'aide du module dns en précisant les options suivantes :
```
dns : Lancer le module DNS
-q : N'imprime pas la bannière et les autres bruits.
-r : utiliser un serveur DNS personnalisé
-d : Un nom de domaine cible
-p : chemin d'accès au fichier de motifs
-w : chemin d'accès à la liste de mots
-o : Fichier de sortie
Dans notre cas, ce sera la commande.
```

```
dsgsec@htb[/htb]$ export TARGET="facebook.com"
dsgsec@htb[/htb]$ export NS="d.ns.facebook.com"
dsgsec@htb[/htb]$ export WORDLIST="numbers.txt"
dsgsec@htb[/htb]$ gobuster dns -q -r "${NS}" -d "${TARGET}" -w "${WORDLIST}" -p ./patterns.txt -o "gobuster_${TARGET}.txt"

Found: lert-api-shv-01-sin6.facebook.com
Found: atlas-pp-shv-01-sin6.facebook.com
Found: atlas-pp-shv-02-sin6.facebook.com
Found: atlas-pp-shv-03-sin6.facebook.com
Found: lert-api-shv-03-sin6.facebook.com
Found: lert-api-shv-02-sin6.facebook.com
Found: lert-api-shv-04-sin6.facebook.com
Found: atlas-pp-shv-04-sin6.facebook.com
```

