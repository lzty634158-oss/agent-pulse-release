# Agent Pulse

Agent Pulse synchronise l’état de fonctionnement de Claude Code et Codex avec un Dashboard local Windows, une fenêtre flottante et un voyant physique tricolore ESP32, afin que vous puissiez suivre la progression des sessions de programmation IA sans surveiller constamment le terminal.

Langues : [English](README.md) | [简体中文](README.zh-CN.md) | [繁體中文](README.zh-TW.md) | [日本語](README.ja.md) | [한국어](README.ko.md) | Français | [Deutsch](README.de.md) | [Español](README.es.md)

## Version actuelle

La version actuelle du code source est `0.4.1` (2026-08-14) et le firmware ESP32 reste en `0.1.21+22`. Un seul voyant conserve le fonctionnement simple et suit la tâche la plus récente. Avec plusieurs voyants, chacun peut suivre indépendamment la dernière tâche, un projet ou un agent tel que Claude Code, Codex, WorkBuddy ou CodeBuddy, et utiliser son propre profil lumineux.

Cette version garde aussi visibles tous les appareils BLE et USB connectés avec leur état de connexion et de batterie, empêche les processus BLE Bridge en double et indique dans la fenêtre flottante l’agent et le projet de la tâche la plus récente. Bluetooth conserve la connexion de proximité et la connexion appairée sous Windows. Les mises à jour utilisent Gitee en priorité, puis GitHub automatiquement en cas d’échec. Consultez le [journal des modifications](CHANGELOG.md) pour les détails.

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

Liens de téléchargement officiels :

