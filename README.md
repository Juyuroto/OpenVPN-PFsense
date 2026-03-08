# Guide Complet : Configuration VPN Home Lab (pfSense)

Ce guide détaille la mise en place d'un accès distant sécurisé vers une infrastructure Home Lab segmentée par VLAN, utilisant **pfSense** comme passerelle VPN et **Fedora** comme client distant.

---

## Étape 1 : Configuration NAT sur la Box Opérateur
L'objectif est de diriger tout le trafic VPN arrivant sur votre IP publique vers votre routeur pfSense.

1. Accédez à l'interface de gestion de votre box (ex: `192.168.1.254`).
2. Cherchez la section **NAT / PAT** ou **Redirection de ports**.
3. Créez une nouvelle règle :
    * **Protocole :** `UDP`
    * **Port externe/interne :** (ex: `1120`)
    * **Équipement cible :** IP de votre interface WAN pfSense.

### NAT & PAT de la Box
![NAT & PAT de la Box](screens/1.png)

---

## Étape 2 : Ouverture du Pare-feu pfSense (WAN)
Assurez-vous que pfSense accepte les connexions entrantes sur son interface WAN.

1. Allez dans **Firewall > Rules > WAN**.
2. Vérifiez la présence de la règle (souvent créée par le Wizard OpenVPN) :
    * **Protocol :** `UDP`
    * **Port :** (ex: `1120`)
    * **Source :** `*`

### Firewall > Rules > WAN
![Règles firewal WAN](screens/2.png)

---

## Étape 3 : Autorisation des flux internes (VLANs)
Pour que vos utilisateurs VPN puissent accéder à vos VLANs, il faut autoriser le trafic provenant du tunnel dans le pare-feu pfSense.

1. Allez dans **Firewall > Rules > OpenVPN**.
2. Assurez-vous que des règles "Pass" autorisent le trafic vers vos réseaux cibles.

### Firewall > Rules > OpenVPN
![Règles firewal VPN](screens/3.png)

---

## Étape 4 : Exportation de la configuration client
1. Allez dans **VPN > OpenVPN > Client Export**.
2. Dans **Host Name Resolution**, sélectionnez "Other" et saisissez impérativement votre **IP publique**.
3. Téléchargez le fichier `.ovpn` (format "Inline Config").

---

## Étape 5 : Configuration sur Fedora (Client)
L'utilisation de *NetworkManager* permet une intégration native.

1. Allez dans **Paramètres > Réseau > VPN > Ajouter (+)**.
2. Cliquez sur **Importer depuis un fichier...** et sélectionnez votre `.ovpn`.
3. Dans l'onglet **IPv4**, vérifiez :
    * **Passerelle (Gateway) :** Votre IP publique (doit être différente de l'IP locale du pfSense).
    * **IP Locale :** Laisser vide (attribuée dynamiquement par le tunnel).

### configuration VPN sur Fedora

Faite **import from file..** et vous allez prendre votre fichier **.ovpn**

![Importer le fichier .ovpn](screens/4.png)

#### Après importation

Lorsque c'est fait vous allez changer les `*` par votre **adresse ip public** (ex: 176.143.189.70)

Dans **User name** vous rentrer le username rentré lors de la création de l'utilisateur vpn

Et dans **Password** vous rentrez le mot de passe que vous avez créé lors de la création du compte vpn

Dans **Name** vous devez remplacé les `*` par le ports que vous avez choisi

Dans **Gateway** vous devez remplacé au début les `*` par votre adresse ip plublic (ex: 176.143.189.70) et après le `:` vous devez remplacer les `*` par le port que vous avez choisi

![Chhangement ip + mettre username et mdps](screens/5.png)


#### Fini

Une fois que c'est fini, vous allez voir ceci:

![Rendu](screens/6.png)


---

## Étape 6 : Validation du test de connexion
Pour valider, testez impérativement depuis l'**extérieur** (partage de connexion 4G/5G).

1. Activez le VPN. L'icône dans la barre des tâches doit devenir active.
2. Tentez de pinger une ressource locale : `ping 192.168.10.1`.

### Ping réussi
![Ping](screens/7.png)

---

### Aide au Diagnostic
* **Logs pfSense :** Consultez `Status > System Logs > OpenVPN`.
* **Logs Client (Fedora) :** Utilisez la commande `journalctl -u NetworkManager -f` dans un terminal pour voir le détail de l'échec de la poignée de main TLS (souvent dû à une mauvaise IP publique dans la Gateway).