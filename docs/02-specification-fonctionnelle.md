# Spécification fonctionnelle — CortexOS IA

| | |
|---|---|
| **Document** | Spécification fonctionnelle |
| **Projet** | CortexOS IA |
| **Document de référence** | Cahier des charges — version 2.0 |
| **Remplace** | « CortexOS AI — Spécification complète des fonctionnalités », version 1.0 |
| **Auteur** | GOMINA Eloge |
| **Version du document** | 1.0 — 23 septembre 2026 |

> Convention : `[À DÉFINIR — D-xx]` signale une décision non encore prise (voir [05-decisions.md](05-decisions.md)). `[SOURCE À AJOUTER]` signale une affirmation à sourcer. **(Proposé)** signale un élément ajouté par ce document sans décision explicite : il doit être validé ou retiré.

---

## Sommaire

1. Objet, périmètre et vocabulaire
2. Principes fonctionnels
3. Acteurs et parcours
4. Comportement fonctionnel de la plateforme
5. Interface Web
6. Accessibilité de l'interface Web
7. Exigences de qualité de l'interface
8. Hors périmètre

Annexe A — Traçabilité · Annexe B — Décisions ouvertes · Annexe C — Devenir de l'ancienne spécification

---

## 1. Objet, périmètre et vocabulaire

### 1.1 Objet

Ce document décrit **comment CortexOS IA se comporte** du point de vue des personnes qui l'utilisent : règles de fonctionnement, états, parcours et fonctionnalités de l'interface Web.

