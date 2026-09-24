# Planning MVP — CortexOS IA

| | |
|---|---|
| **Document** | Planning MVP détaillé par fonctionnalité |
| **Projet** | CortexOS IA |
| **Documents de référence** | Cahier des charges v2.0 · Spécification fonctionnelle v1.0 · Développement sur PC seul v1.0 |
| **Remplace** | Planning MVP v2.0 et « Planning complet MVP CortexOS AI — 1 an jusqu'à la soutenance » |
| **Suivi** | Classeur « Suivi_CortexOS_IA.xlsx » : une ligne par tâche de ce planning |
| **Période** | 28 septembre 2026 → soutenance vers juillet-août 2027 `[À DÉFINIR — D-14]` |
| **Auteur** | GOMINA Eloge |
| **Version du document** | 3.0 — 23 septembre 2026 |

> Situation de départ : aucun code écrit, casque choisi mais pas commandé, un PC seul, environ 8 heures de travail par semaine, développement avec Claude (explications, revue) et Claude Code (écriture de code), la compréhension restant la priorité.

---

## Sommaire

1. Ce qui change dans cette version
2. Hypothèses et choix retenus
3. Méthode de travail
4. Vue d'ensemble : tranches et jalons
5. Planning détaillé par fonctionnalité
6. Le bloc « casque »
7. Diagrammes et documents de conception
8. Suivi de l'avancement
9. Risques et plans de repli
10. Extensions : hors du planning MVP

Annexe — Décisions et échéances

---

## 1. Ce qui change dans cette version

| Version 2.0 | Version 3.0 | Raison |
|---|---|---|
| Documents d'analyse répartis au fil du projet | **Phase de conception de 3 semaines** au départ : 8 diagrammes, Architecture technique v0, contrat d'API, maquettes | Tu veux concevoir avant de coder ; ces diagrammes décrivent le système global et changent peu |
| Développement par couches (Core, puis backend, puis Web) | **Développement par tranches** : chaque fonctionnalité va du Core jusqu'à l'écran, avec de vraies données | Quelque chose fonctionne réellement à la fin de chaque tranche ; les problèmes d'intégration apparaissent tôt |
| Architecture non précisée | **Monolithe modulaire** : un seul backend découpé en modules à interfaces claires ; l'agent ordinateur reste un programme séparé | Les microservices ajoutent latence et complexité, inadaptées à une personne et à une chaîne temps réel |
| Planning par lots | **Planning par fonctionnalité**, avec toutes les tâches, leur semaine et leur mode | Chaque tâche est suivie dans le classeur de suivi |
| — | **Classeur de suivi** généré à partir du planning | Tu le remplis au fil du temps |

Ce qui est conservé : départ le 28 septembre 2026, 8 h par semaine, casque à commander en semaine 1, bloc casque de 4 semaines inséré dès la réception, simulation BrainFlow et jeu de données public, mode A / mode B, gel des fonctionnalités le 9 mai 2027, marge finale et vidéo de secours.

---

## 2. Hypothèses et choix retenus

| Élément | Hypothèse ou choix |
|---|---|
| Temps disponible | Environ 8 h par semaine ; périodes d'examens `[À COMPLÉTER]` |
| Matériel | Un PC, configuration `[À COMPLÉTER]` |
| Casque | Modèle choisi `[À COMPLÉTER — D-04]`, commande en S1, délai de livraison `[À DÉFINIR]` |
| Architecture | Monolithe modulaire ; agent ordinateur séparé |
| Méthode | Contrat d'API défini pendant la conception, puis développement par tranches |
| Système cible réel du MVP | L'ordinateur, avec une lampe simulée en complément `[À CONFIRMER — D-05]` |
| Langage du Core pour le MVP | Python `[À CONFIRMER — D-07]` |
| Stratégie sans matériel | Simulation BrainFlow et jeu de données public `[À CONFIRMER — D-21]` |
| Diagrammes | En Mermaid, versionnés dans `docs/` ; autre format si l'école l'exige `[D-02]` |

Répartition d'une semaine type de 8 h :

