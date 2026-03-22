# Learning-Network-Oriented-Cybersecurity

Ce dossier GitHub me sert juste à garder des notes sur ce que j'apprends du réseau. La majorité seront plutôt orientée cybersécurité.


# Chapitre 1 : Les fondations physiques du réseau (Couche 1 OSI)

## 1. La transmission par le cuivre
- **Le concept :** L'information binaire (0 et 1) est encodée par des variations de tension électrique (le déplacement d'électrons) dans des câbles.
- **La vulnérabilité :** Selon la loi d'Ampère, le passage du courant génère un champ magnétique. Un attaquant peut utiliser une bobine d'induction pour capter ce champ à proximité du câble et reconstituer les données sans couper le fil (*wiretapping*).

## 2. Les défenses physiques (Câbles Ethernet)
- **Le concept :** Les câbles modernes utilisent des paires de fils **torsadés** (souvent blindés). La carte réseau envoie un signal positif sur un fil et son exact opposé (négatif) sur l'autre (transmission en mode différentiel).
- **L'avantage sécurité :** Les courants inverses créent des champs magnétiques opposés qui s'annulent mutuellement. Cela assure la **confidentialité** (empêche le signal de fuir hors du câble) et l'**intégrité** (les perturbations externes s'annulent lors de la lecture à l'arrivée).

## 3. Le matériel de Couche 1 : Le Hub (Concentrateur)
- **Le concept :** Équipement historique permettant de relier plusieurs câbles. Il se contente de régénérer le signal électrique.
- **La vulnérabilité :** Il diffuse le signal entrant sur **tous** ses ports. N'importe quelle machine connectée reçoit le trafic destiné aux autres, rendant l'espionnage local (sniffing en mode promiscuité) trivial.


# Chapitre 2 : L'aiguillage local (Couche 2 OSI - Liaison de données)

## 1. L'identification matérielle : L'adresse MAC
- **Le concept :** Pour éviter que tout le monde reçoive tout (comme avec un Hub), chaque carte réseau possède un identifiant physique unique gravé en usine : l'adresse MAC. 

## 2. Le matériel de Couche 2 : Le Switch (Commutateur)
- **Le concept :** Le Switch remplace le Hub. Il maintient une table en mémoire (Table MAC/CAM) qui associe chaque adresse MAC au port physique sur lequel la machine est branchée. 
- **L'avantage sécurité :** Lorsqu'il reçoit des données, le Switch les envoie **uniquement** sur le câble du destinataire. Cela empêche l'écoute passive (sniffing simple) sur le réseau local.

## 3. La vulnérabilité : Le MAC Flooding
- **L'attaque :** La mémoire du Switch étant limitée, un attaquant peut l'inonder de milliers de fausses adresses MAC générées aléatoirement.
- **La conséquence :** Une fois la table saturée, le Switch ne peut plus mémoriser les vraies adresses. S'il reçoit une donnée pour une adresse inconnue, il la diffuse sur tous ses ports (mode "fail-open"). Le Switch se comporte alors comme un Hub, permettant à nouveau au pirate d'espionner le réseau.
- **La défense :** Activer le *Port Security* sur le Switch pour limiter le nombre d'adresses MAC par port.

# Chapitre 3 : Le routage mondial (Couche 3 OSI - Réseau)

## 1. L'identification logique : L'adresse IP
- **Le concept :** Pour communiquer en dehors du réseau local (Internet), on utilise une adresse IP. Elle est hiérarchique (comme une adresse postale), ce qui permet aux routeurs de déterminer le chemin optimal pour acheminer les paquets de données à l'échelle mondiale.