- [GitHub Releases](https://github.com/lzty634158-oss/agent-pulse-release/releases)
- [Gitee Releases](https://gitee.com/lzty634158/agent-pulse-release/releases)

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

#### Confiance et configuration des hooks Codex

Codex doit autoriser l’exécution de hooks de commande externes pour qu’Agent Pulse puisse recevoir les événements Codex. Lors de la première installation ou lorsque Codex affiche une confirmation de sécurité pour les hooks, choisissez **de faire confiance / d’autoriser les hooks Agent Pulse** ; si vous les refusez ou ne leur accordez pas votre confiance, Codex n’exécutera pas ces commandes, et le Dashboard ainsi que le voyant physique ne changeront pas avec l’état de Codex.

Étapes de configuration :

1. Ouvrez le Dashboard Agent Pulse et cliquez sur « Configuration ».
2. Sur la page de configuration, cliquez sur « Installer les hooks Codex ».
3. Vérifiez que `%USERPROFILE%\.codex\hooks.json` contient les hooks Agent Pulse ; l’installation conserve les autres hooks existants.
4. Redémarrez Codex ou ouvrez une nouvelle session.
5. Lorsque Codex affiche une confirmation de confiance/exécution des hooks, choisissez de faire confiance ou d’autoriser.
6. Envoyez une requête et déclenchez un appel d’outil afin de vérifier que les événements en temps réel du Dashboard affichent l’état Codex.

Si l’état ne se met pas à jour, vérifiez d’abord l’état de confiance des hooks Codex, puis réinstallez les hooks concernés depuis la page de configuration et redémarrez Codex ou ouvrez une nouvelle session. Des installations répétées n’accumulent pas les hooks Agent Pulse ; si vous avez utilisé une ancienne version et constatez des ralentissements importants, réinstallez une fois les hooks afin d’effectuer la migration de nettoyage.

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

#### Connexion BLE de proximité (recommandée)

1. Débranchez le câble de données USB du voyant et mettez-le sous tension. Si la diffusion est arrêtée, effectuez un appui court pour la relancer.
2. Ouvrez le Dashboard. Sans appareil associé, Agent Pulse analyse en continu jusqu’à l’association automatique, l’association manuelle ou l’arrêt de l’analyse par l’utilisateur.
3. Placez le voyant cible près de l’adaptateur Bluetooth de cet ordinateur. La liste affiche en temps réel le nom, l’adresse MAC/l’identifiant, le RSSI et le nombre d’échantillons.
4. L’association automatique exige au moins 3 échantillons, un RSSI de `-45 dBm` ou plus fort et une avance d’au moins `8 dB` sur le deuxième candidat. Si les différences de réception empêchent cette détection, cliquez directement sur « Associer » pour l’appareil voulu.

Après l’association, l’analyse s’arrête et l’identifiant est enregistré dans la configuration locale. Les démarrages suivants se connectent uniquement à ce voyant et ne changent pas d’appareil selon la puissance du signal. Pour le remplacer, cliquez sur « Oublier l’appareil », puis recommencez l’association de proximité ou manuelle.

L’icône BLE est bleue pendant l’analyse/connexion, verte après une interaction valide récente, grise hors connexion et rouge en cas d’erreur. Après connexion, seul l’état actuel valide est synchronisé ; les anciens événements lumineux expirés ne sont pas rejoués. Windows affiche normalement l’adresse MAC Bluetooth. macOS peut afficher un UUID attribué par CoreBluetooth pour des raisons de confidentialité ; il permet une association locale stable, mais ce n’est pas l’adresse MAC matérielle.

Si aucun appareil n’apparaît, vérifiez l’alimentation et la diffusion, le Bluetooth du système, l’absence de connexion USB et qu’aucune autre instance d’Agent Pulse/BLE Bridge ne fonctionne. Sur macOS, autorisez aussi l’accès Bluetooth lors de la première utilisation.

#### Connexion série USB, sélection et récupération

1. Utilisez un câble USB prenant en charge les données. Un câble de charge uniquement ne crée pas de port série.
2. Ouvrez le panneau « Port série USB » du Dashboard. Windows utilise `COM*` ; macOS utilise généralement `/dev/cu.usbmodem*` ou `/dev/cu.usbserial*`.
3. La sélection automatique par défaut ouvre uniquement le périphérique AgentPulse sans pilote `VID:PID 303A:1001`. Elle n’ouvre pas automatiquement les adaptateurs CH340, CP210x, FTDI ou autres.
4. Si plusieurs périphériques `303A:1001` sont connectés, ou si un autre adaptateur compatible est nécessaire, vérifiez le port et le VID/PID dans la liste puis sélectionnez-le manuellement.

La sélection manuelle est enregistrée localement. Si le port choisi disparaît, Agent Pulse reste hors ligne au lieu d’ouvrir silencieusement un autre port. Vous pouvez revenir à la sélection automatique. La liste indique les états par défaut, sélectionné, connecté et absent ; la batterie et la charge apparaissent aussi dans l’en-tête lorsque le firmware les prend en charge.

L’USB est prioritaire sur le BLE : une connexion USB suspend l’analyse et la connexion BLE ; leur fonctionnement reprend après la déconnexion USB. Si aucun port n’apparaît, vérifiez le câble, les pilotes et le firmware USB CDC. L’USB reste la méthode de récupération recommandée pour le premier flashage, la migration de partition ou après un échec OTA.

## Fenêtre flottante

Dans le Dashboard, cliquez sur « Ouvrir le voyant flottant » ou « Fermer le voyant flottant ». La fenêtre flottante affiche la couleur d’état actuelle, le nom du projet, l’état BLE et, lorsqu’il est disponible, le niveau de batterie de l’appareil.

![Fenêtre flottante du bureau (lumière jaune = en cours)](docs/screenshots/floating-window.png)

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
- Les environnements multi-appareils utilisent un identifiant unique associé de façon persistante. Le RSSI ne sert qu’à la première sélection de proximité et ne change jamais un voyant déjà associé. Cet identifiant est généralement une adresse MAC sous Windows et peut être un UUID CoreBluetooth sous macOS.
- La mise à jour du programme et l’OTA du firmware sont des processus différents : la mise à jour du programme installe un EXE Windows ; l’OTA du firmware écrit uniquement l’image d’application sur les appareils compatibles.
