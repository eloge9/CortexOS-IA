# Développement sur PC seul — CortexOS IA

| | |
|---|---|
| **Document** | Fonctionnalités réalisables sans matériel (PC seul) |
| **Projet** | CortexOS IA `[À DÉFINIR — D-14 : graphie officielle]` |
| **Documents de référence** | Cahier des charges v2.0 · Spécification fonctionnelle v1.0 |
| **Situation** | Aucun casque EEG, aucun microcontrôleur, aucun robot ni drone : uniquement un ordinateur |
| **Auteur** | GOMINA Eloge |
| **Version du document** | 1.0 — 23 septembre 2026 |

> Légende : ✅ réalisable entièrement avec le PC seul · 🟡 à développer maintenant, à terminer ou valider avec le casque · ❌ impossible sans matériel. Les horizons (MVP, Ext., Futur) sont ceux du Cahier des charges : ce document ne les modifie pas.

---

## Sommaire

1. Objet
2. Ce que le PC remplace
3. Toutes les fonctions et fonctionnalités, avec leur faisabilité sur PC
4. Systèmes cibles : de l'ordinateur au drone
5. Ce qui attend le matériel
6. Ordre de travail conseillé
7. Règles à respecter pendant le développement sans matériel
8. Outils à installer

Annexe — Décisions concernées

---

## 1. Objet

Ce document indique **tout ce qui peut être développé dès maintenant avec un ordinateur seul**, en reprenant sans exception les 44 fonctions (F-xx) et les 50 fonctionnalités Web (FW-xx) de la Spécification fonctionnelle, ainsi que toute la trajectoire des systèmes cibles, de l'ordinateur jusqu'au drone.

**Bilan** : sur 94 fonctions et fonctionnalités, **79 sont réalisables entièrement** sur PC et **15 en partie**. Aucune n'est entièrement bloquée : ce qui attend le casque, ce sont les **données réelles** et les **mesures officielles** (section 5).

Ce document ne modifie ni le périmètre ni les priorités du Cahier des charges. Développer une fonction « Futur » en simulation ne la fait pas entrer dans le MVP.

---

## 2. Ce que le PC remplace

