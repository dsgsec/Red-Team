# Fournisseurs antivirus

De nombreux fournisseurs antivirus sur le marché se concentrent principalement sur la mise en œuvre d'un produit de sécurité destiné aux utilisateurs domestiques ou professionnels. Les logiciels antivirus modernes se sont améliorés et combinent désormais des capacités antivirus avec d'autres fonctionnalités de sécurité telles que le pare-feu, le cryptage, l'anti-spam, l'EDR, l'analyse des vulnérabilités, le VPN, etc.

Il est important de noter qu'il est difficile de recommander quel logiciel AV est le meilleur. Tout dépend des préférences et de l'expérience de l'utilisateur. De nos jours, les fournisseurs antiviruss se concentrent sur la sécurité des entreprises en plus de la sécurité des utilisateurs finaux. Nous vous suggérons de consulter le[ site Web de comparaison AV ](https://www.av-comparatives.org/list-of-enterprise-av-vendors-pc/)pour plus de détails sur les fournisseurs AV d'entreprise .

## Environnement de test  antivirus

Les environnements de test AV sont un excellent endroit pour vérifier les fichiers suspects ou malveillants. Vous pouvez télécharger des fichiers pour les analyser auprès de différents fournisseurs de logiciels antiviruss. De plus, des plateformes telles que VirusTotal utilisent diverses techniques et fournissent des résultats en quelques secondes. En tant que red teamer ou pentester, nous devons tester une charge utile par rapport aux applications antivirusles les plus connues pour vérifier l'efficacité de la technique de contournement.

## VirusTotal

