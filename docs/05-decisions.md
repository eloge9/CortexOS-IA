# Décisions — CortexOS IA

Registre unique des décisions du projet. Les autres documents renvoient ici avec `[À DÉFINIR — D-xx]`.
Quand une décision est prise : la déplacer de « Questions ouvertes » vers « Décisions prises », avec la date.

## 1. Décisions prises

| ID | Décision | Date |
|---|---|---|
| D-01 | Vision unifiée à trois dimensions : accessibilité, nouvelle forme d'interaction homme-machine, plateforme expérimentale (cahier des charges, section 1.6) | 23/09/2026 |
| D-16 | Pas de document dédié au contenu retiré : il est repris dans les documents compagnons | 23/09/2026 |
| D-17 | Les trois dimensions sont des objectifs à part entière, avec des rôles complémentaires (cahier des charges, section 1.6) | 23/09/2026 |
| D-22 | Le Planning MVP v3.0 (`03-planning-mvp.md`) est le planning de référence | 23/09/2026 |
| D-39 | Diagrammes en Mermaid dans `docs/diagrammes/` ; draw.io seulement à la fin, pour le rapport et la soutenance | 24/09/2026 |
| D-40 | `docs/` (dépôt Git) est la documentation de référence ; Google Drive n'en est qu'une copie de lecture. Petits changements de doc faits sans demander et signalés ; gros changements validés avant | 24/09/2026 |
| D-41 | Structure de `docs/` numérotée ; un fichier n'est créé que lorsque le travail produit son contenu | 24/09/2026 |
| D-42 | Graphie officielle : « CortexOS IA » (celle du logo) | 24/09/2026 |
| D-43 | Charte graphique : variables CSS + Tailwind v4 ; thème sombre par défaut + thème clair ; polices Exo 2 (titres) + Inter (texte) ; ambiance pro et sobre | 24/09/2026 |
| D-45 | Synchronisation docs / Git / Drive : pendant la séance, seuls les fichiers de `docs/` sont mis à jour (économie de tokens) ; mot-clé « ds » (début de séance) = vérifier les fichiers Drive modifiés et les commentaires ; mot-clé « fs » (fin de séance) = vérifier que les copies Drive n'ont pas été modifiées par Eloge ou l'encadreur (ne jamais écraser : montrer, puis reporter dans `docs/` après accord), republier sur Drive les docs modifiés (ancienne version dans « 99 - Archives »), mettre à jour le classeur de suivi, récapitulatif et commit proposé ; l'encadreur commente plutôt que de modifier (sauf le classeur de suivi, qui n'existe que sur Drive) | 24/09/2026 |
| D-46 | Tous les diagrammes (DIAG-1 à DIAG-14) passent en mode B : Claude les écrit ; Eloge les relit, doit pouvoir les expliquer en soutenance et répond à une question de compréhension après chacun | 25/09/2026 |
| D-47 | Rôles retenus : Utilisateur, Accompagnant / opérateur, Expérimentateur, **Administrateur** (gère les comptes et les rôles) ; une même personne peut cumuler plusieurs rôles. Restent ouverts dans D-09 : mécanisme d'authentification, droits détaillés par rôle, stockage, conservation, partage | 25/09/2026 |
| D-48 | Diagrammes de cas d'utilisation : acteurs humains à gauche, acteurs non humains à droite ; deux fichiers — un diagramme par acteur principal (`diag-02a`) et une vue d'ensemble dans un seul cadre avec toutes les généralisations (`diag-02b`) | 25/09/2026 |
| D-49 | Le catalogue des cas d'utilisation (UC-01 à UC-47) et l'analyse des classes du domaine fournis par Eloge sont les références de DIAG-2 et DIAG-3 ; leurs propositions ont été validées par D-50 à D-53 ; les classes « à confirmer » dépendent des décisions ouvertes indiquées | 25/09/2026 |
| D-50 | Session : simple attribut `type` (utilisation / expérimentation), pas de sous-classes | 25/09/2026 |
| D-51 | Une seule classe **Essai**, liée soit à une calibration, soit à une session (`{xor}`) | 25/09/2026 |
| D-52 | Pas de classe **Incident** dans le MVP : Alerte + événements du journal suffisent | 25/09/2026 |
| D-53 | Un **profil peut exister sans compte** (participant créé par l'expérimentateur) : Personne `0..1` — `0..1` Profil | 25/09/2026 |
| D-54 | Composant **Orchestrateur de la chaîne** dans le backend, distinct du Core : il enchaîne source → qualité → prétraitement → détection → Core et enregistre dans la session | 25/09/2026 |
| D-55 | Liaison **backend ↔ agent ordinateur** : WebSocket en local | 25/09/2026 |
| D-56 | **Lampe simulée** : appel direct en S15 ; MQTT seulement si l'objet connecté réel (ESP32) est retenu (D-05) | 25/09/2026 |
| D-57 | **Arrêt d'urgence** : raccourci clavier global géré par un petit programme local (ou par l'agent), qui agit directement sur le Core sans passer par l'interface Web ; qui peut le déclencher reste ouvert (D-19) | 25/09/2026 |
| D-58 | **Stockage** : base de données **PostgreSQL** pour les données métier ; signal EEG enregistré, modèles et exports en **fichiers** sur disque | 25/09/2026 |
| D-59 | **Entraînement** des modèles dans le processus du backend, en tâche de fond (pas de service séparé) | 25/09/2026 |
| D-60 | **Accès et comptes** : authentification minimale en S25, comme prévu au planning | 25/09/2026 |
| D-61 | **Contrôle qualité** du signal : composant séparé du prétraitement (il sert le Core, la calibration et la supervision) | 25/09/2026 |
| D-62 | **Core en Python** pour le MVP ; C++ reste une piste pour plus tard (tranche l'ancienne question D-07) | 25/09/2026 |
| D-63 | DIAG-5 comprend **5 diagrammes d'états** : ① états globaux de CortexOS et ② cycle de vie d'une commande (essentiels) ; ③ session, ④ calibration, ⑤ source de signal (utiles). Pas de diagramme pour le système cible ni pour l'alerte (2 à 3 états : attributs) | 25/09/2026 |
| D-64 | Le diagramme ② modélise le **cycle de vie d'une commande depuis la détection** (de « Détectée » à « Exécutée », rejet compris), fidèle à la spécification 4.5 | 25/09/2026 |
| D-65 | La calibration comporte un sous-état **Entraînement** entre la fin des essais et le résultat | 25/09/2026 |
| D-66 | Dans les diagrammes, les comportements non tranchés sont marqués ⚠ avec leur décision `[D-xx]`, sans être décidés | 25/09/2026 |

## 2. Questions ouvertes

| ID | Question | Options ou raison | À décider avant |
|---|---|---|---|
| D-02 | La table des matières est-elle imposée par l'établissement ? | — | S1 |
| D-03 | Quelles intentions pour le MVP, combien, et quel rôle pour les signaux oculaires (commande ou artefact) ? | Candidats évoqués : imagerie motrice (main gauche, main droite, pieds), repos, états de concentration ou de relaxation, clignement volontaire, ouverture ou fermeture des yeux | Bloc casque (C2) |
| D-04 | Quel casque EEG (modèle, nombre de canaux, coût) ? | — | S1 (commande) |
| D-05 | Quels scénarios de démonstration et quel système cible pour le MVP ? | S1 : allumer ou éteindre un équipement via un microcontrôleur · S2 : déplacer ou sélectionner un élément à l'écran · S3 : supervision en direct (signal, intention, confiance, commande) · S4 : démonstration des garde-fous · S5 : évaluation sur des données enregistrées · S6 : robot simulé (extension) | S2 |
| D-06 | Quel système d'exploitation cible ? | — | S16 |
| D-08 | Quel sort pour les technologies non confirmées : Django, Flutter, Redis, gRPC, Docker ? L'ouverture du code (open source) est-elle décidée ? | — | — |
| D-09 | Quelle authentification précise (S25, D-60), quels droits détaillés par rôle (rôles : D-47), quelle durée de conservation, quel partage des données EEG, stockage local ou distant ? (base et fichiers : D-58) | — | S2 ; détail en S22 |
| D-10 | Quels seuils de réussite et quelles cibles de performance (précision, latence, commandes involontaires) ? | — | Bloc casque (C3) |
| D-11 | Robot et drone : extension en simulation, ou recherche et futur ? | — | — |
| D-12 | Quels participants pour le MVP, et comment identifier les besoins des personnes ayant des limitations motrices ? | Option évoquée : volontaires sans limitation motrice d'abord, public cible ensuite | S31 |
| D-13 | Quel budget, quelle configuration matérielle, quel financement ? | — | — |
| D-14 | Année universitaire, encadreurs, projet solo ou en équipe, échéance et dates des jalons | — | Dès qu'elle est connue |
| D-15 | Faut-il un persona illustratif dans la section 1 ? | — | — |
| D-18 | Faut-il distinguer un mode expérimentation et un mode utilisation ? | — | S7 |
| D-19 | Qui peut déclencher l'arrêt d'urgence (utilisateur, accompagnant, opérateur) ? Le mécanisme est fixé par D-57 | — | S9 |
| D-20 | L'association intention → commande est-elle configurable par profil dès le MVP ou en extension ? | — | S17 |
| D-21 | Quelle stratégie de repli si le casque est indisponible ou si la précision est insuffisante ? | Simulateur, données enregistrées, jeux de données publics | Appliquée ici ; à confirmer dans le Cahier des charges |
| D-23 | Qui peut modifier le seuil de confiance, et depuis quel rôle ? | Le seuil agit directement sur le risque de commande involontaire | — |
| D-24 | Quelles commandes sont sensibles, et comment les confirmer (intention EEG, autre moyen, accompagnant) ? | Une confirmation par clic est contraire à l'objectif d'accessibilité | S7 |
| D-25 | L'interface Web doit-elle être pilotable par intentions, et à quel horizon ? | Frontière entre CortexOS outil d'accessibilité et interface accessible | — |
| D-26 | Quel niveau d'accessibilité vise-t-on pour l'interface ? | Sans cible, ENF-09 n'est pas vérifiable | — |
| D-27 | Autorise-t-on le test manuel d'un système cible sans EEG ? | Utile au diagnostic, mais risque de fausser une démonstration | S17 |
| D-28 | Le repos est-il une intention sans commande, ou peut-il déclencher une action ? | Associer le repos à une action augmente le risque de commande involontaire | S7 |
| D-29 | Les commandes sont-elles suspendues au démarrage et après tout incident, jusqu'à une reprise explicite ? | Évite qu'une reconnexion réactive les commandes sans contrôle humain | S7 |
| D-30 | Faut-il un délai minimal entre deux commandes, et lequel ? | Évite des répétitions involontaires d'une même commande | S7 |
| D-31 | Quel délai d'expiration pour une confirmation, et est-il ajustable ? | Sûreté d'un côté, accessibilité (A-06) de l'autre | S7 |
| D-32 | Que fait la plateforme si l'interface Web est fermée ou déconnectée pendant une session active ? | Sans interface, plus personne ne voit l'état ni ne peut suspendre depuis l'écran | — |
| D-33 | Combien de temps une calibration reste-t-elle valable, et quand recommander une recalibration ? | Les signaux varient d'une session à l'autre | — |
| D-34 | Les données exportées ou partagées sont-elles pseudonymisées ? | Protection des données EEG, données personnelles sensibles | S25 |
| D-35 | Faut-il des alertes non visuelles (sonores) ? | Une personne peut ne pas regarder l'écran pendant l'utilisation | — |
| D-36 | Quelle(s) langue(s) pour l'interface ? | Compréhension des messages par tous les profils | — |
| D-37 | Durée maximale d'une session et pauses obligatoires ? | La fatigue dégrade le signal et le confort | — |
| D-38 | Que couvre la suppression des données : sessions, modèles, journal, résultats déjà exportés ? | Rendre le droit à l'effacement applicable concrètement | S25 |
| D-44 | Faut-il une version vectorielle (SVG) du logo, et qui la réalise ? | Les PNG actuels suffisent à l'écran ; le SVG est net à toutes tailles (favicon, impression) | S4 |
| D-67 | **État global** : perte du signal en *Préparation* → État sûr ou Arrêté ? Quelles conditions vérifier avant d'accepter l'activation (qualité, cible disponible) ? | — | S5 |
| D-68 | **Commande** : délai d'attente du résultat ; que faire si la cible devient indisponible entre la décision et l'envoi ; peut-on annuler une action déjà envoyée (arrêt d'urgence) ? | — | S8 |
| D-69 | **Calibration** : une erreur d'entraînement donne « Interrompue » ou « Insuffisante » ? | — | S26 |
| D-70 | **Source de signal** : reconnexion automatique ou manuelle ; que se passe-t-il à la fin d'un enregistrement rejoué ? | — | S10 |
| D-71 | **Session** : effet d'une perte du signal (pause, interrompue, continue ?) ; une session interrompue peut-elle reprendre ? l'enregistrement continue-t-il pendant la pause ? à la reprise, les commandes restent-elles suspendues jusqu'à une reprise explicite ? | — | S22 |

« À décider avant » : semaine du Planning MVP (S1 = 28/09/2026).