| Activité | Temps |
|---|---|
| Tâches de la tranche en cours | 5 h |
| Piste IA (jusqu'en janvier) | 2 h |
| Tests, commit, mise à jour du suivi et de la documentation | 1 h |

---

## 3. Méthode de travail

### 3.1 Deux modes

| Mode | Qui écrit | Pour quoi |
|---|---|---|
| **A — Tu codes** | Toi, guidé étape par étape ; Claude relit et explique | Ce que tu dois savoir expliquer en soutenance : Core, IA, source de données, mesures, consentement, diagrammes déduits de la Spécification |
| **B — Claude Code code** | Claude Code ; tu relis chaque fichier avec le compte rendu pédagogique | Squelettes, backend, écrans Web, diagrammes d'architecture (contexte, composants, déploiement) |

**Règle** : aucun code n'entre dans le projet si tu ne peux pas l'expliquer.

### 3.2 Une tranche est terminée quand…

1. le diagramme de conception nécessaire existe (s'il y en a un) ;
2. le code du module est écrit et **testé** ;
3. la route d'API respecte le **contrat d'API** ;
4. l'écran affiche les **vraies données** du backend ;
5. le travail est **commité** avec un message clair ;
6. le **classeur de suivi** est à jour ;
7. tu as répondu à la **question de compréhension**.

### 3.3 Déroulement d'une séance

Préparer (tâche et concept, 10 min) → coder (mode A ou B) → tester → commiter → vérifier ta compréhension → mettre à jour le suivi.

---

## 4. Vue d'ensemble : tranches et jalons

| Tranche | Semaines | Dates | Contenu |
|---|---|---|---|
| 0 — Démarrage et conception | S1–S4 | 28/09 → 25/10/2026 | Casque, environnement, 8 diagrammes, Architecture technique v0, contrat d'API, maquettes, squelette |
| Piste IA (en parallèle) | S1–S16 | 28/09/2026 → 17/01/2027 | Apprentissage et code de l'IA sur données publiques |
| 1 — États et contrôle humain | S5–S6 | 26/10 → 08/11/2026 | États de CortexOS, activer / suspendre / reprendre |
| 2 — Décision et sûreté | S7–S9 | 09/11 → 29/11/2026 | Détection, seuil, rejets, cycle d'une commande, arrêt, journal |
| 3 — Source et temps réel | S10–S13 | 30/11 → 27/12/2026 | Source simulée, qualité, incidents, WebSocket, écrans du signal et de supervision |
| Marge (fêtes) | S14 | 28/12/2026 → 03/01/2027 | — |
| 4 — Systèmes cibles | S15–S19 | 04/01 → 07/02/2027 | Lampe simulée, agent ordinateur, chaîne complète en simulation |
| 5 — IA intégrée | S20–S21 | 08/02 → 21/02/2027 | Modèle branché sur le Core, rejeu de données publiques |
| 6 — Sessions et mesures | S22–S24 | 22/02 → 14/03/2027 | Base de données, sessions, enregistrement, mesures |
| 7 — Profils et données | S25 | 15/03 → 21/03/2027 | Profils, consentement, suppression, export, authentification minimale |
| 8 — Calibration | S26 | 22/03 → 28/03/2027 | Parcours et écrans de calibration |
| Bloc casque | 4 semaines dès réception | S27–S30 si possible | Signal réel, intentions, modèle personnel |
| Évaluation | S31–S32 | 26/04 → 09/05/2027 | Campagne de mesures, gel des fonctionnalités |
| Rapport et soutenance | S33–S44 | 10/05 → 01/08/2027 | Rapport, relecture, répétitions, vidéo de secours, marge |

| Jalon | Résultat attendu | Date cible |
|---|---|---|
| — | Casque commandé | 04/10/2026 |
| J1 · J2 | Cahier des charges validé, architecture de principe et diagrammes de conception | 18/10/2026 |
| — | Squelette : frontend ↔ backend | 25/10/2026 |
| — | Core testé (décision et sûreté) | 29/11/2026 |
| J3 | Acquisition fonctionnelle (simulation) | 20/12/2026 |
| J6 | Supervision opérationnelle | 10/01/2027 |
| J5 | Chaîne complète en simulation : commande réelle sur l'ordinateur avec garde-fous | 07/02/2027 |
| J4 | Première détection mesurée (données publiques) | 14/03/2027 |
| — | Tout ce qui ne dépend pas du casque est prêt | 28/03/2027 |
| J7 | Campagne d'évaluation terminée, gel des fonctionnalités | 09/05/2027 |
| J8 | Soutenance | `[D-14]` |

---

## 5. Planning détaillé par fonctionnalité

Chaque ligne correspond à une fonctionnalité ; ses tâches sont reprises une par une dans le classeur de suivi. Les références (F-xx, FW-xx) sont celles de la Spécification fonctionnelle : **les 44 fonctions et les 50 fonctionnalités Web y figurent toutes**, dans ce planning ou dans les extensions (section 10).

### Tranche 0 — Démarrage et conception (S1–S4)

| Sem. | Dates | Réf. | Fonctionnalité | Tâches | Mode |
|---|---|---|---|---|---|
| S1 | 28/09 → 04/10 | — | **Commande du casque EEG** | Confirmer le modèle (D-04) · Passer la commande · Noter le délai de livraison annoncé | — |
| S1 | 28/09 → 04/10 | — | **Environnement de travail** | Installer Git, Python 3, Node.js LTS, VS Code · Créer le dépôt CortexOS et le premier commit · Créer docs/ avec Cahier des charges, Spécification, Planning | A |
| S1 | 28/09 → 04/10 | DIAG-1 | **Diagramme de contexte** | Lister les acteurs et systèmes externes · Dessiner en Mermaid · Relire et expliquer | B |
| S1 | 28/09 → 04/10 | DIAG-2 | **Diagrammes de cas d'utilisation** | Lister les cas par acteur (parcours A à E) · Découper en 3 diagrammes : Préparer, Utiliser, Mesurer · Relecture avec Claude | A |
| S2 | 05/10 → 11/10 | DIAG-3 | **Diagramme de classes de domaine** | Reprendre les classes de la Spécification · Définir les relations et multiplicités · Relecture avec Claude | A |
| S2 | 05/10 → 11/10 | DIAG-4 | **Diagramme de composants** | Modules du monolithe modulaire et leurs interfaces · Place de l'agent ordinateur · Explication et relecture | B |
| S2 | 05/10 → 11/10 | DIAG-5 | **Diagrammes d'états-transitions** | États globaux de CortexOS (spéc. 4.1) · Cycle de vie d'une commande (spéc. 4.5) · Relecture avec Claude | A |
| S2 | 05/10 → 11/10 | ARCH-0 | **Architecture technique v0** | Rédiger à partir des diagrammes 1 à 5 · Trancher D-05, D-07 et D-09 (version minimale) | B |
| S3 | 12/10 → 18/10 | DIAG-6 | **Diagrammes de séquence** | (a) Intention acceptée → action · (b) Rejet pour confiance insuffisante · (c) Commande sensible : confirmation ou expiration · (d) Perte du signal → état sûr | A |
| S3 | 12/10 → 18/10 | DIAG-7 | **Diagramme d'activité** | Déroulement d'une session d'expérimentation (parcours C) | A |
| S3 | 12/10 → 18/10 | DIAG-8 | **Diagramme de déploiement** | Ce qui tourne sur le PC maintenant · Ajouts futurs : casque, ESP32 | B |
| S3 | 12/10 → 18/10 | API-0 | **Contrat d'API v0** | Routes REST et messages temps réel déduits des FW-01 à FW-50 · Format des objets : détection, décision, commande, événement | B |
| S3 | 12/10 → 18/10 | MAQ | **Maquettes des écrans** | Maquettes basse fidélité : supervision, signal, calibration, commandes, sessions, journal, profil | A/B |
| S4 | 19/10 → 25/10 | SQL | **Squelette du projet** | Package core/ en Python pur, avec tests · Backend FastAPI : GET /api/health et son test · Frontend Next.js qui affiche le statut · README : installation et lancement | B |

### Piste IA — en parallèle, environ 2 h par semaine (S1–S16)

| Sem. | Dates | Réf. | Fonctionnalité | Tâches | Mode |
|---|---|---|---|---|---|
| S1 | 28/09 → 04/10 | IA | **Piste IA** | Étape 1 : charger un enregistrement public, explorer canaux, fréquence, annotations | A |
| S2 | 05/10 → 11/10 | IA | **Piste IA** | Étape 1 (fin) : tracé du signal, signification de T0, T1, T2 | A |
| S3 | 12/10 → 18/10 | IA | **Piste IA** | Étape 2 : découper en essais (epochs) avec leurs étiquettes | A |
| S4 | 19/10 → 25/10 | IA | **Piste IA** | Étape 3 : filtrage | A |
| S5 | 26/10 → 01/11 | IA | **Piste IA** | Révision | A |
| S6 | 02/11 → 08/11 | IA | **Piste IA** | Étape 4 : caractéristiques (CSP) | A |
| S7 | 09/11 → 15/11 | IA | **Piste IA** | Étape 4 (fin) | A |
| S8 | 16/11 → 22/11 | IA | **Piste IA** | Étape 5 : classifieur (LDA ou SVM) | A |
| S9 | 23/11 → 29/11 | IA | **Piste IA** | Étape 5 (fin) | A |
| S10 | 30/11 → 06/12 | IA | **Piste IA** | Étape 6 : validation croisée, comparaison au hasard | A |
| S11 | 07/12 → 13/12 | IA | **Piste IA** | Étape 6 (fin) | A |
| S12 | 14/12 → 20/12 | IA | **Piste IA** | Étape 7 : prédiction avec niveau de confiance | A |
| S15 | 04/01 → 10/01 | IA | **Piste IA** | Passer du notebook à un module propre | A |
| S16 | 11/01 → 17/01 | IA | **Piste IA** | Tests du module IA ; sauvegarde et rechargement du modèle | A |

### Tranche 1 — États et contrôle humain (S5–S6)

| Sem. | Dates | Réf. | Fonctionnalité | Tâches | Mode |
|---|---|---|---|---|---|
| S5 | 26/10 → 01/11 | DIAG-9 | **Classes de conception du Core** | Classes, attributs typés, méthodes · Relecture avec Claude | A |
| S5 | 26/10 → 01/11 | 4.1 | **États de CortexOS** | Modéliser les 7 états · Coder les transitions autorisées · Refuser les transitions interdites · Tests unitaires | A |
| S6 | 02/11 → 08/11 | F-19 · FW-25 · FW-26 | **Activer, suspendre, reprendre les commandes** | Méthodes du Core · Routes API · Indicateur d'état toujours visible · Boutons Activer / Suspendre / Reprendre · Tests | A (Core) · B (API, écran) |

### Tranche 2 — Décision et sûreté (S7–S9)

| Sem. | Dates | Réf. | Fonctionnalité | Tâches | Mode |
|---|---|---|---|---|---|
| S7 | 09/11 → 15/11 | F-11 · F-12 · F-13 | **Détection et détecteur de test** | Objet Détection : intention, confiance, horodatage · Règle du repos (D-28) · Détecteur de test clairement signalé comme simulé | A |
| S7 | 09/11 → 15/11 | F-15 · F-16 · F-21 | **Seuil et rejets avec motif** | Seuil de confiance configurable · Rejet si non Actif, signal insuffisant ou cible indisponible · Motif de rejet enregistré · Tests | A |
| S8 | 16/11 → 22/11 | F-14 · F-18 · F-22 | **Cycle de vie d'une commande** | Correspondance intention → commande · États de la commande · Confirmation et expiration (D-24, D-31) · Résultat, échec après délai · Tests | A |
| S8 | 16/11 → 22/11 | F-17 | **Délai anti-répétition** | Délai minimal entre deux commandes (si D-30 retenue) · Tests | A |
| S9 | 23/11 → 29/11 | F-20 | **Arrêt indépendant de l'EEG** | Mécanisme selon D-19 (à décider avant S9) · Tests : l'arrêt fonctionne même si l'interface est fermée | A |
| S9 | 23/11 → 29/11 | F-36 · F-37 | **Journal et gravité** | Événement de journal · Niveaux de gravité · Route de lecture filtrée | A/B |
| S9 | 23/11 → 29/11 | FW-03 · FW-04 · FW-06 · FW-27 · FW-28 · FW-36 · FW-37 | **Écrans de décision et de journal** | Détection, décision et motif · Commande et résultat · Fil d'événements · Seuil affiché · Fenêtre de confirmation avec délai restant · Journal filtrable · Messages d'erreur compréhensibles | B |

### Tranche 3 — Source de données et temps réel (S10–S13)

| Sem. | Dates | Réf. | Fonctionnalité | Tâches | Mode |
|---|---|---|---|---|---|
| S10 | 30/11 → 06/12 | DIAG-10 | **Classes de l'interface SourceEEG** | Interface et implémentations · Relecture avec Claude | A |
| S10 | 30/11 → 06/12 | F-01 · F-03 | **Source de données simulée** | Interface SourceEEG : démarrer, lire, arrêter · Implémentation « carte synthétique » BrainFlow · Caractéristiques : canaux, fréquence · Tests | A |
| S11 | 07/12 → 13/12 | F-02 · F-04 · F-05 | **Qualité du signal et incidents** | Indicateur de qualité · Qualité insuffisante → aucune commande · Déconnexion → état sûr · Simulation d'incidents · Tests | A |
| S11 | 07/12 → 13/12 | DIAG-11 | **Temps réel backend → frontend** | Diagramme de séquence temps réel · Canal WebSocket · Test | B |
| S12 | 14/12 → 20/12 | FW-02 · FW-08 à FW-12 · FW-15 · FW-49 | **Écrans du signal** | Source des données affichée · Connexion du casque · Caractéristiques · Qualité globale et par canal · Courbes en temps réel · Alertes sur le signal · Fraîcheur des données | B |
| S13 | 21/12 → 27/12 | FW-01 · FW-05 · FW-07 · FW-44 | **Supervision et accessibilité de base** | Vue de la chaîne étape par étape · État du système cible · Indicateurs de la session en cours · Accessibilité : clavier, contrastes, libellés | B |

### Tranche 4 — Systèmes cibles et chaîne complète (S15–S19)

| Sem. | Dates | Réf. | Fonctionnalité | Tâches | Mode |
|---|---|---|---|---|---|
| S15 | 04/01 → 10/01 | DIAG-12 | **Classes de l'interface Connecteur** | Interface et implémentations · Relecture avec Claude | A |
| S15 | 04/01 → 10/01 | F-23 · F-24 · F-25 | **Lampe simulée** | Interface Connecteur · Lampe simulée · Liste fermée de commandes · Disponibilité de la cible · Tests | A |
| S16 | 11/01 → 17/01 | F-23 | **Agent ordinateur** | Programme agent séparé (D-06) · Commandes : curseur, sélection (liste fermée) · Liaison backend ↔ agent · Tests manuels | A/B |
| S17 | 18/01 → 24/01 | F-26 · FW-20 à FW-24 | **Écrans des commandes et systèmes cibles** | Correspondance consultable (modifiable si D-20) · Liste des cibles et leur état · Test manuel si D-27 · Types de systèmes cibles | B |
| S18 | 25/01 → 31/01 | — | **Intégration de la chaîne** | Détecteur de test → Core → lampe · Détecteur de test → Core → ordinateur · Tests d'intégration | A |
| S19 | 01/02 → 07/02 | — | **Démonstration interne** | Scénario de bout en bout en simulation · Corrections | A |

### Tranche 5 — IA intégrée (S20–S21)

| Sem. | Dates | Réf. | Fonctionnalité | Tâches | Mode |
|---|---|---|---|---|---|
| S20 | 08/02 → 14/02 | F-11 · F-34 | **Détection par le modèle** | Source « enregistrement public » · Module de détection branché sur le Core · Rejeu d'un enregistrement | A |
| S21 | 15/02 → 21/02 | FW-18 · FW-02 | **Modèle actif et source** | Modèle actif affiché · Vérifier l'affichage de la source · Tests de bout en bout | A/B |

### Tranche 6 — Sessions et mesures (S22–S24)

| Sem. | Dates | Réf. | Fonctionnalité | Tâches | Mode |
|---|---|---|---|---|---|
| S22 | 22/02 → 28/02 | DIAG-13 · F-27 · F-28 | **Modèle de données et sessions** | MCD / MLD (D-09) · Mise en place du stockage · Créer, démarrer, mettre en pause, reprendre, arrêter une session | A/B |
| S23 | 01/03 → 07/03 | F-29 · F-30 · FW-29 · FW-30 · FW-32 | **Enregistrement des sessions** | Enregistrer signal, détections, décisions, commandes · Intention attendue par essai · Écrans : création, conduite, liste des sessions | A/B |
| S24 | 08/03 → 14/03 | F-31 · F-32 · FW-31 | **Mesures** | Précision et niveau du hasard · Matrice de confusion · Latence de bout en bout · Taux de rejet, commandes involontaires · Séparation réel / simulé · Écran des mesures | A |

### Tranche 7 — Profils, consentement et données (S25)

| Sem. | Dates | Réf. | Fonctionnalité | Tâches | Mode |
|---|---|---|---|---|---|
| S25 | 15/03 → 21/03 | F-35 · F-38 à F-44 · FW-34 · FW-39 à FW-42 · FW-50 | **Profils, consentement, données** | Profil · Consentement obligatoire avant enregistrement · Retrait et suppression (D-38) · Export (D-34) · Conservation du journal (D-09) · Authentification minimale (D-09) · Historique des modifications de paramètres | A (consentement) · B |

### Tranche 8 — Calibration (S26)

| Sem. | Dates | Réf. | Fonctionnalité | Tâches | Mode |
|---|---|---|---|---|---|
| S26 | 22/03 → 28/03 | DIAG-14 · F-06 à F-09 · FW-16 à FW-18 | **Calibration** | Diagramme de séquence de la calibration · Consignes et progression · Refus si qualité insuffisante · Interrompre, recommencer · Résultat exploitable, version du modèle · Écrans | A/B |

### Bloc casque (4 semaines dès réception) (S27–S30 si possible)

| Sem. | Dates | Réf. | Fonctionnalité | Tâches | Mode |
|---|---|---|---|---|---|
| S27 | 29/03 → 04/04 | F-01 · F-02 | **C1 — Source « casque »** | Connexion réelle, stabilité · Qualité réelle du signal · Réglage des seuils de qualité | A |
| S28 | 05/04 → 11/04 | F-06 à F-09 · D-03 | **C2 — Calibrations sur toi** | Premières calibrations · Comparer les intentions candidates · Choisir les intentions (D-03) | A |
| S29 | 12/04 → 18/04 | F-09 · F-15 | **C3 — Modèle personnel** | Réentraîner sur tes données · Régler le seuil de confiance (D-10) | A |
| S30 | 19/04 → 25/04 | F-31 | **C4 — Chaîne sur ton signal** | Chaîne complète sur signal réel · Premières mesures réelles | A |

### Évaluation (S31–S32)

| Sem. | Dates | Réf. | Fonctionnalité | Tâches | Mode |
|---|---|---|---|---|---|
| S31 | 26/04 → 02/05 | — | **Campagne d'évaluation** | Protocole de la campagne · Sessions avec volontaires (D-12) · Mesures officielles, latence complète | A |
| S32 | 03/05 → 09/05 | — | **Analyse et gel** | Analyse des résultats · Gel des fonctionnalités | A |

### Rapport et soutenance (S33–S44)

| Sem. | Dates | Réf. | Fonctionnalité | Tâches | Mode |
|---|---|---|---|---|---|
| S35 | 24/05 → 30/05 | — | **Rapport final** | Rédaction · Mise à jour des diagrammes, de l'Architecture technique et de la Documentation technique selon le code réel | — |
| S38 | 14/06 → 20/06 | — | **Relecture** | Présentation à l'encadreur · Corrections | — |
| S41 | 05/07 → 11/07 | — | **Préparation de la soutenance** | Slides · Répétitions de la démonstration · Vidéo de secours | — |
| S44 | 26/07 → 01/08 | — | **Marge finale** | Ne pas la remplir à l'avance | — |

---

## 6. Le bloc « casque »

Le bloc dure **4 semaines** et commence **dès la réception** du casque, quelle que soit la tranche en cours. Comme la source de données est interchangeable, la tranche en cours est interrompue puis reprise. Son contenu figure dans le tableau « Bloc casque » de la section 5.

| Si le casque arrive… | Conséquence |
|---|---|
| Avant le 8 février 2027 | Le bloc s'insère dans les tranches 4 à 6 ; la fin du planning est tenue avec de la marge |
| Entre le 8 février et le 29 mars 2027 | Le bloc prend place en S27–S30 ; planning tenu |
| Après le 29 mars 2027 | **Plan B** (section 9), à discuter avec l'encadreur |

---

## 7. Diagrammes et documents de conception

| Réf. | Document | Quand | Mode |
|---|---|---|---|
| DIAG-1 | Contexte | S1 | B |
| DIAG-2 | Cas d'utilisation (3 diagrammes) | S1 | A |
| DIAG-3 | Classes de domaine | S2 | A |
| DIAG-4 | Composants | S2 | B |
| DIAG-5 | États-transitions (2 diagrammes) | S2 | A |
| ARCH-0 | Architecture technique v0 | S2 | B |
| DIAG-6 | Séquences (4 scénarios) | S3 | A |
| DIAG-7 | Activité | S3 | A |
| DIAG-8 | Déploiement | S3 | B |
| API-0 | Contrat d'API v0 | S3 | B |
| MAQ | Maquettes des écrans | S3 | A/B |
| DIAG-9 | Classes de conception du Core | S5, avant le Core | A |
| DIAG-10 | Classes de l'interface SourceEEG | S10 | A |
| DIAG-11 | Séquence temps réel | S11 | B |
| DIAG-12 | Classes de l'interface Connecteur | S15 | A |
| DIAG-13 | Modèle de données | S22 | A |
| DIAG-14 | Séquence de la calibration | S26 | A |
| — | Robot et drone | Seulement si l'extension est réalisée | — |

Tous les diagrammes sont mis à jour en S33–S35 pour correspondre au code réellement écrit.

---

## 8. Suivi de l'avancement

Le classeur **Suivi_CortexOS_IA.xlsx** contient :

- **Suivi** : une ligne par tâche de ce planning (187 tâches), avec la tranche, la référence, la fonctionnalité, la tâche, la semaine, la date prévue, la **date réalisée**, le **statut** (À faire, En cours, Terminé, Bloqué), le mode et des **notes** ;
- **Avancement** : le pourcentage de tâches terminées par tranche, calculé automatiquement ;
- **Jalons** : les grandes étapes, avec leur date atteinte.

Tu remplis uniquement les cellules jaunes : date réalisée, statut, notes. **Une fois par semaine**, compare la date prévue et la date réalisée : deux tâches en retard dans la même tranche sont le signal d'alerte de la section 9.

---

## 9. Risques et plans de repli

| Risque | Signal d'alerte | Plan de repli |
|---|---|---|
| **Casque livré tard** | Pas de casque au 29 mars 2027 | Démonstration sur données publiques et simulation, clairement présentées comme telles ; courte démonstration réelle si le casque arrive avant la soutenance ; en parler tôt à l'encadreur |
| **Conception trop longue** | Diagrammes non terminés au 18 octobre | Terminer les diagrammes 1 à 5 et le contrat d'API ; reporter les autres juste avant leur tranche |
| **Précision faible sur ton signal** | Précision proche du hasard dans le bloc casque | Réduire à 2 intentions ; allonger la calibration ; documenter le résultat honnêtement |
| **Moins de 8 h par semaine** | Deux tâches en retard dans une tranche | Retirer d'abord les extensions, puis les écrans non indispensables ; ne jamais retirer la sûreté ni les mesures |
| **Code produit sans être compris** | Tu ne sais pas expliquer un fichier | Arrêter, faire l'explication ligne par ligne, repasser en mode A |
| **Examens** | Périodes chargées | Me les indiquer : les semaines concernées deviennent des semaines de marge |
| **Problème le jour de la soutenance** | — | Vidéo de secours de la démonstration |

---

## 10. Extensions : hors du planning MVP

À réaliser seulement si l'avance le permet (au moins 2 semaines d'avance au 28 mars 2027), sinon après la soutenance.

| Réf. | Élément |
|---|---|
| FW-13 | Comparaison signal brut / filtré |
| FW-14 | Bandes de fréquences et spectre |
| F-10 · FW-19 · FW-47 | Historique des calibrations, validité, recalibration |
| F-33 · FW-33 | Comparaison de sessions |
| FW-35 | Rejeu d'une session dans l'interface |
| FW-38 | Vue technique des services |
| FW-43 | Aide contextuelle |
| FW-45 | Vues simplifiée / détaillée (D-18) |
| FW-48 | Alertes sonores (D-35) |
| FW-46 | Pilotage de l'interface par intentions (Futur) |
| Robot, drone | En simulation, après validation de la chaîne (D-11) |

Restent hors projet pour l'instant : deep learning (CNN, LSTM), Core en C++ (D-07), Rust, contrôle d'applications précises (Cahier des charges 4.6), ESP32 physique (option si achat, D-05), Docker (D-08, utile en fin de projet pour l'installation).

---

## Annexe — Décisions et échéances

Les échéances de chaque décision sont dans le registre [05-decisions.md](05-decisions.md) (colonne « À décider avant »).