![368287c08435172ac9740a2a68e4e3fa](https://github.com/dsgsec/Red-Team/assets/82456829/8c316752-e021-4dda-957a-244bc3a12d6b)

VirusTotal est une plate-forme d'analyse Web bien connue pour vérifier les fichiers suspects. Il permet aux utilisateurs de télécharger des fichiers à analyser avec plus de 70 moteurs de détection antivirus. VirusTotal transmet les fichiers téléchargés aux moteurs antivirus pour qu'ils soient vérifiés, renvoie le résultat et signale s'ils sont malveillants ou non. De nombreux points de contrôle sont appliqués, notamment la vérification des URL ou des services sur liste noire, les signatures, l'analyse binaire, l'analyse comportementale, ainsi que la vérification des appels d'API. De plus, le binaire sera exécuté et vérifié dans un environnement simulé et isolé pour de meilleurs résultats. Pour plus d'informations et pour vérifier d'autres fonctionnalités, vous pouvez visiter le site Web [de VirusTotal](https://www.virustotal.com/) .

## Alternatives à VirusTotal

Remarque importante :  VirusTotal est une plate-forme d'analyse pratique dotée d'excellentes fonctionnalités, mais elle dispose d'une politique de partage. Tous les résultats analysés seront transmis et partagés avec les fournisseurs d'antivirus afin d'améliorer leurs produits et de mettre à jour leurs bases de données pour détecter les logiciels malveillants connus. En tant que membre de l'équipe rouge, cela brûlera un compte-gouttes ou une charge utile que vous utilisez lors des engagements. Ainsi, des solutions alternatives sont disponibles pour tester auprès de différents fournisseurs de produits de sécurité, et l'avantage le plus important est qu'elles n'ont pas de politique de partage. Cependant, il existe d'autres limites. Vous disposerez d'un nombre limité de fichiers à analyser par jour ; sinon, un  abonnement  est nécessaire  pour des tests illimités. Pour ces raisons, nous vous recommandons de tester vos malwares uniquement sur des sites qui ne partagent pas d'informations, tels que :

-   [AntiscanMe](https://antiscan.me/)  (6 analyses gratuites par jour)
-   [Analyse antivirus L'analyse des logiciels malveillants de Jotti](https://virusscan.jotti.org/)

## Logiciel AV de prise d'empreintes digitales

En tant qu'équipe rouge, nous ne savons pas quel logiciel antivirus est en place une fois que nous obtenons un premier accès à une machine cible. Par conséquent, il est important de rechercher et d'identifier les produits de sécurité basés sur l'hôte installés, y compris les logiciels antivirus. L'empreinte digitale AV est un processus essentiel pour déterminer quel fournisseur AV est présent. Savoir quel logiciel AV est installé est également très utile pour créer le même environnement pour tester les techniques de contournement.

Cette section présente différentes manières d'examiner et d'identifier les logiciels antivirus en fonction d'artefacts statiques, notamment les noms de services, les noms de processus, les noms de domaine, les clés de registre et les systèmes de fichiers.

Le tableau suivant contient des logiciels antiviruss bien connus et couramment utilisés . 

| Nom de l'antivirus  | Nom du service                    | Nom du processus                   |
| ------------------- | --------------------------------- | ---------------------------------- |
| Microsoft Défenseur | WinDefend                         | MSMpEng.exe                        |
| Tendance Micro      | TMBMSRV                           | TMBMSRV.exe<br>                    |
| Avira               | AntivirService, Avira.ServiceHost | avguard.exe, Avira.ServiceHost.exe |
| Bitdefender         | VSSERV                            | bdagent.exe, vsserv.exe            |
| Kaspersky           | AVP<Version #>                    | avp.exe, ksde.exe<br>              |
| MOYENNE             | AVG Antivirus                     | AVGSvc.exe                         |
| Norton              | Sécurité Norton                   | NortonSecurity.exe                 |
| McAfee              | McAPExe, Mfemms                   | MCAPExe.exe, mfemms.exe            |
| Panda               | PavPrSvr                          | PavPrSvr.exe                       |
| Avast               | Antivirus Avast                   | afwServ.exe, AvastSvc.exe<br><br>  |

## SharpEDRChecker

Une façon de prendre des empreintes AV consiste à utiliser des outils publics tels que [ SharpEDRChecker ](https://github.com/PwnDexter/SharpEDRChecker). Il est écrit en C# et effectue diverses vérifications sur une machine cible, notamment des vérifications des logiciels antivirus , tels que les processus en cours d'exécution, les métadonnées des fichiers, les fichiers DLL chargés, les clés de registre, les services, les répertoires et les fichiers.

Nous avons pré-téléchargé SharpEDRChecker depuis le [dépôt GitHub ](https://github.com/PwnDexter/SharpEDRChecker) afin de pouvoir l'utiliser dans la VM ci-jointe . Nous devons maintenant compiler le projet et nous avons déjà créé un raccourci vers le projet sur le bureau (SharpEDRChecker). Pour ce faire, double-cliquez dessus pour l'ouvrir dans Microsoft Visual Studio 2022. Maintenant que notre projet est prêt, nous devons le compiler, comme le montre la capture d'écran suivante :

![9f2dc2083d2a8227b688cef623ac9bf8](https://github.com/dsgsec/Red-Team/assets/82456829/b7c4a397-74a6-40e5-a1f4-07f3474beec2)


Une fois compilé, nous pouvons trouver le chemin de la version compilée dans la section de sortie, comme souligné à l'étape 3. Nous avons également ajouté une copie de la version compilée dans le  répertoire C:\Users\thm\Desktop\Files . Essayons maintenant de l'exécuter et voyons le résultat comme suit :

|

Invite de commande !

```
C:\> SharpEDRChecker.exe

```

 | ➜ |

Résumé de SharpEDRChecker

```
[!] Directory Summary:
   [-] C:\Program Files\Windows Defender : defender
   [-] C:\Program Files\Windows Defender Advanced Threat Protection : defender, threat
   [-] C:\Program Files (x86)\Windows Defender : defender

[!] Service Summary:
   [-] PsShutdownSvc : sysinternal
   [-] Sense : defender, threat
   [-] WdNisSvc : defender, nissrv
   [-] WinDefend : antimalware, defender, malware, msmpeng
   [-] wscsvc : antivirus
```

 |

En conséquence, Windows Defender est trouvé en fonction des dossiers et des services. Notez que ce programme peut être signalé par un logiciel antivirus comme malveillant car il effectue diverses vérifications et appels d'API.

## Vérifications d'empreintes digitales C#

Une autre façon d'énumérer les logiciels antiviruss consiste à coder notre propre programme.  Nous avons préparé un programme C# dans la machine virtuelle Windows 10 Pro fournie , afin que nous puissions faire quelques expériences pratiques ! Vous pouvez trouver l'icône du projet sur le bureau (AV-Check) et double-cliquer dessus pour l'ouvrir à l'aide de Microsoft Visual Studio 2022. 

Le code C# suivant est simple et son objectif principal est de déterminer si le logiciel AV est installé sur la base d'une liste prédéfinie d'applications AV bien connues.

```
using System;
using System.Management;

internal class Program
{
    static void Main(string[] args)
    {
        var status = false;
        Console.WriteLine("[+] Antivirus check is running .. ");
        string[] AV_Check = {
            "MsMpEng.exe", "AdAwareService.exe", "afwServ.exe", "avguard.exe", "AVGSvc.exe",
            "bdagent.exe", "BullGuardCore.exe", "ekrn.exe", "fshoster32.exe", "GDScan.exe",
            "avp.exe", "K7CrvSvc.exe", "McAPExe.exe", "NortonSecurity.exe", "PavFnSvr.exe",
            "SavService.exe", "EnterpriseService.exe", "WRSA.exe", "ZAPrivacyService.exe"
        };
        var searcher = new ManagementObjectSearcher("select * from win32_process");
        var processList = searcher.Get();
        int i = 0;
        foreach (var process in processList)
        {
            int _index = Array.IndexOf(AV_Check, process["Name"].ToString());
            if (_index > -1)
            {
                Console.WriteLine("--AV Found: {0}", process["Name"].ToString());
                status = true;
            }
            i++;
        }
        if (!status) { Console.WriteLine("--AV software is not found!");  }
    }
}
```

Expliquons un peu plus le code. Nous avons prédéfini une liste d'applications antivirusles bien connues dans le  tableau AV_Check dans notre code, qui est tirée de la section précédente, où nous avons discuté du logiciel antivirus de prise d'empreintes digitales (tableau ci-dessus). Ensuite, nous utilisons la requête WMIC (Windows Management Instrumentation Command-Line) ( select * from win32_process ) pour répertorier tous les processus en cours d'exécution sur la machine cible et les stocker dans la   variable processList . Ensuite, nous passons en revue les processus en cours d'exécution et comparons s'ils existent dans le tableau prédéfini  . Si une correspondance est trouvée, alors nous avons installé un logiciel AV . 

Le programme C# utilise un objet WMIC pour répertorier les processus en cours d'exécution, qui peuvent être surveillés par un logiciel AV . Si le logiciel AV est mal implémenté pour surveiller les requêtes WMIC ou les API Windows, cela peut entraîner des résultats faussement positifs lors de l'analyse de notre programme C#.

Compilons une version x86 du programme C#, téléchargeons-la sur le site Web de VirusTotal et vérifions les résultats ! Pour compiler le programme C# dans Microsoft Visual Studio 2022, sélectionnez Créer dans le menu de la barre et choisissez l' option Créer une solution . Ensuite, si tout s'est bien passé, vous pouvez trouver le chemin de la version compilée dans la section de sortie, comme souligné à l'étape 3 dans la capture d'écran ci-dessous.

![4abf4090797989e9457af9a2cb18a35b](https://github.com/dsgsec/Red-Team/assets/82456829/ae020b75-64a7-4355-82d8-6737e88f09fe)

Si nous téléchargeons le programme AV-Check sur le  [site Web de VirusTotal ](https://www.virustotal.com/gui/home/upload) et vérifions le résultat, étonnamment, VirusTotal a montré que deux fournisseurs d'antivirus (MaxSecure et SecureAge APEX) ont signalé notre programme comme malveillant ! Il s'agit donc d'un résultat faussement positif qui identifie de manière incorrecte un fichier comme malveillant alors qu'il ne l'est pas. L'une des raisons possibles est que les logiciels de ces fournisseurs antiviruss utilisent un classificateur d'apprentissage automatique ou une méthode de détection basée sur des règles qui est mal implémentée. Pour plus de détails sur le rapport de soumission proprement dit, voir [ ici ](https://www.virustotal.com/gui/file/5f7d3e6cf58596a0186d89c20004c76805769f9ef93dc39e346e7331eee9e7ff?nocache=1). Il y a quatre sections principales : Détection, Détails, Comportement et Communauté. Si nous vérifions la section Comportement, nous pouvons voir tous les appels des API Windows, des clés de registre, des modules et de la requête WMIC.

![87289129012d58c2d77c1791131333c3](https://github.com/dsgsec/Red-Team/assets/82456829/a0bc6c43-5833-4402-9af7-b96306c96100)

Dans la section Détection, il existe des règles Sigma qui, si un événement système lors de l'exécution correspond (dans l'environnement sandbox), considèrent le fichier comme malveillant. Ce résultat est probablement basé sur les règles ; VirusTotal a signalé notre programme en raison de la  [technique Process Ghosting ](https://pentestlaboratories.com/2021/12/08/process-ghosting/), comme le montre la capture d'écran suivante.

![c4de9def10a394bc77b0b05580cae8e5](https://github.com/dsgsec/Red-Team/assets/82456829/bbe3f026-6b1a-4a21-b9d0-8725eab1f518)

Recompilons maintenant le programme C# à l'aide d'un processeur x64 et vérifions si les moteurs de contrôle agissent différemment. Cette fois, lors de notre tentative de soumission, les logiciels de trois fournisseurs d'antivirus (Cyren AV est ajouté à la liste) ont signalé le fichier comme malveillant. Pour plus de détails sur le rapport de soumission proprement dit, regardez  [ici ](https://www.virustotal.com/gui/file/b092173827888ed62adea0c2bf4f451175898d158608cf7090e668952314e308?nocache=1). 

![6992f821933117c890b129f63cbbcebf](https://github.com/dsgsec/Red-Team/assets/82456829/1b55f836-28c6-4b86-8c60-79337e9e51ca)

Remarque :  si vous essayez de soumettre un fichier au site Web VirusTotal, le résultat peut être différent. Gardez à l'esprit que VirusTotal partage des rapports de soumission avec les fournisseurs d'antivirus pour améliorer leurs moteurs de détection antivirus , y compris les résultats faussement positifs.