## 2. Le matériel de Couche 3 : Le Routeur
- **Le concept :** Équipement qui lit l'adresse IP de destination des paquets et les transfère d'un réseau à un autre (ex: de votre réseau domestique vers le réseau de votre fournisseur d'accès).

## 3. La vulnérabilité : L'IP Spoofing (Usurpation d'IP)
- **L'attaque :** Le protocole IP de base ne vérifie pas l'authenticité de l'expéditeur. Un attaquant peut modifier l'en-tête de son paquet de données pour y inscrire une fausse adresse IP source (celle d'une machine de confiance ou une adresse aléatoire).
- **La conséquence :** Permet de contourner certains pare-feu basés sur l'IP, de cacher l'origine d'une attaque, ou de mener des attaques DDoS par réflexion.
- **La défense :** Implémentation de filtres anti-spoofing sur les routeurs pour bloquer les paquets dont l'adresse source est illogique par rapport à leur point d'entrée physique.


# Chapitre 4 : La livraison applicative (Couche 4 OSI - Transport)

## 1. L'identification logicielle : Le Port
- **Le concept :** Une fois les paquets arrivés sur la bonne machine (via l'adresse IP), le système d'exploitation utilise un numéro de port (de 0 à 65535) pour savoir à quelle application précise (serveur web, messagerie, etc.) livrer les données.

## 2. La vulnérabilité : Le scan de ports furtif
- **Le concept :** Le protocole TCP nécessite un "3-way handshake" (SYN -> SYN/ACK -> ACK) pour établir une connexion fiable. Une connexion complète est généralement journalisée par l'application.
- **L'attaque (Le SYN Scan) :** L'attaquant envoie une demande de synchronisation (`SYN`). Si le port est ouvert, le serveur répond `SYN/ACK`. L'attaquant s'arrête alors volontairement avant d'envoyer le `ACK` final. Il découvre que le port est ouvert tout en contournant la journalisation, car la connexion applicative n'a jamais abouti.

## 3. L'effacement des traces : Le drapeau RST
- **Le problème :** Ne pas finaliser une connexion (comme dans un SYN Scan) laisse des processus en attente sur le serveur cible, consommant sa mémoire (RAM). Répété des milliers de fois, cela crée une attaque par déni de service (SYN Flood).
- **La technique :** Pour rester furtif et maintenir la cible en vie, l'attaquant envoie un paquet avec le drapeau TCP `RST` (Reset) juste après avoir reçu le `SYN/ACK`. Cela ordonne au système d'exploitation adverse d'avorter immédiatement la connexion.


# Chapitre 5 : L'annuaire d'Internet (Couche 7 OSI - Application)

## 1. Le concept : Le protocole DNS
- **Le rôle :** Les routeurs (Couche 3) communiquent via des adresses IP, mais les humains utilisent des noms de domaine (ex: `github.com`). Le DNS (Domain Name System) est l'annuaire qui traduit les noms en adresses IP.
- **La mécanique :** Pour privilégier la rapidité, le DNS standard s'appuie sur le protocole UDP (Couche 4), qui envoie des données sans processus de vérification ou d'établissement de session préalable.

## 2. La vulnérabilité : Le DNS Spoofing (Usurpation DNS)
- **L'attaque :** Un attaquant sur le même réseau local écoute les requêtes DNS (qui circulent en clair). Lorsqu'une cible demande l'adresse IP d'un site légitime, l'attaquant forge immédiatement une fausse réponse contenant sa propre adresse IP.
- **La condition de succès :** C'est une course de vitesse (*race condition*). La fausse réponse UDP de l'attaquant doit atteindre la machine cible **avant** la réponse légitime.
- **La conséquence :** La machine cible enregistre la fausse adresse IP. Le navigateur de l'utilisateur l'emmène sur le serveur de l'attaquant (souvent une copie parfaite du vrai site pour voler des identifiants).

## 3. L'authentification Web : Le JWT (JSON Web Token)
- **Le concept :** Le protocole web (HTTP) étant amnésique, le serveur confie au navigateur un jeton (JWT) après une connexion réussie. Le navigateur le renvoie à chaque clic pour prouver son identité. Il contient un Header (algorithme), un Payload (données utilisateur) et une Signature.
- **La règle d'or :** Le contenu d'un JWT (Header et Payload) n'est pas chiffré, il est juste encodé en Base64. Il est lisible par tous. Sa sécurité repose sur son **intégrité** garantie par la signature.
- **La vulnérabilité (Attaque "None Algorithm") :** Faille logique (CVE-2015-9256) où un attaquant modifie le Header pour définir l'algorithme sur `"none"`, et s'octroie des privilèges d'administrateur dans le Payload. Les serveurs mal codés contournaient l'étape de vérification de la signature et acceptaient le faux jeton.
- **La défense :** Configurer les bibliothèques backend pour rejeter systématiquement les jetons utilisant l'algorithme `"none"` et imposer une liste stricte d'algorithmes autorisés.