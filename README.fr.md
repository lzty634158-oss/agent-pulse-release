# Agent Pulse

Agent Pulse synchronise l’état de fonctionnement de Claude Code et Codex avec un Dashboard local Windows, une fenêtre flottante et un voyant physique tricolore ESP32, afin que vous puissiez suivre la progression des sessions de programmation IA sans surveiller constamment le terminal.

Langues : [English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | Français | [Deutsch](README.de.md) | [Español](README.es.md)

## Signification des états

| Couleur | Signification courante |
|---|---|
| Vert | Session inactive, tâche terminée ou résultat prêt à consulter |
| Jaune | Réponse en cours, appel d’outil, traitement poursuivi après l’outil ou informations complémentaires nécessaires |
| Rouge | Demande d’autorisation, échec d’outil, blocage ou intervention humaine nécessaire |

Les états sont enregistrés et affichés par répertoire de projet ; plusieurs projets sur le même ordinateur peuvent apparaître simultanément dans le Dashboard.

## Installation

### Recommandé : package d’installation Windows

Téléchargez et exécutez `AgentPulseSetup-<版本>.exe`. Les utilisateurs ordinaires n’ont pas besoin d’installer eux-mêmes Node.js, npm, Python, BLE Bridge, PyInstaller ou les outils Arduino.

Pour l’utilisateur Windows actuel, le programme d’installation :

- installe le daemon Agent Pulse, le runtime Node intégré, BLE Bridge et la fenêtre flottante ;
- fusionne en toute sécurité les hooks Claude Code et Codex, sans écraser les autres hooks existants ;
- configure le démarrage automatique après connexion ;
- démarre Agent Pulse et ouvre le Dashboard une fois terminé.

L’emplacement d’installation par défaut est généralement :

```text
%LOCALAPPDATA%\AgentPulse
```

Une fois l’installation terminée, redémarrez Claude Code/Codex ou ouvrez une nouvelle session afin de recharger les hooks.

### Installation depuis les sources/la ligne de commande

Il s’agit du parcours développeur, et non d’une étape nécessaire pour les utilisateurs du package d’installation Windows. Consultez l’[annexe développeur](#annexe-développeur) à la fin du document.

## Utilisation quotidienne

### Dashboard

Ouvrir :

```text
http://127.0.0.1:7900
```

Vous pouvez également l’ouvrir depuis **Open Dashboard** dans le menu Démarrer. Le Dashboard est le point d’entrée des opérations quotidiennes et permet de consulter :

- le projet actuel, la couleur du voyant et les événements en temps réel ;
- l’état de la connexion BLE, le niveau de batterie de l’appareil (lorsqu’il est pris en charge) ;
- l’affichage/le masquage de la fenêtre flottante ;
- les mises à jour du programme et du firmware ;
- l’accès à la configuration.

Le Dashboard n’écoute que l’adresse loopback locale et n’est pas exposé au réseau local.

### Page de configuration

Cliquez sur « Configuration » dans le Dashboard pour ouvrir la page de configuration. Son adresse par défaut est :

```text
http://127.0.0.1:4321/?lang=zh
```

Vous pouvez régler les notifications, le délai de détection de blocage, les effets de couleur/clignotement/respi­ration des différents événements, la luminosité et le son. `7900` correspond au Dashboard et `4321` à la page de configuration indépendante ; leurs usages sont différents.

### Intégration de Claude Code et Codex

Le programme d’installation fusionne les hooks globaux d’Agent Pulse dans :

```text
%USERPROFILE%\.claude\settings.json
%USERPROFILE%\.codex\hooks.json
```

Il détecte les événements tels que le démarrage de session, l’envoi par l’utilisateur, avant et après les appels d’outils, les demandes d’autorisation, les arrêts et les échecs, puis met à jour le Dashboard, la fenêtre flottante et le voyant matériel. Vos autres hooks et paramètres sont conservés.

Méthode de vérification : ouvrez une nouvelle session Claude Code ou Codex, envoyez une requête et déclenchez un appel d’outil ou une demande d’autorisation, puis observez les événements en temps réel et la couleur d’état dans le Dashboard.

> Codex Offline Sandbox peut bloquer le réseau loopback local ; Agent Pulse continue à synchroniser l’état en surveillant les fichiers d’état locaux et ne dépend pas de ce canal réseau.

Si l’état ne se met pas à jour, réinstallez les hooks correspondants depuis la page de configuration, puis redémarrez Claude Code/Codex ou ouvrez une nouvelle session.

## Voyant d’état matériel

L’appareil physique actuel HW v2 / ESP32-C3-next utilise **trois LED indépendantes rouge, jaune et verte** ; il ne possède pas de LED bleue. Les états du voyant physique sont les suivants :

- **LED verte** : tâche terminée, session inactive ou résultat prêt à consulter.
- **LED jaune** : réponse en cours, appel d’outil ou traitement en cours.
- **LED rouge** : demande d’autorisation, échec d’outil, blocage ou intervention humaine nécessaire.

Les effets allumé fixe, clignotant et respiration peuvent être librement ajustés dans la page de configuration.

> L’icône BLE du Dashboard peut s’afficher en bleu ; elle indique uniquement que l’ordinateur recherche ou connecte le Bluetooth, **et ne signifie pas que l’appareil allumera une LED bleue**.

### Mise sous tension, arrêt et bouton

- **Mise sous tension** : lorsque l’appareil est éteint, maintenez le bouton enfoncé environ 2 secondes ; l’appareil maintient son alimentation et démarre.
- **Retour de mise sous tension** : l’appareil affiche successivement rouge → jaune → vert, puis entre dans l’état par défaut de clignotement vert et de diffusion BLE ; si le son est activé, il joue une tonalité de démarrage.
- **Arrêt** : après la mise sous tension, maintenez de nouveau le bouton environ 2 secondes ; l’appareil éteint les LED et désactive le maintien d’alimentation ; si le son est activé, il joue une tonalité d’arrêt.
- **Appui court** : affiche le niveau de batterie pendant environ 2 secondes ; en l’absence de connexion BLE, redémarre/réveille aussi la diffusion.
- **Pendant la mise à niveau** : les actions sur le bouton sont ignorées pendant le transfert OTA afin d’éviter toute interruption accidentelle de la mise à niveau.

### Voyant physique et retours d’identification

- **Diffusion en cours, en attente de connexion** : LED verte en respiration.
- **BLE connecté** : LED verte allumée fixe, puis restauration/réception de l’état Agent actuel.
- **Connexion interrompue** : l’appareil recommence à diffuser et revient à la respiration verte.
- **Toujours non connecté après environ 60 secondes** : arrêt de la diffusion et clignotement rouge ; un appui court permet de recommencer la diffusion.
- **Identifier l’appareil** : lors de l’exécution de l’identification depuis le Dashboard, l’appareil affiche rapidement rouge → jaune → vert → éteint, répète le cycle plusieurs fois puis revient à son état d’origine.
- **Animation de connexion** : l’appareil indique le processus de connexion dans l’ordre rouge, jaune, vert ; une fois connecté, l’hôte envoie l’état de travail actuel.

### Batterie, charge et son

Un appui court sur l’appareil affiche le niveau de batterie pendant environ 2 secondes :

| Estimation de tension | Effet lumineux |
|---|---|
| Environ ≥ 4.0 V | Rouge, jaune et vert tous allumés |
| Environ 3.7–4.0 V | Rouge et jaune allumés |
| Environ < 3.7 V | LED rouge allumée |

Lorsqu’il est connecté et que l’appareil le prend en charge, le Dashboard et la fenêtre flottante affichent la tension estimée, le pourcentage et l’état de charge. Ces valeurs servent à l’évaluation quotidienne et ne doivent pas être utilisées comme un indicateur de batterie de précision.

L’interrupteur « Son » de la page de configuration contrôle les tonalités du buzzer ; il est désactivé par défaut. Les réglages de luminosité tricolore et de son sont enregistrés dans l’appareil et persistent après coupure de l’alimentation.

### Méthodes de connexion

#### Connexion BLE (recommandée)

1. Mettez l’appareil sous tension ; s’il reste longtemps sans connexion, effectuez un appui court pour qu’il recommence à diffuser.
2. Ouvrez le Dashboard et attendez que l’état BLE passe de recherche/connexion à connecté.
3. Une fois la connexion établie, Agent Pulse synchronise automatiquement l’état actuel avec le voyant physique.

L’icône BLE du Dashboard signifie généralement : bleu pour recherche/connexion en cours, vert pour une interaction valide récente reçue de l’appareil, gris pour non connecté et rouge pour une erreur de connexion. Le bleu est uniquement l’état de l’icône logicielle, pas une LED physique.

Si l’appareil est introuvable, vérifiez qu’il est sous tension et en diffusion, que le Bluetooth Windows est disponible, qu’il est suffisamment proche, et évitez d’exécuter simultanément plusieurs instances d’Agent Pulse/BLE Bridge.

#### Connexion USB, diagnostic et récupération

L’USB peut servir au contrôle filaire du voyant, à la lecture des informations/de la batterie de l’appareil, au diagnostic, à la récupération ainsi qu’à la mise à niveau du firmware des appareils compatibles. Utilisez un câble de données plutôt qu’un câble de charge uniquement, et vérifiez qu’un port COM apparaît dans le Gestionnaire de périphériques.

La version actuelle filtre les appareils candidats selon l’identifiant du fabricant du port série USB ; si plusieurs ESP32 ou appareils série USB courants sont connectés, spécifiez explicitement le port cible dans la ligne de commande, par exemple :

```powershell
agent-traffic-light-monitor device list
agent-traffic-light-monitor device push --port COM3
```

N’utilisez pas de périphérique série non concerné comme cible du contrôle de voyant Agent Pulse. Dans une version ultérieure, une requête `deviceInfo` sera d’abord envoyée aux ports série candidats ; la connexion automatique ne sera établie qu’après réception d’une réponse d’appareil valide.

Si l’appareil ne possède aucun port série, vérifiez le câble, les pilotes et si USB CDC est activé dans le firmware. L’USB est la méthode de récupération prioritaire pour le premier flashage, la migration de partition ou après un échec OTA.

## Fenêtre flottante

Dans le Dashboard, cliquez sur « Ouvrir le voyant flottant » ou « Fermer le voyant flottant ». La fenêtre flottante affiche la couleur d’état actuelle, le nom du projet, l’état BLE et, lorsqu’il est disponible, le niveau de batterie de l’appareil.

Elle est gérée par le daemon de la version installée ; même si le démarrage de la fenêtre flottante échoue, le Dashboard et la synchronisation d’état peuvent continuer à fonctionner.

## Mise à jour du programme

Dans la zone **Mise à jour du programme AgentPulse** du Dashboard :

1. Cliquez sur « Vérifier les mises à jour du programme ».
2. Lorsqu’une nouvelle version est trouvée, cliquez sur « Confirmer l’installation ».
3. Agent Pulse télécharge et vérifie le manifeste de mise à jour signé, le nom du package d’installation, la taille et SHA-256.
4. Une fois la vérification réussie, l’Explorateur Windows s’ouvre et sélectionne le package d’installation vérifié.
5. Double-cliquez manuellement sur cet EXE et terminez l’installation dans l’assistant Inno Setup visible.

Agent Pulse n’exécute pas silencieusement le package d’installation et ne termine pas l’assistant d’installation à votre place. Lors d’une installation par remplacement, l’assistant Inno ferme le daemon Agent Pulse, la fenêtre flottante et BLE Bridge du répertoire d’installation actuel afin de libérer les fichiers utilisés ; il ne ferme pas largement des programmes non concernés.

Les packages d’installation téléchargés sont mis en cache par défaut dans :

```text
%LOCALAPPDATA%\AgentPulse\updates\desktop\
```

## Mise à jour du firmware

Capacités matérielles :

- **Firmware ESP32-C3-next pouvant être mis à niveau** : les informations de l’appareil doivent signaler l’identifiant matériel et la capacité OTA suivants pour pouvoir utiliser l’OTA BLE/USB du Dashboard :

  ```text
  agentpulse-esp32c3-next
  ```

Avant la mise à niveau, confirmez les informations de l’appareil et le firmware cible. L’OTA n’accepte que l’**image d’application** Arduino `.ino.bin` ; ne téléversez pas de bootloader, de table de partitions, de `merged.bin` ni d’autres fichiers complets de premier flashage.

### Limites importantes

L’OTA actuelle reste une fonctionnalité de laboratoire : le firmware n’implémente pas encore la vérification de signature d’image, Secure Boot, Flash Encryption, la confirmation de santé ni le rollback automatique. Pendant la mise à niveau, ne coupez pas l’alimentation, ne débranchez pas le câble, ne désactivez pas Bluetooth et ne quittez pas le daemon ; en cas d’échec, privilégiez une récupération par USB.

***Il est recommandé de brancher l’alimentation pendant la mise à niveau afin d’éviter qu’une coupure soudaine ne provoque un échec de la mise à niveau et n’affecte l’utilisation.***

La disposition des partitions OTA des anciens appareils ne peut pas être migrée par une OTA application BLE/USB ordinaire. Pour migrer la disposition des partitions ou effectuer le premier flashage, vous devez flasher intégralement le bootloader, la table de partitions, le OTA boot selector et la factory app via le mode USB download/bootloader.

## Données et confidentialité

Les données d’exécution sont enregistrées localement par défaut :

```text
%LOCALAPPDATA%\AgentPulse\
  config.json
  projects\<projectId>\status.json
  projects\<projectId>\events.jsonl
  daemon\
  updates\
```

Par défaut, Agent Pulse ne téléverse ni code, ni prompts, ni sorties de terminal, ni fichiers de projet. L’ancien `.agent-pulse` dans la racine du projet est utilisé uniquement pour la compatibilité/la migration ; les nouvelles versions n’écrivent plus de nouvelles données d’exécution dans le répertoire du projet.

## Questions fréquentes

### Impossible d’ouvrir le Dashboard

Vérifiez que vous accédez à `http://127.0.0.1:7900`, et non au port `4321` de la page de configuration. Avec la version installée, vous pouvez essayer de redémarrer Agent Pulse depuis le menu Démarrer ; les développeurs peuvent vérifier depuis la ligne de commande :

```powershell
agent-traffic-light-monitor daemon status
agent-traffic-light-monitor daemon logs
```

N’exécutez pas simultanément le daemon des sources et la version installée : ils se disputent `7900`, `47801`, `7950` et l’appareil BLE.

### L’état de Claude Code/Codex ne change pas

1. Ouvrez une nouvelle session Claude Code/Codex.
2. Réinstallez les hooks correspondants depuis la page de configuration.
3. Vérifiez que `%USERPROFILE%\.claude\settings.json` ou `%USERPROFILE%\.codex\hooks.json` contient toujours la configuration Agent Pulse.
4. Les utilisateurs de Claude Code peuvent exécuter :

   ```powershell
   agent-traffic-light-monitor doctor
   ```

### Impossible de connecter le BLE

Vérifiez l’alimentation de l’appareil, le Bluetooth Windows, la distance et l’état du Dashboard ; ne démarrez pas manuellement un BLE Bridge supplémentaire afin de ne pas occuper `47801`.

### Appareil USB introuvable

Utilisez un câble de données, consultez « Ports (COM et LPT) » dans le Gestionnaire de périphériques et, si nécessaire, sélectionnez un port COM explicite. S’il n’y a aucun port COM, vérifiez le firmware USB CDC et les pilotes.

### Notifications trop fréquentes

Désactivez les rappels de fin/erreur/blocage dans la page de configuration, ou ajustez le délai de détection de « blocage ».

## Remarques

- Ce document s’adresse aux utilisateurs du package d’installation Windows. Avant une utilisation en production, vérifiez les hooks Claude/Codex, le BLE, la fenêtre flottante et le processus de mise à jour sur l’ordinateur et le matériel cibles.
- Plusieurs appareils Agent Pulse ne doivent actuellement pas être distingués automatiquement sur la seule base du même nom BLE ; les futurs scénarios multi-appareils devront utiliser une liaison `deviceId` unique, et RSSI ne convient que comme base de tri lors de la première découverte.
- La mise à jour du programme et l’OTA du firmware sont des processus différents : la mise à jour du programme installe un EXE Windows ; l’OTA du firmware écrit uniquement l’image d’application sur les appareils compatibles.
