Scenario
========

* * * * *

We are Penetration Testers working for `CAT-5 Security`. After a few successful engagements shadowing with the team, the more senior members want to see how well we can do starting an assessment on our own. The team lead sent us the following email detailing what we need to accomplish.

#### Tasking Email

![image](https://academy.hackthebox.com/storage/modules/143/scenario-email.png)

This module will allow us to practice our skills (both prior and newly minted) with these tasks. The final assessment for this module is the execution of `two` internal penetration tests against the company Inlanefreight. During these assessments, we will work through an internal penetration test simulating starting from an external breach position and a second one beginning with an attack box inside the internal network as clients often request. Completing the skills assessments signifies the successful completion of the tasks mentioned in the scoping document and tasking email above. In doing so, we will demonstrate a firm grasp of many automated and manual AD attack and enumeration concepts, knowledge of and experience with a wide array of tools, and the ability to interpret data gathered from an AD environment to make critical decisions to advance the assessment. The content in this module is meant to cover core enumeration concepts necessary for anyone to be successful in performing internal penetration tests in Active Directory environments. We will also cover many of the most common attack techniques in great depth while working through some more advanced concepts as a primer for AD-focused material that will be covered in more advanced modules.

Below you will find a completed scoping document for the engagement containing all pertinent information provided by the customer.

* * * * *

Assessment Scope
----------------

The following `IPs`, `hosts`, and `domains` defined below make up the scope of the assessment.

#### In Scope For Assessment

| Range/Domain | Description |
| --- | --- |
| `INLANEFREIGHT.LOCAL` | Customer domain to include AD and web services. |
| `LOGISTICS.INLANEFREIGHT.LOCAL` | Customer subdomain |
| `FREIGHTLOGISTICS.LOCAL` | Subsidiary company owned by Inlanefreight. External forest trust with INLANEFREIGHT.LOCAL |
| `172.16.5.0/23` | In-scope internal subnet. |
|  |  |

#### Out Of Scope

-   `Any other subdomains of INLANEFREIGHT.LOCAL`
-   `Any subdomains of FREIGHTLOGISTICS.LOCAL`
-   `Any phishing or social engineering attacks`
-   `Any other IPS/domains/subdomains not explicitly mentioned`
-   `Any types of attacks against the real-world inlanefreight.com website outside of passive enumeration shown in this module`

* * * * *

Methods Used
------------

The following methods are authorized for assessing Inlanefreight and its systems :

### External Information Gathering (Passive Checks)

External information gathering is authorized to demonstrate the risks associated with information that can be gathered about the company from the internet. To simulate a real-world attack, CAT-5 and its assessors will conduct external information gathering from an anonymous perspective on the internet with no information provided in advance regarding Inlanefreight outside of what is provided within this document.

Cat-5 will perform passive enumeration to uncover information that may help with internal testing. Testing will employ various degrees of information gathering from open-source resources to identify publicly accessible data that may pose a risk to Inlanefreight and assist with the internal penetration test. No active enumeration, port scans, or attacks will be performed against internet-facing "real-world" IP addresses or the website located at `https://www.inlanefreight.com`.

### Internal Testing

The internal assessment portion is designed to demonstrate the risks associated with vulnerabilities on internal hosts and services ( `Active Directory specifically`) by attempting to emulate attack vectors from within Inlanefreight's area of operations. The result will allow Inlanefreight to assess the risks of internal vulnerabilities and the potential impact of a successfully exploited vulnerability.

To simulate a real-world attack, Cat-5 will conduct the assessment from an untrusted insider perspective with no advance information outside of what's provided in this documentation and discovered from external testing. Testing will start from an anonymous position on the internal network with the goal of obtaining domain user credentials, enumerating the internal domain, gaining a foothold, and moving laterally and vertically to achieve compromise of all in-scope internal domains. Computer systems and network operations will not be intentionally interrupted during the test.

### Password Testing

Password files captured from Inlanefreight devices, or provided by the organization, may be loaded onto offline workstations for decryption and utilized to gain further access and accomplish the assessment goals. At no time will a captured password file or the decrypted passwords be revealed to persons not officially participating in the assessment. All data will be stored securely on Cat-5 owned and approved systems and retained for a period of time defined in the official contract between Cat-5 and Inlanefreight.

* * * * *

We provided the above scoping documentation so we become used to seeing this style of documentation. As we progress through our Infosec Careers, especially on the offensive side, it will be common to receive scoping documents and Rules of Engagement (RoE) documents that outline these types of information.

* * * * *

The Stage Is Set
----------------

Now that we have our scope clearly defined for this module, we can dive into exploring Active Directory enumeration and attack vectors. Now, let's dive into performing passive external enumeration against Inlanefreight.d atteindre les objectifs de l'évaluation. À aucun moment, un fichier de mots de passe capturés ou les mots de passe décryptés ne seront révélés à des personnes ne participant pas officiellement à l'évaluation. Toutes les données seront stockées en toute sécurité sur des systèmes détenus et approuvés par Cat-5 et conservées pendant une période définie dans le contrat officiel entre Cat-5 et Inlanefreight.

* * * * *

Nous avons fourni la documentation de portée ci-dessus afin que nous nous habituions à voir ce style de documentation. Au fur et à mesure que nous progressons dans nos carrières Infosec, en particulier du côté offensif, il sera courant de recevoir des documents d'orientation et des documents de règles d'engagement (RoE) qui décrivent ces types d'informations.

* * * * *

Le décor est planté
----------------

Maintenant que notre portée est clairement définie pour ce module, nous pouvons plonger dans l'exploration de l'énumération Active Directory et des vecteurs d'attaque. Maintenant, plongeons dans l'exécution d'une énumération externe passive contre Inlanefreight.