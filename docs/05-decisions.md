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

## 2. Questions ouvertes

| ID | Question | Options ou raison | À décider avant |
|---|---|---|---|
| D-02 | La table des matières est-elle imposée par l'établissement ? | — | S1 |
| D-03 | Quelles intentions pour le MVP, combien, et quel rôle pour les signaux oculaires (commande ou artefact) ? | Candidats évoqués : imagerie motrice (main gauche, main droite, pieds), repos, états de concentration ou de relaxation, clignement volontaire, ouverture ou fermeture des yeux | Bloc casque (C2) |
| D-04 | Quel casque EEG (modèle, nombre de canaux, coût) ? | — | S1 (commande) |
| D-05 | Quels scénarios de démonstration et quel système cible pour le MVP ? | S1 : allumer ou éteindre un équipement via un microcontrôleur · S2 : déplacer ou sélectionner un élément à l'écran · S3 : supervision en direct (signal, intention, confiance, commande) · S4 : démonstration des garde-fous · S5 : évaluation sur des données enregistrées · S6 : robot simulé (extension) | S2 |
| D-06 | Quel système d'exploitation cible ? | — | S16 |
| D-07 | Quel langage pour le Core dans le MVP ? | Python, ou C++ dès le départ | S2 |
| D-08 | Quel sort pour les technologies non confirmées : Django, Flutter, Redis, gRPC, Docker ? L'ouverture du code (open source) est-elle décidée ? | — | — |
| D-09 | Où stocker les données (local ou distant), quelle authentification et quels rôles, quelle durée de conservation, quel partage des données EEG ? | — | S2 ; détail en S22 |
| D-10 | Quels seuils de réussite et quelles cibles de performance (précision, latence, commandes involontaires) ? | — | Bloc casque (C3) |
| D-11 | Robot et drone : extension en simulation, ou recherche et futur ? | — | — |
| D-12 | Quels participants pour le MVP, et comment identifier les besoins des personnes ayant des limitations motrices ? | Option évoquée : volontaires sans limitation motrice d'abord, public cible ensuite | S31 |
| D-13 | Quel budget, quelle configuration matérielle, quel financement ? | — | — |
| D-14 | Graphie officielle (CortexOS IA ou AI), année universitaire, encadreurs, projet solo ou en équipe, échéance et dates des jalons | — | Dès qu'elle est connue |
| D-15 | Faut-il un persona illustratif dans la section 1 ? | — | — |
| D-18 | Faut-il distinguer un mode expérimentation et un mode utilisation ? | — | S7 |
| D-19 | Quel mécanisme d'arrêt d'urgence, et qui peut le déclencher (utilisateur, accompagnant, opérateur) ? | — | S9 |
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

« À décider avant » : semaine du Planning MVP (S1 = 28/09/2026).