Il ne décrit ni les algorithmes (traitement du signal, modèles d'IA), ni les technologies, ni l'organisation du code. Le **Cahier des charges** reste la référence pour la vision, le périmètre et les exigences : aucune fonction décrite ici n'élargit ce périmètre.

### 1.2 Place dans la documentation

| Document | Question traitée |
|---|---|
| Cahier des charges | **Quoi** : vision, périmètre, exigences (BF-xx, ENF-xx) |
| **Spécification fonctionnelle** | **Comment le système se comporte** : règles, états, parcours, écrans |
| Architecture technique | Comment les modules sont organisés et communiquent |
| Documentation technique | Comment chaque module fonctionne en détail (signal, IA, Core, connecteurs, protocole de test) |
| Planning MVP | Dans quel ordre et à quelle date |
| Plan d'apprentissage | Ce qu'il faut apprendre pour chaque partie |

### 1.3 Identifiants et statuts

- **F-xx** : fonction de la plateforme (section 4).
- **FW-xx** : fonctionnalité de l'interface Web (section 5).
- **BF-xx / ENF-xx** : exigences du Cahier des charges auxquelles chaque fonction se rattache (Annexe A).
- **Priorité** : **MVP**, **Ext.** (extension), **Futur** (recherche et futur) ou `[À DÉFINIR — D-xx]`, selon les horizons définis en section 4 du Cahier des charges.

### 1.4 Vocabulaire

Les termes du glossaire du Cahier des charges s'appliquent. La chaîne décrite dans ce document est toujours la suivante :

> **intention → détection → décision → commande → action → résultat**

| Terme | Définition |
|---|---|
| **Détection** | Intention reconnue par le système, avec son niveau de confiance |
| **Décision** | Choix du Core pour une détection : acceptée, rejetée (avec un motif) ou en attente de confirmation |
| **Commande** | Instruction envoyée à un système cible après une décision acceptée |
| **Action** | Effet produit par le système cible |
| **Résultat** | Retour du système cible : succès ou échec |
| **Intention attendue** | Intention que le protocole demande de produire à un instant donné ; elle sert à mesurer la précision en session d'expérimentation |
| **Source des données** | Origine du signal : casque réel, simulation ou enregistrement rejoué |
| **État sûr** | État dans lequel aucune commande ne peut être exécutée |
| **Commande sensible** | Commande qui exige une confirmation avant exécution (liste `[À DÉFINIR — D-24]`) |
| **Session d'utilisation / d'expérimentation** | Session destinée à utiliser le système, ou à le mesurer selon un protocole `[À DÉFINIR — D-18]` |

Le système ne « lit » pas les pensées : il reconnaît uniquement un ensemble limité d'intentions préalablement définies et détectables.

---

## 2. Principes fonctionnels

Ces principes s'appliquent à toutes les fonctions et à tous les écrans.

| # | Principe | Conséquence |
|---|---|---|
| P1 | **Transparence** | Chaque étape de la chaîne est visible dans l'interface et enregistrée dans le journal |
| P2 | **Sûreté par défaut** | En cas de doute (confiance insuffisante, signal mauvais, erreur, déconnexion), aucune commande n'est exécutée (ENF-03) |
| P3 | **Contrôle humain** | L'exécution des commandes peut toujours être suspendue ; un arrêt indépendant de l'EEG existe (ENF-04, `[À DÉFINIR — D-19]`) |
| P4 | **Honnêteté** | La source des données est toujours indiquée ; aucune valeur d'exemple n'est présentée comme un résultat ; aucune formulation de type « pensée » |
| P5 | **Mesurabilité** | Toute session d'expérimentation produit des mesures traçables |
| P6 | **Sobriété** | Aucune fonctionnalité ni visualisation sans objectif lié à une exigence |
| P7 | **Accessibilité de l'interface** | L'interface Web est elle-même utilisable par des profils variés (section 6) |
| P8 | **Cohérence** | Ce que l'interface affiche correspond exactement à ce qui est enregistré dans le journal |

---

## 3. Acteurs et parcours

### 3.1 Acteurs

| Acteur | Ce qu'il fait avec CortexOS | Accès |
|---|---|---|
| Utilisateur | Porte le casque, réalise la calibration, déclenche des commandes par ses intentions, gère son consentement et ses données | `[À DÉFINIR — D-09]` |
| Accompagnant / opérateur | Suit la session, peut suspendre les commandes et, selon décision, confirmer ou arrêter | `[À DÉFINIR — D-09, D-19, D-24]` |
| Expérimentateur | Prépare et conduit les sessions d'expérimentation, consulte les mesures, exporte les résultats | `[À DÉFINIR — D-09]` |
| Administrateur | Gère les comptes et les rôles (D-47) | `[À DÉFINIR — D-09]` (authentification) |

Une même personne peut tenir plusieurs rôles, notamment pendant le développement (D-47).

### 3.2 Parcours A — Préparer une première utilisation

1. L'utilisateur prend connaissance des traitements de données et donne son consentement (F-40).
2. Un profil est créé ou sélectionné (F-39).
3. Le casque est connecté ; la source des données est affichée (F-01, F-03).
4. La qualité du signal est vérifiée ; tant qu'elle est insuffisante, la calibration ne peut pas démarrer (F-02, F-07).
5. L'utilisateur suit la calibration guidée (F-06).
6. Le résultat indique si le modèle est exploitable ; sinon, la calibration peut être recommencée (F-08, F-09).
7. Le système passe à l'état **Prêt**, commandes suspendues (section 4.1).

### 3.3 Parcours B — Utiliser CortexOS pour commander un système cible

1. L'utilisateur, ou l'accompagnant, vérifie l'état de la chaîne et du système cible (FW-01, FW-05).
2. Les commandes sont activées explicitement (F-19).
3. L'utilisateur produit une intention ; l'interface affiche la détection, la confiance et la décision (FW-03).
4. Si la décision est acceptée, la commande est envoyée et l'action est exécutée ; le résultat est affiché (FW-04).
5. Si la commande est sensible, une confirmation est demandée avant l'envoi (F-18).
6. Si la décision est rejetée, le motif est affiché ; aucune commande n'est envoyée (F-21).
7. L'utilisateur ou l'accompagnant peut suspendre les commandes à tout moment (F-19).

### 3.4 Parcours C — Conduire une session d'expérimentation

1. L'expérimentateur crée une session : profil, scénario, conditions, notes (F-27).
2. Il démarre la session ; l'enregistrement commence (F-28, F-29).
3. Le protocole indique l'intention attendue à chaque essai, et celle-ci est enregistrée (F-30).
4. La session peut être mise en pause (fatigue, incident), puis reprise ou arrêtée (F-28).
5. À la fin, les mesures sont calculées et consultables (F-31).
6. Les résultats peuvent être exportés si le consentement le permet (F-35).

### 3.5 Parcours D — Gérer un incident

1. Un problème survient : signal dégradé, casque déconnecté, cible indisponible, commande non voulue, perte de l'interface.
2. Le système passe dans l'**état sûr** ou refuse les commandes concernées (F-04, F-05, F-16).
3. Une alerte est affichée, avec sa gravité et l'action possible (FW-06, FW-37).
4. L'utilisateur ou l'accompagnant peut suspendre les commandes, ou utiliser l'arrêt indépendant de l'EEG (F-19, F-20).
5. Après résolution, les commandes restent suspendues jusqu'à une reprise explicite `[À DÉFINIR — D-29]`.
6. L'incident est enregistré dans le journal (F-36).

### 3.6 Parcours E — Gérer ses données

1. L'utilisateur consulte ses sessions, calibrations et consentements (FW-32, FW-39, FW-40).
2. Il peut exporter ses données (F-35), retirer son consentement ou demander la suppression de ses données (F-41).

---

## 4. Comportement fonctionnel de la plateforme

### 4.1 États globaux de CortexOS

| État | Signification | Commandes exécutables |
|---|---|---|
| **Arrêté** | Aucune acquisition en cours | Non |
| **Préparation** | Casque connecté ou source choisie, pas de modèle exploitable pour ce profil | Non |
| **Calibration** | Calibration en cours | Non |
| **Prêt** | Modèle exploitable, chaîne fonctionnelle, commandes suspendues | Non |
| **Actif** | Commandes autorisées | Oui, sous réserve des règles de la section 4.5 |
| **Suspendu** | Commandes suspendues par une personne | Non |
| **État sûr** | Suspension automatique après un incident (signal, connexion, erreur) | Non |

Règles :

- Seul l'état **Actif** permet d'exécuter des commandes.
- Le passage à **Actif** demande toujours une action explicite d'une personne.
- Au démarrage et après tout retour de l'**état sûr**, le système se place dans un état où les commandes sont suspendues **(Proposé)** `[À DÉFINIR — D-29]`.
- L'état courant est toujours visible dans l'interface (FW-25).

### 4.2 Acquisition et qualité du signal

| ID | Fonction | Priorité | Exigence |
|---|---|---|---|
| F-01 | Connecter et déconnecter le casque ; connaître son état de connexion et ses caractéristiques (modèle, nombre de canaux, fréquence d'échantillonnage) | MVP (valeurs selon `[D-04]`) | BF-01, BF-02 |
| F-02 | Évaluer en continu la qualité du signal, globale et, si le casque le permet, par canal | MVP (par canal : `[D-04]`) | BF-02 |
| F-03 | Identifier la source des données : casque réel, simulation ou enregistrement rejoué | MVP si la simulation ou le rejeu existe `[D-21]` | ENF-09 |
| F-04 | Quand la qualité est insuffisante, ne pas utiliser les détections pour déclencher des commandes et émettre une alerte | MVP | ENF-02, ENF-03 |
| F-05 | En cas de perte du signal ou de déconnexion du casque, passer à l'état sûr et émettre une alerte ; une reconnexion ne réactive pas les commandes | MVP (reprise : `[D-29]`) | ENF-02 |

Le calcul de la qualité du signal relève de la Documentation technique.

### 4.3 Calibration

**États d'une calibration** : non commencée → en cours → terminée (exploitable ou insuffisante), ou interrompue.

| ID | Fonction | Priorité | Exigence |
|---|---|---|---|
| F-06 | Guider l'utilisateur par des consignes, pour chacune des intentions retenues, et indiquer la progression | MVP (intentions : `[D-03]`) | BF-06 |
| F-07 | Refuser le démarrage d'une calibration tant que la qualité du signal est insuffisante | MVP **(Proposé)** | BF-06, ENF-03 |
| F-08 | Permettre d'interrompre ou de recommencer une calibration ; une calibration interrompue n'est jamais utilisée | MVP | BF-06 |
| F-09 | Indiquer si le résultat est exploitable, associer le modèle obtenu au profil et enregistrer sa version | MVP (critère : `[D-10]`) | BF-07 |
| F-10 | Signaler qu'une calibration est ancienne ou que les performances baissent, et recommander une nouvelle calibration | Ext. `[D-33]` | BF-09 |

La méthode de calibration et d'entraînement relève de la Documentation technique.

### 4.4 Détection d'intention

| ID | Fonction | Priorité | Exigence |
|---|---|---|---|
| F-11 | Produire des détections horodatées : une intention parmi l'ensemble retenu et son niveau de confiance | MVP (intentions : `[D-03]`) | BF-08 |
| F-12 | Traiter l'état de repos ou l'absence d'intention comme ne déclenchant aucune commande **(Proposé)** | `[À DÉFINIR — D-28]` | BF-08, ENF-03 |
| F-13 | Présenter une détection comme la reconnaissance d'une intention définie, jamais comme la lecture d'une pensée | MVP | Cahier 4.7 |

### 4.5 Décision et sûreté

**Cycle de vie d'une commande**

| Étape | États possibles |
|---|---|
| 1. Détection | Détectée |
| 2. Décision | Rejetée (avec motif) · Acceptée · En attente de confirmation |
| 3. Confirmation (commandes sensibles) | Confirmée · Annulée · Expirée |
| 4. Envoi | Envoyée |
| 5. Résultat | Exécutée (succès) · Échouée |

**Motifs de rejet** : confiance insuffisante, qualité du signal insuffisante, système non actif (Prêt, Suspendu, état sûr), système cible indisponible, délai minimal entre deux commandes non écoulé **(Proposé)**, commande non autorisée.

| ID | Fonction | Priorité | Exigence |
|---|---|---|---|
| F-14 | Associer chaque intention à une commande selon une correspondance active ; la correspondance est consultable | MVP (modification : `[D-20]`) | BF-10, BF-14 |
| F-15 | Rejeter toute détection dont la confiance est inférieure au seuil | MVP (valeur : `[D-10]` ; modification : `[D-23]`) | BF-11 |
| F-16 | Rejeter toute détection quand le système n'est pas Actif, quand le signal est insuffisant ou quand la cible est indisponible | MVP | ENF-03 |
| F-17 | Imposer un délai minimal entre deux commandes pour éviter les répétitions involontaires | **(Proposé)** `[D-30]` | ENF-03 |
| F-18 | Demander une confirmation avant toute commande sensible ; sans confirmation dans le délai prévu, la commande expire | MVP (liste et modalité : `[D-24]` ; délai : `[D-31]`) | BF-12 |
| F-19 | Permettre de suspendre et de reprendre l'exécution des commandes ; la reprise est toujours explicite | MVP | BF-13 |
| F-20 | Disposer d'un moyen d'arrêt indépendant de la détection EEG et de l'interface Web | MVP (mécanisme et déclencheur : `[D-19]`) | ENF-04 |
| F-21 | Enregistrer et afficher le motif de chaque rejet | MVP | BF-18, ENF-09 |
| F-22 | Recevoir le résultat de chaque action ; sans retour dans un délai défini, considérer la commande comme échouée **(Proposé)** | MVP | BF-15 |

Le seuil de confiance ne garantit pas qu'une détection est correcte. Il réduit seulement le nombre de commandes envoyées sur des détections peu sûres.

### 4.6 Systèmes cibles

| ID | Fonction | Priorité | Exigence |
|---|---|---|---|
| F-23 | Connaître l'état de chaque système cible (disponible, indisponible) et transmettre les commandes à au moins un système cible réel | MVP `[D-05]` | BF-15 |
| F-24 | Limiter chaque système cible à une liste fermée de commandes définies par son connecteur **(Proposé)** | MVP | ENF-03 |
| F-25 | Ajouter un nouveau type de système cible sous forme de connecteur, sans modifier le Core | Ext. | BF-16, ENF-07 |
| F-26 | Permettre un test manuel d'un système cible, sans EEG ; ces commandes sont signalées comme manuelles et exclues des mesures | `[À DÉFINIR — D-27]` | — |

**Trajectoire des systèmes cibles**, conforme à la section 4.1 du Cahier des charges :

| Système cible | Horizon |
|---|---|
| Ordinateur | MVP ou Ext. `[D-05]` |
| Objets connectés | MVP ou Ext. `[D-05]` |
| Systèmes embarqués | Ext. |
| Robot en simulation | Ext. ou Futur `[D-11]` |
| Robot réel, drone | Futur `[D-11]` |

### 4.7 Sessions et mesures

**États d'une session** : créée → en cours ⇄ en pause → terminée, ou interrompue (incident).

| ID | Fonction | Priorité | Exigence |
|---|---|---|---|
| F-27 | Créer une session avec son type (utilisation ou expérimentation), son profil, son scénario, ses conditions et des notes ; la source des données et la version du modèle sont enregistrées automatiquement | MVP (types : `[D-18]` ; scénarios : `[D-05]`) | BF-19 |
| F-28 | Démarrer, mettre en pause, reprendre et arrêter une session ; la mise en pause suspend les commandes | MVP (durée maximale et pauses : `[D-37]`) | BF-03, BF-19 |
| F-29 | Enregistrer le signal, les détections, les décisions, les commandes, les actions, les résultats et leurs horodatages | MVP | BF-19, ENF-10 |
| F-30 | En session d'expérimentation, enregistrer l'intention attendue à chaque essai | MVP `[D-18]` | BF-20 |
| F-31 | Calculer les mesures de la session (voir tableau ci-dessous) | MVP (seuils : `[D-10]`) | BF-20 |
| F-32 | Distinguer toujours les mesures obtenues sur casque réel de celles obtenues en simulation ou sur enregistrement ; ne jamais les agréger ensemble sans l'indiquer | MVP **(Proposé)** | ENF-09 |
| F-33 | Comparer plusieurs sessions | Ext. **(Proposé)** | BF-20 |
| F-34 | Rejouer une session enregistrée | Ext. `[À DÉFINIR]` | ENF-10 |
| F-35 | Exporter les données et résultats d'une session, avec leurs métadonnées, uniquement si le consentement le permet | MVP (pseudonymisation : `[D-34]` ; formats : Documentation technique) | BF-21 |

**Mesures d'une session** (section 3.3 du Cahier des charges) :

| Mesure | Définition fonctionnelle | Condition |
|---|---|---|
| Précision de détection | Part des détections égales à l'intention attendue, comparée au niveau du hasard | Session d'expérimentation |
| Matrice de confusion | Répartition des détections selon l'intention attendue | Session d'expérimentation |
| Latence de bout en bout | Délai entre la fin de la fenêtre de signal analysée et la commande exécutée | Toutes sessions |
| Taux de rejet | Part des détections rejetées, par motif | Toutes sessions |
| Taux de commandes involontaires | Part des commandes exécutées alors que l'intention attendue était différente ou qu'aucune commande n'était attendue | Session d'expérimentation |
| Réussite du scénario | Part des scénarios menés à terme | Selon le scénario `[D-05]` |

La précision et les commandes involontaires ne peuvent être mesurées que si l'intention attendue est connue. **Elles ne sont donc pas affichées en direct lors d'une session d'utilisation.**

### 4.8 Journal et alertes

| ID | Fonction | Priorité | Exigence |
|---|---|---|---|
| F-36 | Tenir un **journal fonctionnel unique** : connexions, qualité du signal, calibrations, détections, décisions et motifs, commandes, actions, résultats, suspensions, reprises, arrêts, événements de session, erreurs, et modifications de paramètres (seuil, correspondance) avec leur auteur | MVP | BF-25, ENF-10 |
| F-37 | Classer les événements par gravité (information, avertissement, erreur, critique) et émettre une alerte pour ceux qui demandent une réaction | MVP (alertes non visuelles : `[D-35]`) | BF-18 |
| F-38 | Conserver le journal selon une durée définie | `[D-09]` | ENF-06 |

Le journal fonctionnel est destiné aux utilisateurs. Les journaux techniques du code relèvent de la Documentation technique.

### 4.9 Profils, consentement et données

| ID | Fonction | Priorité | Exigence |
|---|---|---|---|
| F-39 | Gérer un profil : identifiant ou pseudonyme, calibrations, modèles, correspondance intention → commande (si elle est modifiable), préférences d'affichage | MVP (forme : `[D-09]`) | BF-22 |
| F-40 | Exiger un consentement explicite avant tout enregistrement de données EEG ; préciser sa portée (utilisation, expérimentation, export) ; le consentement est consultable | MVP (portée : `[D-09, D-34]`) | BF-23, ENF-06 |
| F-41 | Permettre de retirer son consentement et de supprimer ses données ; le retrait arrête tout nouvel enregistrement | MVP (périmètre de la suppression : `[D-38]`) | BF-24 |
| F-42 | Limiter l'accès aux données et aux fonctions selon le rôle | `[D-09]` | ENF-05 |
| F-43 | Authentifier les personnes qui accèdent à l'interface | `[D-09]` | ENF-05 |
| F-44 | Ne collecter que les données nécessaires aux fonctions décrites ici **(Proposé)** | MVP | ENF-06 |

---

## 5. Interface Web

L'interface Web est l'interface de **supervision, de configuration, de calibration et d'expérimentation** de CortexOS. Elle ne contient pas de logique de décision : elle affiche l'état de la plateforme et transmet les actions des personnes.

Elle est organisée en **9 types de fonctionnalités**, regroupés en trois familles :

| Famille | Types | Utilisateur principal |
|---|---|---|
| **Préparer** | Profil, consentement et données · Casque et signal · Calibration | Utilisateur |
| **Utiliser** | Supervision · Commandes et systèmes cibles · Sûreté | Utilisateur, accompagnant |
| **Mesurer** | Sessions et mesures · Journal et alertes | Expérimentateur |
| **Transversal** | Accessibilité et aide | Tous |

### 5.1 Supervision

| ID | Fonctionnalité | Objectif | Priorité | Réf. |
|---|---|---|---|---|
| FW-01 | État de la chaîne étape par étape : casque, traitement, détection, Core, système cible | Savoir immédiatement si CortexOS fonctionne | MVP | 4.1, F-01, F-23 |
| FW-02 | Indicateur permanent de la source des données | Ne jamais présenter une simulation comme une détection réelle | MVP si la simulation ou le rejeu existe `[D-21]` | F-03 |
| FW-03 | Intention détectée, niveau de confiance et décision (acceptée, rejetée avec motif, en attente) | Comprendre pourquoi une commande part ou non | MVP | F-11, F-15, F-21 |
| FW-04 | Commande envoyée, action exécutée et résultat | Voir ce qui s'est réellement passé | MVP | F-14, F-22 |
| FW-05 | État du système cible actif | Savoir si la cible peut recevoir des commandes | MVP | F-23 |
| FW-06 | Fil des événements importants récents | Réagir sans ouvrir le journal | MVP | F-36, F-37 |
| FW-07 | Indicateurs de la session en cours : durée, détections, rejets, commandes, latence | Suivre la session en direct (sans précision en direct, voir 4.7) | MVP | F-31 |

### 5.2 Casque et signal

| ID | Fonctionnalité | Objectif | Priorité | Réf. |
|---|---|---|---|---|
| FW-08 | Connexion et déconnexion du casque, état de connexion | Démarrer l'acquisition | MVP | F-01 |
| FW-09 | Caractéristiques du casque : modèle, nombre de canaux, fréquence d'échantillonnage | Connaître la configuration réelle | MVP (valeurs : `[D-04]`) | F-01 |
| FW-10 | Qualité globale du signal | Éviter de détecter sur un mauvais signal | MVP | F-02 |
| FW-11 | Qualité du signal par canal | Repérer une électrode mal placée | MVP ou Ext. `[D-04]` | F-02 |
| FW-12 | Visualisation du signal en temps réel, canaux sélectionnables | Vérifier le signal et démontrer la chaîne | MVP | F-02 |
| FW-13 | Comparaison du signal brut et du signal filtré | Comprendre l'effet du traitement | Ext. | — |
| FW-14 | Bandes de fréquences et spectre | Exploration scientifique | `[À DÉFINIR]` (Ext. proposé) | — |
| FW-15 | Alertes sur le signal : déconnexion, qualité insuffisante | Signaler un problème sans attendre | MVP | F-04, F-05 |

### 5.3 Calibration

| ID | Fonctionnalité | Objectif | Priorité | Réf. |
|---|---|---|---|---|
| FW-16 | Calibration guidée : consignes, progression | Construire le modèle de l'utilisateur | MVP | F-06, F-07 |
| FW-17 | État et résultat de la calibration ; interrompre ou recommencer | Savoir si le modèle est exploitable | MVP | F-08, F-09 |
| FW-18 | Dernière calibration et modèle actif du profil | Savoir avec quel modèle on travaille | MVP | F-09 |
| FW-19 | Historique et comparaison des calibrations, choix d'une version de modèle | Suivre l'évolution | Ext. | F-09 |
| FW-47 | Indication de validité de la calibration et recommandation de recalibrer | Éviter de travailler avec un modèle dépassé | Ext. `[D-33]` | F-10 |

### 5.4 Commandes et systèmes cibles

| ID | Fonctionnalité | Objectif | Priorité | Réf. |
|---|---|---|---|---|
| FW-20 | Consultation de la correspondance intention → commande active | Savoir ce que déclenche chaque intention | MVP | F-14 |
| FW-21 | Modification de la correspondance intention → commande | Adapter le système à l'usage | `[D-20]` | F-14 |
| FW-22 | Liste des systèmes cibles et de leur état | Choisir et surveiller la cible | MVP | F-23 |
| FW-23 | Test manuel d'un système cible, clairement signalé | Diagnostiquer la cible sans EEG | `[D-27]` | F-26 |
| FW-24 | Affichage des types de systèmes cibles selon la trajectoire | Rendre visibles les horizons MVP, Ext. et Futur | Selon 4.6 | F-24, F-25 |

### 5.5 Sûreté

| ID | Fonctionnalité | Objectif | Priorité | Réf. |
|---|---|---|---|---|
| FW-25 | État global de CortexOS (section 4.1) toujours visible, en particulier « commandes actives » ou « suspendues » | Savoir en permanence si CortexOS peut agir | MVP | 4.1 |
| FW-26 | Suspendre et reprendre les commandes | Garder le contrôle | MVP. **Ce bouton ne remplace pas l'arrêt indépendant de l'EEG (F-20)** | F-19 |
| FW-27 | Affichage du seuil de confiance ; modification | Comprendre et régler la sévérité du filtrage | Affichage : MVP. Modification : `[D-23]` | F-15 |
| FW-28 | Confirmation d'une commande sensible, avec le délai restant | Éviter une action sensible involontaire | MVP (modalité : `[D-24]` ; délai : `[D-31]`) | F-18 |

### 5.6 Sessions et mesures

| ID | Fonctionnalité | Objectif | Priorité | Réf. |
|---|---|---|---|---|
| FW-29 | Création d'une session : type, profil, scénario, conditions, notes | Préparer une utilisation ou une mesure | MVP | F-27 |
| FW-30 | Démarrer, mettre en pause, reprendre, arrêter une session | Conduire la session | MVP | F-28, F-29 |
| FW-31 | Mesures d'une session : précision et niveau du hasard, matrice de confusion, latence, taux de rejet, commandes involontaires | Évaluer objectivement | MVP | F-31, F-32 |
| FW-32 | Liste des sessions passées, avec type et source des données | Retrouver une session | MVP | F-27 |
| FW-33 | Comparaison de sessions | Observer une évolution | Ext. **(Proposé)** | F-33 |
| FW-34 | Export d'une session | Analyser hors de la plateforme | MVP | F-35 |
| FW-35 | Rejeu d'une session enregistrée | Analyser a posteriori | Ext. `[À DÉFINIR]` | F-34 |

### 5.7 Journal et alertes

| ID | Fonctionnalité | Objectif | Priorité | Réf. |
|---|---|---|---|---|
| FW-36 | Journal unique, filtrable par session, type et gravité | Comprendre après coup ce qui s'est passé | MVP | F-36 |
| FW-37 | Messages d'erreur compréhensibles : cause et action possible | Ne jamais échouer silencieusement | MVP | F-37 |
| FW-38 | Vue technique de l'état des services | Diagnostic | Ext. ou `[À DÉFINIR]` | — |
| FW-48 | Alertes non visuelles (par exemple sonores) | Alerter une personne qui ne regarde pas l'écran | `[D-35]` | F-37 |
| FW-50 | Historique des modifications de paramètres (seuil, correspondance), avec auteur et date | Savoir qui a changé quoi | MVP si ces paramètres sont modifiables | F-36 |

### 5.8 Profil, consentement et données

| ID | Fonctionnalité | Objectif | Priorité | Réf. |
|---|---|---|---|---|
| FW-39 | Choix ou création d'un profil | Associer calibration et modèle à une personne | MVP (forme : `[D-09]`) | F-39 |
| FW-40 | Recueil et consultation du consentement | Respecter l'utilisateur | MVP | F-40 |
| FW-41 | Retrait du consentement et suppression des données | Maîtrise de ses données | MVP | F-41 |
| FW-42 | Authentification, rôles, administration des comptes | Sécuriser l'accès | `[D-09]` | F-42, F-43 |

### 5.9 Accessibilité et aide

| ID | Fonctionnalité | Objectif | Priorité | Réf. |
|---|---|---|---|---|
| FW-43 | Aide contextuelle | Comprendre chaque écran sans documentation externe | Ext. | — |
| FW-44 | Interface conforme aux exigences de la section 6 | Interface utilisable par tous | MVP (niveau : `[D-26]`) | ENF-09 |
| FW-45 | Vue simplifiée pour l'utilisateur, vue détaillée pour l'expérimentateur | Adapter l'information au rôle | `[D-18]` | — |
| FW-46 | Pilotage de l'interface Web elle-même par intentions | Rendre l'interface utilisable sans geste | Futur `[D-25]` | — |
| FW-49 | Indicateur de fraîcheur des données affichées (dernière mise à jour, perte de connexion) | Ne jamais afficher un état périmé comme actuel | MVP **(Proposé)** | P8, section 7 |

---

## 6. Accessibilité de l'interface Web

Deux sujets distincts :

- **CortexOS comme outil d'accessibilité** : c'est une dimension du projet (Cahier des charges, section 1.6), dont le MVP ne démontre pas le bénéfice sans tests avec le public concerné (Cahier des charges, section 4.3).
- **L'accessibilité de l'interface Web** : l'interface doit être utilisable par l'utilisateur, l'accompagnant et l'expérimentateur, quels que soient leurs moyens d'interaction. C'est l'objet de cette section.

| ID | Exigence | Priorité |
|---|---|---|
| A-01 | Toutes les fonctions sont utilisables au clavier seul, avec un focus toujours visible | MVP |
| A-02 | Contrastes suffisants, taille du texte ajustable sans perte d'information | MVP |
| A-03 | Éléments correctement libellés pour les lecteurs d'écran | MVP |
| A-04 | Aucune information portée uniquement par la couleur (états, qualité, alertes) | MVP |
| A-05 | Messages courts, en langage simple, sans jargon technique non expliqué | MVP |
| A-06 | Aucun délai imposé trop court : tout délai (par exemple de confirmation) est ajustable ou suffisamment long `[D-31]` | MVP |
| A-07 | Aucun contenu clignotant ou à flash rapide, pour la sécurité des personnes photosensibles et pour limiter la perturbation du signal EEG par des stimulations visuelles `[SOURCE À AJOUTER]` | MVP **(Proposé)** |
| A-08 | Alertes importantes également perceptibles autrement que visuellement | `[D-35]` |
| A-09 | Langue de l'interface | `[D-36]` |
| A-10 | Pilotage de l'interface par intentions | Futur `[D-25]` |

Le niveau de conformité visé, par exemple un niveau des règles WCAG, est `[À DÉFINIR — D-26]`.

---

## 7. Exigences de qualité de l'interface

| ID | Exigence |
|---|---|
| Q-01 | L'état affiché est mis à jour en temps réel ; le délai d'affichage acceptable est `[À DÉFINIR — D-10]` |
| Q-02 | Toute perte de connexion entre l'interface et la plateforme est signalée immédiatement ; les données affichées indiquent alors leur dernière mise à jour (FW-49) |
| Q-03 | Le comportement de la plateforme quand l'interface est fermée ou déconnectée pendant une session active est `[À DÉFINIR — D-32]` |
| Q-04 | Ce qui est affiché correspond exactement à ce qui est enregistré dans le journal (P8) |
| Q-05 | Aucune valeur d'exemple ni donnée fictive n'apparaît dans l'interface en dehors d'un mode de démonstration explicitement signalé |
| Q-06 | Les actions qui ont un effet sur la sûreté (reprise des commandes, modification du seuil ou de la correspondance) demandent une validation explicite et sont journalisées |

---

## 8. Hors périmètre

| Élément | Raison |
|---|---|
| Pilotage d'applications précises (traitement de texte, messagerie, environnements de développement, jeux) | Hors périmètre du Cahier des charges (4.6) |
| Actions système sensibles sur l'ordinateur (arrêt, redémarrage) | Risque élevé, aucune exigence correspondante |
| Extensions ou plugins pour des logiciels tiers | Hors périmètre du Cahier des charges (4.6) |
| Application mobile | Aucune exigence dans le Cahier des charges |
| Statistiques de ressources (CPU, GPU, mémoire) pour l'utilisateur | Aucun lien avec les objectifs |
| Inscription publique en libre accès | Non nécessaire à une plateforme expérimentale `[D-09]` |
| Configuration des filtres, du buffer, de la fenêtre d'analyse, du broker MQTT | Paramètres techniques : Documentation technique |
| Pilotage d'un drone réel | Futur (Cahier des charges 4.5) |

---

## Annexe A — Traçabilité

| Exigence du Cahier des charges | Fonctions | Fonctionnalités Web |
|---|---|---|
| BF-01 Connexion au casque | F-01 | FW-08, FW-09 |
| BF-02 Qualité et état du casque | F-01, F-02 | FW-10, FW-11, FW-12 |
| BF-03 Démarrer / arrêter l'acquisition | F-28 | FW-30 |
| BF-04, BF-05 Traitement du signal | — (Documentation technique) | FW-13, FW-14 |
| BF-06 Calibration guidée | F-06, F-07, F-08 | FW-16, FW-17 |
| BF-07 Modèle propre à l'utilisateur | F-09 | FW-18 |
| BF-08 Détection avec confiance | F-11, F-12, F-13 | FW-03 |
| BF-09 Adaptation progressive | F-10 | FW-19, FW-47 |
| BF-10 Association intention → commande | F-14 | FW-20 |
| BF-11 Rejet sous le seuil | F-15 | FW-03, FW-27 |
| BF-12 Confirmation des commandes sensibles | F-18 | FW-28 |
| BF-13 Arrêt de l'exécution | F-19 | FW-25, FW-26 |
| BF-14 Correspondance configurable | F-14 | FW-21 |
| BF-15 Au moins un système cible réel | F-22, F-23 | FW-04, FW-05, FW-22 |
| BF-16 Nouveau type de système cible | F-25 | FW-24 |
| BF-17 Supervision temps réel | F-11, F-14, F-22 | FW-01 à FW-07 |
| BF-18 Signalement des erreurs et rejets | F-21, F-37 | FW-06, FW-15, FW-37 |
| BF-19 Enregistrement des sessions | F-27, F-28, F-29 | FW-29, FW-30, FW-32 |
| BF-20 Calcul des mesures | F-30, F-31, F-33 | FW-31, FW-33 |
| BF-21 Export | F-35 | FW-34 |
| BF-22 Profils et modèles | F-39 | FW-39 |
| BF-23 Consentement | F-40 | FW-40 |
| BF-24 Suppression des données | F-41 | FW-41 |
| BF-25 Journalisation | F-36 | FW-36, FW-50 |
| ENF-03 / ENF-04 Sûreté | F-04, F-05, F-16, F-17, F-20, F-24 | FW-25, FW-26 |
| ENF-05 / ENF-06 Sécurité, confidentialité | F-38, F-40 à F-44 | FW-40 à FW-42 |
| ENF-09 Utilisabilité et accessibilité | F-03, F-21, F-32 | FW-02, FW-44, FW-49, section 6 |
| ENF-10 Traçabilité | F-29, F-34, F-36 | FW-35, FW-36 |

---

## Annexe B — Décisions ouvertes

### B.1 Décisions du Cahier des charges qui concernent ce document

| ID | Question (résumée) | Effet sur ce document |
|---|---|---|
| D-03 | Intentions du MVP | Consignes de calibration (F-06), libellés affichés |
| D-04 | Casque | Qualité par canal (F-02, FW-11), caractéristiques affichées (FW-09) |
| D-05 | Scénarios et système cible du MVP | F-23, F-27, FW-22, FW-24 |
| D-09 | Stockage, authentification, rôles, conservation | Section 3.1, F-38 à F-43, FW-42 |
| D-10 | Seuils et cibles de performance | F-09, F-15, F-31, Q-01 |
| D-11 | Robot et drone | Trajectoire (4.6) |
| D-18 | Mode expérimentation / mode utilisation | F-27, F-30, mesures (4.7), FW-45 |
| D-19 | Arrêt d'urgence et déclencheur | F-20, parcours D |
| D-20 | Correspondance configurable | F-14, FW-21 |
| D-21 | Stratégie de repli (simulation, enregistrements, jeux de données publics) | F-03, FW-02 |

### B.2 Nouvelles décisions ouvertes par ce document

D-23 à D-38 : voir le registre [05-decisions.md](05-decisions.md).

---

## Annexe C — Devenir de l'ancienne spécification

| Anciennes sections | Contenu | Devenir |
|---|---|---|
| §1, §2, §75, §76 | Présentation, objectifs, principe, vision | Couverts par le Cahier des charges ; principe repris en section 2 |
| §3, §4, §50–53 | Comptes, profil, permissions, rôles, administration | F-39 à F-43, FW-39 à FW-42, réduits en attendant D-09 |
| §5, §38, §40, §54 | Tableau de bord, ordinateurs, objets connectés, monitoring | Section 5.1, FW-22, FW-38 |
| §6, §8, §9, §10 | Casque, canaux, qualité, visualisation | F-01, F-02, section 5.2 |
| §7, §11–14 | Acquisition, traitement, filtres, bandes, FFT / PSD | Documentation technique (affichages éventuels : FW-13, FW-14) |
| §15, §16 | Calibration | F-06 à F-10, section 5.3 |
| §17–19, §22–26 | IA, classes, modèles, évaluation, jeux de données | Documentation technique ; affichages : FW-18, FW-19, FW-31 |
| §20, §21, §35, §49 | Confiance, seuil, validation, sécurité des commandes | Section 4.5 |
| §27–29 | Données, export, suppression | F-29, F-35, F-41 |
| §30–34 | Core, événements, journal, commandes, personnalisation | Sections 4.5 et 4.8 ; noms d'événements : Architecture technique |
| §36, §37, §39, §41, §42 | Contrôle PC, agent, objets connectés, MQTT, robots | Section 4.6 ; agent et MQTT : Architecture technique |
| §43–48, §59, §60 | Historiques, statistiques, latence, notifications, alertes, erreurs, logs | Sections 4.7 et 4.8 (journal unique) ; logs techniques : Documentation technique |
| §55 | Configuration système | Seuil : F-15 ; reste : Documentation technique |
| §56, §57, §64–67 | Temps réel, API, modules, technologies, base de données | Architecture technique |
| §58 | Documentation intégrée | FW-43 et manuel d'utilisation |
| §61 | Tests | Documentation technique et Planning MVP |
| §62, §63 | Mode simulation, mode réel | F-03, FW-02, D-21 ; mise en œuvre : Architecture technique |
| §68–70, §73 | Cycle et exemples, démonstration | Parcours (section 3) ; démonstration : Cahier des charges 4.2 et D-05 |
| §71, §72, §74 | Phases, priorités, résumé des modules | Planning MVP ; priorités remplacées par les colonnes « Priorité » |