| Matériel absent | Remplacement sur PC | Limite |
|---|---|---|
| Casque EEG (signal en direct) | **Carte synthétique** BrainFlow : faux signal EEG en temps réel | Aucune intention dans ce signal : sert à tester la chaîne, pas l'IA |
| Données EEG réelles | **Jeux de données publics** d'imagerie motrice, avec leurs étiquettes (par exemple *EEG Motor Movement/Imagery* de PhysioNet) | Autres personnes, autre matériel : le modèle devra être réentraîné sur tes données |
| Détections en direct | **Rejeu** d'enregistrements publics à travers ton modèle | Ce ne sont pas tes intentions |
| Détections pendant le développement, avant l'IA | **Détecteur de test**, clairement signalé comme simulé | Uniquement pour développer le Core et l'interface |
| ESP32 et lampe | **Appareil simulé** (programme « lampe ») ; éventuellement un broker MQTT local | Pas d'action physique |
| Système embarqué | **Simulateur de microcontrôleur** (par exemple un simulateur d'ESP32 en ligne) | Optionnel |
| Robot | **Simulation ROS 2** (d'abord un robot 2D simple, puis un simulateur 3D) | Horizon Ext. ou Futur `[D-11]` |
| Drone | **Drone simulé** : version simple faite par toi, ou simulateur réaliste | Horizon Futur `[D-11]` |
| Système cible réel | **Ton ordinateur lui-même** (curseur, sélection…) | Aucune : c'est un système cible réel `[D-05, D-06]` |

La stratégie « simulation + données publiques » correspond à la décision D-21 du Cahier des charges. Ce document la retient pour la phase sans matériel **(Proposé)** ; elle reste à confirmer dans le Cahier des charges.

---

## 3. Toutes les fonctions et fonctionnalités, avec leur faisabilité sur PC

### 3.1 Fonctions de la plateforme (F-01 à F-44)

#### Acquisition et qualité du signal (spécification 4.2)

| ID | Fonction | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| F-01 | Connecter et déconnecter le casque ; connaître son état de connexion et ses caractéristiques (modèle, nombre de canaux, fréquence d'échantillonnage) | 🟡 | Carte synthétique BrainFlow | Vrai casque, connexion Bluetooth/USB (D-04) |
| F-02 | Évaluer en continu la qualité du signal, globale et, si le casque le permet, par canal | 🟡 | Algorithme écrit et testé sur enregistrements publics | Réglage des seuils sur ton signal |
| F-03 | Identifier la source des données : casque réel, simulation ou enregistrement rejoué | ✅ | Sources « simulation » et « enregistrement public » | Ajout de la source « casque » |
| F-04 | Quand la qualité est insuffisante, ne pas utiliser les détections pour déclencher des commandes et émettre une alerte | ✅ | Signal volontairement dégradé en simulation | — |
| F-05 | En cas de perte du signal ou de déconnexion du casque, passer à l'état sûr et émettre une alerte ; une reconnexion ne réactive pas les commandes | ✅ | Déconnexion simulée | — |

#### Calibration (spécification 4.3)

| ID | Fonction | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| F-06 | Guider l'utilisateur par des consignes, pour chacune des intentions retenues, et indiquer la progression | 🟡 | Parcours complet (consignes, progression) | Vraies données de calibration |
| F-07 | Refuser le démarrage d'une calibration tant que la qualité du signal est insuffisante | ✅ | Qualité simulée | — |
| F-08 | Permettre d'interrompre ou de recommencer une calibration ; une calibration interrompue n'est jamais utilisée | ✅ | Logique complète | — |
| F-09 | Indiquer si le résultat est exploitable, associer le modèle obtenu au profil et enregistrer sa version | 🟡 | Modèle entraîné par sujet d'un jeu public | Ton modèle personnel |
| F-10 | Signaler qu'une calibration est ancienne ou que les performances baissent, et recommander une nouvelle calibration | ✅ | Logique (extension) | Critère réel (D-33) |

#### Détection d'intention (spécification 4.4)

| ID | Fonction | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| F-11 | Produire des détections horodatées : une intention parmi l'ensemble retenu et son niveau de confiance | 🟡 | Détections sur enregistrements publics rejoués | Détections sur ton signal |
| F-12 | Traiter l'état de repos ou l'absence d'intention comme ne déclenchant aucune commande (Proposé) | ✅ | Règle du repos (D-28) | — |
| F-13 | Présenter une détection comme la reconnaissance d'une intention définie, jamais comme la lecture d'une pensée | ✅ | Libellés de l'interface | — |

#### Décision et sûreté (Core) (spécification 4.5)

| ID | Fonction | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| F-14 | Associer chaque intention à une commande selon une correspondance active ; la correspondance est consultable | ✅ | Logique pure, tests unitaires | — |
| F-15 | Rejeter toute détection dont la confiance est inférieure au seuil | ✅ | Logique pure, tests unitaires | — |
| F-16 | Rejeter toute détection quand le système n'est pas Actif, quand le signal est insuffisant ou quand la cible est indisponible | ✅ | Logique pure, tests unitaires | — |
| F-17 | Imposer un délai minimal entre deux commandes pour éviter les répétitions involontaires | ✅ | Logique pure, tests unitaires | — |
| F-18 | Demander une confirmation avant toute commande sensible ; sans confirmation dans le délai prévu, la commande expire | ✅ | Logique pure, tests unitaires | — |
| F-19 | Permettre de suspendre et de reprendre l'exécution des commandes ; la reprise est toujours explicite | ✅ | Logique pure, tests unitaires | — |
| F-20 | Disposer d'un moyen d'arrêt indépendant de la détection EEG et de l'interface Web | ✅ | Logique pure, tests unitaires | — |
| F-21 | Enregistrer et afficher le motif de chaque rejet | ✅ | Logique pure, tests unitaires | — |
| F-22 | Recevoir le résultat de chaque action ; sans retour dans un délai défini, considérer la commande comme échouée (Proposé) | ✅ | Logique pure, tests unitaires | — |

#### Systèmes cibles (spécification 4.6)

| ID | Fonction | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| F-23 | Connaître l'état de chaque système cible (disponible, indisponible) et transmettre les commandes à au moins un système cible réel | ✅ | Ordinateur réel, ou cible simulée | — |
| F-24 | Limiter chaque système cible à une liste fermée de commandes définies par son connecteur (Proposé) | ✅ | Liste fermée par connecteur | — |
| F-25 | Ajouter un nouveau type de système cible sous forme de connecteur, sans modifier le Core | ✅ | Interface de connecteur | — |
| F-26 | Permettre un test manuel d'un système cible, sans EEG ; ces commandes sont signalées comme manuelles et exclues des mesures | ✅ | Test manuel (D-27) | — |

#### Sessions et mesures (spécification 4.7)

| ID | Fonction | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| F-27 | Créer une session avec son type (utilisation ou expérimentation), son profil, son scénario, ses conditions et des notes ; la source des données et la version du modèle sont enregistrées automatiquement | ✅ | Logiciel uniquement | — |
| F-28 | Démarrer, mettre en pause, reprendre et arrêter une session ; la mise en pause suspend les commandes | ✅ | Logiciel uniquement | — |
| F-29 | Enregistrer le signal, les détections, les décisions, les commandes, les actions, les résultats et leurs horodatages | ✅ | Logiciel uniquement | — |
| F-30 | En session d'expérimentation, enregistrer l'intention attendue à chaque essai | ✅ | Étiquettes du jeu public = intention attendue | — |
| F-31 | Calculer les mesures de la session (voir tableau ci-dessous) | 🟡 | Calcul complet sur données publiques | Mesures officielles (cahier 3.3) |
| F-32 | Distinguer toujours les mesures obtenues sur casque réel de celles obtenues en simulation ou sur enregistrement ; ne jamais les agréger ensemble sans l'indiquer | ✅ | Logiciel uniquement | — |
| F-33 | Comparer plusieurs sessions | ✅ | Logiciel uniquement | — |
| F-34 | Rejouer une session enregistrée | ✅ | Rejeu d'enregistrements publics | — |
| F-35 | Exporter les données et résultats d'une session, avec leurs métadonnées, uniquement si le consentement le permet | ✅ | Logiciel uniquement | — |

#### Journal et alertes (spécification 4.8)

| ID | Fonction | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| F-36 | Tenir un **journal fonctionnel unique** : connexions, qualité du signal, calibrations, détections, décisions et motifs, commandes, actions, résultats, suspensions, reprises, arrêts, événements de session, erreurs, et modifications de paramètres (seuil, correspondance) avec leur auteur | ✅ | Logiciel uniquement | — |
| F-37 | Classer les événements par gravité (information, avertissement, erreur, critique) et émettre une alerte pour ceux qui demandent une réaction | ✅ | Logiciel uniquement | — |
| F-38 | Conserver le journal selon une durée définie | ✅ | Logiciel uniquement | — |

#### Profils, consentement et données (spécification 4.9)

| ID | Fonction | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| F-39 | Gérer un profil : identifiant ou pseudonyme, calibrations, modèles, correspondance intention → commande (si elle est modifiable), préférences d'affichage | ✅ | Logiciel uniquement | — |
| F-40 | Exiger un consentement explicite avant tout enregistrement de données EEG ; préciser sa portée (utilisation, expérimentation, export) ; le consentement est consultable | ✅ | Logiciel uniquement | — |
| F-41 | Permettre de retirer son consentement et de supprimer ses données ; le retrait arrête tout nouvel enregistrement | ✅ | Logiciel uniquement | — |
| F-42 | Limiter l'accès aux données et aux fonctions selon le rôle | ✅ | Logiciel uniquement | — |
| F-43 | Authentifier les personnes qui accèdent à l'interface | ✅ | Logiciel uniquement | — |
| F-44 | Ne collecter que les données nécessaires aux fonctions décrites ici (Proposé) | ✅ | Logiciel uniquement | — |

### 3.2 Fonctionnalités de l'interface Web (FW-01 à FW-50)

#### Supervision (spécification 5.1)

| ID | Fonctionnalité | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| FW-01 | État de la chaîne étape par étape : casque, traitement, détection, Core, système cible | ✅ | Données simulées venant du backend | — |
| FW-02 | Indicateur permanent de la source des données | ✅ | Données simulées venant du backend | — |
| FW-03 | Intention détectée, niveau de confiance et décision (acceptée, rejetée avec motif, en attente) | ✅ | Données simulées venant du backend | — |
| FW-04 | Commande envoyée, action exécutée et résultat | ✅ | Données simulées venant du backend | — |
| FW-05 | État du système cible actif | ✅ | Données simulées venant du backend | — |
| FW-06 | Fil des événements importants récents | ✅ | Données simulées venant du backend | — |
| FW-07 | Indicateurs de la session en cours : durée, détections, rejets, commandes, latence | ✅ | Données simulées venant du backend | — |

#### Casque et signal (spécification 5.2)

| ID | Fonctionnalité | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| FW-08 | Connexion et déconnexion du casque, état de connexion | 🟡 | Avec la carte synthétique | Vrai casque |
| FW-09 | Caractéristiques du casque : modèle, nombre de canaux, fréquence d'échantillonnage | 🟡 | Valeurs de la carte synthétique | Valeurs du vrai casque |
| FW-10 | Qualité globale du signal | 🟡 | Affichage complet, qualité simulée | Qualité réelle |
| FW-11 | Qualité du signal par canal | 🟡 | Affichage par canal simulé | Canaux réels (D-04) |
| FW-12 | Visualisation du signal en temps réel, canaux sélectionnables | ✅ | Signal simulé ou rejoué | — |
| FW-13 | Comparaison du signal brut et du signal filtré | ✅ | Données simulées venant du backend | — |
| FW-14 | Bandes de fréquences et spectre | ✅ | Données simulées venant du backend | — |
| FW-15 | Alertes sur le signal : déconnexion, qualité insuffisante | ✅ | Données simulées venant du backend | — |

#### Calibration (spécification 5.3)

| ID | Fonctionnalité | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| FW-16 | Calibration guidée : consignes, progression | 🟡 | Écrans complets | Vraie calibration |
| FW-17 | État et résultat de la calibration ; interrompre ou recommencer | 🟡 | Écrans complets | Vrai résultat |
| FW-18 | Dernière calibration et modèle actif du profil | 🟡 | Modèle issu du jeu public | Ton modèle |
| FW-19 | Historique et comparaison des calibrations, choix d'une version de modèle | ✅ | Données simulées venant du backend | — |
| FW-47 | Indication de validité de la calibration et recommandation de recalibrer | ✅ | Données simulées venant du backend | — |

#### Commandes et systèmes cibles (spécification 5.4)

| ID | Fonctionnalité | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| FW-20 | Consultation de la correspondance intention → commande active | ✅ | Données simulées venant du backend | — |
| FW-21 | Modification de la correspondance intention → commande | ✅ | Données simulées venant du backend | — |
| FW-22 | Liste des systèmes cibles et de leur état | ✅ | Données simulées venant du backend | — |
| FW-23 | Test manuel d'un système cible, clairement signalé | ✅ | Données simulées venant du backend | — |
| FW-24 | Affichage des types de systèmes cibles selon la trajectoire | ✅ | Données simulées venant du backend | — |

#### Sûreté (spécification 5.5)

| ID | Fonctionnalité | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| FW-25 | État global de CortexOS (section 4.1) toujours visible, en particulier « commandes actives » ou « suspendues » | ✅ | Données simulées venant du backend | — |
| FW-26 | Suspendre et reprendre les commandes | ✅ | Données simulées venant du backend | — |
| FW-27 | Affichage du seuil de confiance ; modification | ✅ | Données simulées venant du backend | — |
| FW-28 | Confirmation d'une commande sensible, avec le délai restant | ✅ | Données simulées venant du backend | — |

#### Sessions et mesures (spécification 5.6)

| ID | Fonctionnalité | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| FW-29 | Création d'une session : type, profil, scénario, conditions, notes | ✅ | Données simulées venant du backend | — |
| FW-30 | Démarrer, mettre en pause, reprendre, arrêter une session | ✅ | Données simulées venant du backend | — |
| FW-31 | Mesures d'une session : précision et niveau du hasard, matrice de confusion, latence, taux de rejet, commandes involontaires | 🟡 | Mesures sur données publiques | Mesures officielles |
| FW-32 | Liste des sessions passées, avec type et source des données | ✅ | Données simulées venant du backend | — |
| FW-33 | Comparaison de sessions | ✅ | Données simulées venant du backend | — |
| FW-34 | Export d'une session | ✅ | Données simulées venant du backend | — |
| FW-35 | Rejeu d'une session enregistrée | ✅ | Données simulées venant du backend | — |

#### Journal et alertes (spécification 5.7)

| ID | Fonctionnalité | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| FW-36 | Journal unique, filtrable par session, type et gravité | ✅ | Données simulées venant du backend | — |
| FW-37 | Messages d'erreur compréhensibles : cause et action possible | ✅ | Données simulées venant du backend | — |
| FW-38 | Vue technique de l'état des services | ✅ | Données simulées venant du backend | — |
| FW-48 | Alertes non visuelles (par exemple sonores) | ✅ | Données simulées venant du backend | — |
| FW-50 | Historique des modifications de paramètres (seuil, correspondance), avec auteur et date | ✅ | Données simulées venant du backend | — |

#### Profil, consentement et données (spécification 5.8)

| ID | Fonctionnalité | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| FW-39 | Choix ou création d'un profil | ✅ | Données simulées venant du backend | — |
| FW-40 | Recueil et consultation du consentement | ✅ | Données simulées venant du backend | — |
| FW-41 | Retrait du consentement et suppression des données | ✅ | Données simulées venant du backend | — |
| FW-42 | Authentification, rôles, administration des comptes | ✅ | Données simulées venant du backend | — |

#### Accessibilité et aide (spécification 5.9)

| ID | Fonctionnalité | Sur PC | Comment, avec le PC seul | Ce qui attend le casque |
|---|---|---|---|---|
| FW-43 | Aide contextuelle | ✅ | Données simulées venant du backend | — |
| FW-44 | Interface conforme aux exigences de la section 6 | ✅ | Données simulées venant du backend | — |
| FW-45 | Vue simplifiée pour l'utilisateur, vue détaillée pour l'expérimentateur | ✅ | Données simulées venant du backend | — |
| FW-46 | Pilotage de l'interface Web elle-même par intentions | 🟡 | Prototype avec détections rejouées (Futur) | Pilotage réel |
| FW-49 | Indicateur de fraîcheur des données affichées (dernière mise à jour, perte de connexion) | ✅ | Données simulées venant du backend | — |

---

## 4. Systèmes cibles : de l'ordinateur au drone

Toute la trajectoire du Cahier des charges peut être développée sur PC, en simulation. L'horizon de chaque système reste celui du Cahier des charges.

| Système cible | Horizon (cahier) | Sur PC seul | Comment | Ce qui manquera |
|---|---|---|---|---|
| **Ordinateur** | MVP ou Ext. `[D-05]` | ✅ **Réel** | Un agent local sur ton PC reçoit les commandes et agit sur le système : déplacer le curseur, sélectionner un élément. Liste fermée de commandes (F-24) ; OS `[D-06]` | Rien |
| **Objets connectés** | MVP ou Ext. `[D-05]` | ✅ Simulé | Une « lampe » simulée qui reçoit les commandes et affiche son état ; même protocole qu'un vrai appareil (MQTT local, optionnel) | L'action physique (ESP32 + lampe) |
| **Systèmes embarqués** | Ext. | 🟡 Simulé | Simulateur de microcontrôleur | La carte réelle |
| **Robot** | Ext. en simulation ou Futur `[D-11]` | ✅ Simulé | ROS 2 : d'abord un robot 2D simple (tortue qui avance, tourne, s'arrête), puis un simulateur 3D si le PC le permet | Le robot réel |
| **Drone** | Futur `[D-11]` | ✅ Simulé | Option simple : un drone simulé en 2D, fait par toi (décoller, monter, descendre, tourner, atterrir, arrêt). Option réaliste : simulateur de vol open source avec un simulateur 3D (plus lourd, Linux conseillé) | Le drone réel (hors MVP) |

Règle du Cahier des charges (section 4.1) : **aucun système plus complexe n'est intégré tant que la chaîne EEG → IA → commande n'a pas été validée sur un système plus simple.** Le robot et le drone simulés viennent donc après la validation de la chaîne sur l'ordinateur ou la lampe simulée.

Chaque nouveau système cible est un nouveau **connecteur** (F-25) : le Core ne change pas.

---

## 5. Ce qui attend le matériel

### 5.1 Uniquement avec le casque

1. Enregistrer **ton propre signal** : placement des électrodes, bruit et artefacts réels.
2. Faire une **vraie calibration**, sur toi ou sur des volontaires.
3. Entraîner le **modèle personnel** de l'utilisateur.
4. Choisir les **intentions définitives** (D-03) : celles qui se distinguent réellement.
5. Obtenir les **mesures officielles** du Cahier des charges (section 3.3) : précision réelle, commandes involontaires réelles, latence complète.
6. Tester la **fatigue et le confort** sur des sessions longues (D-37).
7. Réaliser la **démonstration finale réelle**.

### 5.2 Matériel à prévoir plus tard

| Matériel | Utilité | Nécessaire au MVP ? |
|---|---|---|
| Casque EEG | Tout le point 5.1 | Oui `[D-04]` |
| ESP32 et composants | Action physique sur un objet connecté | Seulement si les objets connectés sont retenus pour le MVP `[D-05]` |
| Robot, drone | Démonstration physique | Non (Ext. ou Futur) `[D-11]` |

---

## 6. Ordre de travail conseillé

Chaque lot produit un résultat visible. L'ordre suit le principe « une chaîne complète d'abord, puis on enrichit ». Les dates relèvent du Planning MVP.

| Lot | Contenu | Réf. | Résultat visible |
|---|---|---|---|
| L1 | Fondations : dépôt Git, structure du projet, environnement Python | — | Projet versionné |
| L2 | **Core** : états de CortexOS, correspondance intention → commande, seuil, rejets, confirmation, suspension, cycle d'une commande | 4.1, F-14 à F-22 | Tests unitaires qui passent |
| L3 | **Source de données** et acquisition simulée, qualité, incidents simulés | F-01 à F-05 | Signal synthétique lu en continu |
| L4 | **Backend** et envoi en temps réel vers l'interface | — | L'état de CortexOS est consultable |
| L5 | **Interface Web** de base : supervision, signal, sûreté, journal, accessibilité | FW-01 à FW-12, FW-15, FW-25 à FW-28, FW-36, FW-37, FW-44, FW-49 | Tableau de bord en direct |
| L6 | **Systèmes cibles** : lampe simulée, puis ordinateur réel via l'agent local | F-23 à F-26, FW-20 à FW-24 | **Chaîne complète en simulation** avec le détecteur de test |
| L7 | **IA hors ligne** sur jeu de données public (ton parcours d'apprentissage, mode A) | F-09, F-11 | Modèle entraîné, précision comparée au hasard |
| L8 | **Intégration de l'IA** : rejeu d'enregistrements publics → modèle → Core → cible | F-11, F-34 | Vraies détections d'un modèle (sur données publiques) |
| L9 | **Sessions, mesures, calibration (écrans), profils, consentement, données** | F-06 à F-10, F-27 à F-44, FW-16 à FW-19, FW-29 à FW-42, FW-47, FW-50 | Session mesurée et exportée |
| L10 | **Extensions en simulation** : robot ROS 2, puis drone simulé | F-25, `[D-11]` | Robot, puis drone, pilotés par la chaîne |

À la fin du lot L9, tout ce qui ne dépend pas du casque est prêt. L'arrivée du casque ajoute une nouvelle source de données (F-03), puis les étapes de la section 5.1.

---

## 7. Règles à respecter pendant le développement sans matériel

1. **Toujours afficher la source des données** (FW-02) : simulation, enregistrement public ou casque.
2. **Ne jamais présenter un résultat obtenu en simulation ou sur données publiques comme un résultat de CortexOS avec un vrai casque** (F-32). Dans le rapport, ces résultats sont présentés comme des validations de la méthode.
3. **Appliquer les règles de sûreté même en simulation** : c'est en simulation qu'on les teste.
4. **Le détecteur de test sert au développement uniquement** : il est retiré des démonstrations ou clairement signalé.
5. **Coder contre des interfaces** (source de données, connecteur) pour que le casque et le matériel réel se branchent sans réécrire le reste.
6. **Ne pas commencer le robot ou le drone** avant d'avoir validé la chaîne complète (section 4).

---

## 8. Outils à installer

Uniquement des outils déjà prévus par le Cahier des charges, installés au fur et à mesure des lots.

| Lot | Outils | Rôle |
|---|---|---|
| L1 | Git, Python et environnement virtuel | Versionner, isoler les dépendances |
| L2 | Outil de tests Python (par exemple pytest) | Tester le Core |
| L3 | BrainFlow | Carte synthétique, puis casque réel |
| L4 | FastAPI | Backend et temps réel |
| L5 | Node.js, Next.js (TypeScript) | Interface Web |
| L6 | Broker MQTT local (optionnel) | Communiquer avec la lampe simulée comme avec un vrai ESP32 |
| L7 | MNE-Python, scikit-learn, NumPy | Données EEG publiques, traitement, IA |
| L9 | Base de données `[D-09]` | Profils, sessions, journal |
| L10 | ROS 2 (Linux conseillé), simulateur 3D éventuel | Robot, puis drone simulés |

La configuration de ton PC (processeur, mémoire, système) détermine le confort pour l'IA et surtout pour les simulateurs 3D du lot L10 `[À COMPLÉTER]`.

---

## Annexe — Décisions concernées

| ID | Question | Effet sur ce document |
|---|---|---|
| D-03 | Intentions du MVP | Choix définitif après les essais avec le casque |
| D-04 | Casque | Section 5.2, source « casque » |
| D-05 | Système cible du MVP | Ordinateur réel ou lampe simulée puis ESP32 |
| D-06 | Système d'exploitation | Agent local de l'ordinateur |
| D-09 | Stockage, authentification | Lot L9 |
| D-11 | Robot et drone | Lot L10 |
| D-21 | Stratégie de repli | Retenue ici pour la phase sans matériel (Proposé) : à confirmer dans le Cahier des charges |
| D-22 | Planning existant | À confirmer : ce document peut servir de base au Planning MVP |
