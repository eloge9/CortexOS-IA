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
| D-39 | Diagrammes en Mermaid dans `docs/diagrammes/` ; draw.io seulement à la fin, pour le rapport et la soutenance. **Complétée par D-102** : une V1 draw.io et PDF existe déjà sur Drive | 24/09/2026 |
| D-40 | `docs/` (dépôt Git) est la documentation de référence ; Google Drive n'en est qu'une copie de lecture. Petits changements de doc faits sans demander et signalés ; gros changements validés avant | 24/09/2026 |
| D-41 | Structure de `docs/` numérotée ; un fichier n'est créé que lorsque le travail produit son contenu | 24/09/2026 |
| D-42 | Graphie officielle : « CortexOS IA » (celle du logo) | 24/09/2026 |
| D-43 | Charte graphique : variables CSS + Tailwind v4 ; thème sombre par défaut + thème clair ; polices Exo 2 (titres) + Inter (texte) ; ambiance pro et sobre | 24/09/2026 |
| D-45 | Synchronisation docs / Git / Drive : pendant la séance, seuls les fichiers de `docs/` sont mis à jour (économie de tokens) ; mot-clé « ds » (début de séance) = vérifier les fichiers Drive modifiés et les commentaires ; mot-clé « fs » (fin de séance) = vérifier que les copies Drive n'ont pas été modifiées par Eloge ou l'encadreur (ne jamais écraser : montrer, puis reporter dans `docs/` après accord), republier sur Drive les docs modifiés (ancienne version dans « 99 - Archives »), mettre à jour le classeur de suivi, récapitulatif et commit proposé ; l'encadreur commente plutôt que de modifier (sauf le classeur de suivi, qui n'existe que sur Drive) | 24/09/2026 |
| D-46 | Tous les diagrammes (DIAG-1 à DIAG-14) passent en mode B : Claude les écrit ; Eloge les relit, doit pouvoir les expliquer en soutenance et répond à une question de compréhension après chacun. **Modifiée par D-96** : DIAG-1 à DIAG-8 et ARCH-0 en mode A | 25/09/2026 |
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
| D-72 | **Système cible du MVP** : l'**ordinateur** (réel, via l'agent) **+ une lampe simulée** ; objet connecté réel (ESP32, MQTT) en extension. Tranche la partie « système cible » de D-05 | 25/09/2026 |
| D-73 | **Système d'exploitation** de l'agent ordinateur : **Windows** (celui du PC de développement) ; autres OS plus tard. Tranche D-06 | 25/09/2026 |
| D-74 | **Accès (version minimale de D-09)** : comptes **locaux** (identifiant + mot de passe haché) dans PostgreSQL, **session par cookie**, rôles en base, **toutes les données restent sur le PC** (aucun cloud) | 25/09/2026 |
| D-75 | **Canal de l'arrêt d'urgence** : le programme C18 envoie un **POST sur une route HTTP locale dédiée** du backend, acceptée **uniquement depuis 127.0.0.1** et avec un **jeton local dédié** ; canal indépendant de l'interface Web **et** de l'agent ordinateur, pour rester disponible si l'agent tombe | 25/09/2026 |
| D-76 | L'application Web comporte une **page vitrine publique** (FW-51), seule page accessible sans compte ; contenu et semaine `[À DÉFINIR]`. **Complétée par D-108** : deux autres pages publiques | 25/09/2026 |
| D-77 | Une commande EEG peut partir **hors session** (utilisation libre) ; elle est **toujours journalisée** | 25/09/2026 |
| D-78 | **Journal des détections** : en session d'**expérimentation**, toutes les détections sont journalisées ; en **utilisation**, seulement celles qui mènent à une décision (pas le « repos ») | 25/09/2026 |
| D-79 | **Ordre des garde-fous** du Core (et motif affiché si plusieurs échouent : le premier) : 1 intention « repos », 2 état Actif, 3 qualité, 4 confiance ≥ seuil, 5 cible disponible / commande autorisée / non sensible, 6 délai minimal | 25/09/2026 |
| D-80 | Une confiance **égale au seuil** est **acceptée** (règle « confiance ≥ seuil ») | 25/09/2026 |
| D-81 | En cas de rejet, l'interface affiche le **motif**, la **confiance** et la **valeur du seuil** | 25/09/2026 |
| D-82 | **Rejets répétés** : au MVP, aucune alerte ; ils sont seulement comptés dans les mesures (recommandation de recalibration : extension, F-10, D-33) | 25/09/2026 |
| D-83 | En expérimentation, une détection **rejetée compte dans la précision** (la précision porte sur les détections, pas sur les commandes) | 25/09/2026 |
| D-24 | **Confirmation des commandes sensibles** : par l'**accompagnant** (clic dans l'interface) **ou** par l'**utilisateur** avec une **intention EEG « oui »** ; le clic de l'utilisateur est accepté en développement. Liste des commandes sensibles `[À DÉFINIR]` (S7) | 25/09/2026 |
| D-31 | **Délai d'expiration** d'une confirmation : valeur par défaut, **réglable par profil** (accessibilité A-06) ; valeur par défaut `[À DÉFINIR]` (S7) | 25/09/2026 |
| D-85 | Pendant qu'une commande attend sa confirmation, les nouvelles détections sont **rejetées** (nouveau motif « confirmation en attente ») ; une seule commande en attente à la fois ; exception : l'intention « oui » confirme | 25/09/2026 |
| D-86 | L'**expiration** d'une confirmation est décidée par le **minuteur du Core** ; l'interface affiche seulement le compte à rebours calculé à partir de l'échéance | 25/09/2026 |
| D-87 | `confirmer` et `annuler` sont **idempotents** : une deuxième demande sur la même commande répond « déjà traitée » | 25/09/2026 |
| D-88 | Le **temps de décision humaine** (attente d'une confirmation) est mesuré à part et **exclu** de la latence de bout en bout | 25/09/2026 |
| D-89 | **Auteur** d'une confirmation ou d'une annulation : le compte connecté (D-74) ; « intention EEG » pour une confirmation par EEG | 25/09/2026 |
| D-29 | Au **démarrage**, les commandes sont suspendues (état Prêt). **Après un incident résolu**, CortexOS passe en **Suspendu** ; les commandes ne reprennent que par une **action explicite** (« Reprendre ») | 26/09/2026 |
| D-90 | Le **chien de garde du signal** est dans le **Contrôle qualité (C12)**, avec sa **propre minuterie**, indépendante de la boucle d'acquisition ; délai sans échantillon `[À DÉFINIR — Documentation technique]` | 26/09/2026 |
| D-91 | **Conditions pour reprendre** les commandes : signal présent, qualité suffisante, modèle chargé, cible disponible | 26/09/2026 |
| D-92 | Après une reconnexion du casque, l'interface **propose** (sans l'imposer) une **recalibration** si le casque a été retiré | 26/09/2026 |
| D-93 | Une commande **déjà envoyée** au moment d'un passage en état sûr n'est **pas rappelée** : on la laisse finir et son résultat est journalisé « reçu en état sûr » | 26/09/2026 |
| D-94 | **Reconnexion du casque** : tentatives **automatiques** périodiques **et** bouton « Reconnecter » ; dans tous les cas, jamais de reprise automatique des commandes (F-05) | 26/09/2026 |
| D-95 | **Perte du signal pendant une session** : la session passe **En pause** automatiquement ; la période sans signal est **exclue des mesures** | 26/09/2026 |
| D-96 | **DIAG-1 à DIAG-8 et ARCH-0 passent en mode A** : Eloge rédige l'analyse textuelle ; Claude écrit le Mermaid, complète et corrige. D-46 (mode B) reste valable pour DIAG-9 à DIAG-14. Relectures de DIAG-2 (vue d'ensemble), DIAG-4 et DIAG-5 validées le 25/09/2026 | 26/09/2026 |
| D-97 | **DIAG-7 compte 8 diagrammes d'activité** : A1 session d'expérimentation (vue d'ensemble) et A1-b boucle des essais, A2 traitement d'une fenêtre jusqu'à la décision, A3 première utilisation, A4 à A7 activités des séquences (a) à (d) | 27/09/2026 |
| D-98 | Pendant une session d'**expérimentation**, les commandes sont **réellement exécutées**, sur la lampe simulée comme sur l'ordinateur (permet de mesurer les commandes involontaires) | 27/09/2026 |
| D-99 | Si le profil n'a **pas de modèle exploitable** à la création d'une session, la calibration (activité A3) est faite **avant** de démarrer la session | 27/09/2026 |
| D-100 | Le contrôle « **une commande attend déjà sa confirmation** » (D-85) se place **après le garde-fou 4** de D-79 : une intention « oui » ne confirme que si la qualité et la confiance sont suffisantes | 27/09/2026 |
| D-04 | **Casque EEG** : module d'acquisition **ADS1299, 8 canaux** (lien noté dans le classeur de suivi). Commande bloquée au 27/09/2026 (raison `[À DÉFINIR]`). **À vérifier avant la commande** : électrodes et bonnet fournis ou non ; liaison avec le PC ; compatibilité avec BrainFlow (D-21) | 27/09/2026 |
| D-101 | **Propositions P1 à P8 d'ARCH-0 validées** : monodépôt (P1) ; ports 3000 / 8000 / 5432 et routes `/api/v1`, `/api/local`, `/ws/flux`, `/ws/agent` (P2) ; bibliothèques Uvicorn, Pydantic, SQLAlchemy 2, psycopg 3, Alembic, argon2-cffi, pynput, websockets, httpx (P3) ; lampe simulée dans le processus du backend (P4) ; l'agent se connecte au backend (P5) ; lecture BrainFlow et entraînement dans des fils séparés (P6) ; premier administrateur créé en ligne de commande (P7) ; jetons locaux générés au démarrage dans `data/` (P8) | 03/10/2026 |
| D-102 | Les versions **draw.io et PDF** des diagrammes (dossier Drive « 04 - Conception / Diagrammes / V1 (pdf et drawio) ») restent **sur Drive seulement** ; la référence versionnée reste le Mermaid de `docs/diagrammes/` | 03/10/2026 |
| D-103 | **Diagrammes DIAG-1 à DIAG-8 terminés et validés** : DIAG-6 (séquences a à d), DIAG-7 (8 diagrammes d'activité) et DIAG-8 (déploiement 8a et 8b) validés par Eloge ; la conception d'analyse (mode A, D-96) est close. Suite : contrat d'API (API-0) et maquettes | 04/10/2026 |
| D-104 | **Conventions du contrat d'API** (API-0, partie 1, propositions P-A1 à P-A6) : noms de champs et valeurs en français, `snake_case`, sans accents ; identifiants UUID ; dates ISO 8601 en UTC à la milliseconde ; format unique des erreurs `{ erreur: { code, message, action_possible, details } }` ; pagination `limite` / `decalage` ; qualité du signal sur deux niveaux (`suffisante`, `insuffisante`) au MVP | 04/10/2026 |
| D-105 | **Drive seulement avec l'accord d'Eloge** (économie de tokens) : pendant le travail, Claude met à jour librement `docs/` sur le PC ; aucune écriture sur Drive sans l'accord d'Eloge. **Taper `fs` vaut accord** : au `fs`, Claude republie directement, en entier, tous les documents modifiés depuis la dernière republication (précisé le 05/10/2026) ; en dehors du `fs`, seulement sur demande explicite. Complète D-45 | 04/10/2026 |
| D-106 | **Routes REST du contrat d'API** (API-0, partie 2, propositions P-A7 à P-A10) : une transition d'état = un `POST` sur un verbe (`/sessions/{id}/demarrer`) ; réponse **202** pour connecter et reconnecter la source (résultat sur `/ws/flux`) ; `/api/health` hors de `/v1` ; rôles minimaux proposés par route en attendant D-09, **suspendre permis à toute personne connectée** | 05/10/2026 |
| D-107 | **Temps réel et canaux locaux du contrat d'API** (API-0, partie 3, propositions P-A11 à P-A16) : enveloppe commune `{ type, horodatage, sequence, donnees }` sur `/ws/flux` ; `/ws/flux` sert à recevoir, toutes les actions passent par REST (le navigateur n'envoie que `abonner` / `desabonner`) ; signal envoyé aux seuls abonnés par paquets d'environ 0,1 s ; battement toutes les 5 s, « données périmées » après 10 s sans message ; jetons locaux dans le premier message (agent) ou l'en-tête `X-Jeton-Local` (arrêt d'urgence), jamais dans l'adresse ; arrêt d'urgence toujours 200. **Contrat d'API v1.0 validé (API-0 terminé)** | 05/10/2026 |
| D-108 | Deux **pages publiques** s'ajoutent à la vitrine : **« Confidentialité et données »** (FW-52, politique de confidentialité / « page RGPD ») et **« Conditions d'utilisation »** (FW-53, CGU). Contenu rédigé dans `docs/` et relu par l'encadreur ; cadre légal applicable `[À VÉRIFIER]` ; réalisation en S25 avec le consentement, **obligatoirement avant la campagne d'évaluation (S31)** | 05/10/2026 |
| D-109 | **Règles « pas d'apparence générique »** (charte § 9) : aucun dégradé décoratif (seule exception : une échelle de couleur qui représente une valeur), pas de boutons pilule, pas d'émojis comme icônes, pas d'animation au défilement ni de curseur animé, pas d'effet lumineux, pas d'image générée par IA, pas de badge d'outil, favicon obligatoire ; **aucun chiffre, avis ou compteur inventé**, aucune accroche vague, aucun texte non relu par Eloge | 05/10/2026 |

## 2. Questions ouvertes

| ID | Question | Options ou raison | À décider avant |
|---|---|---|---|
| D-02 | La table des matières est-elle imposée par l'établissement ? | — | S1 |
| D-03 | Quelles intentions pour le MVP, combien, et quel rôle pour les signaux oculaires (commande ou artefact) ? | Candidats évoqués : imagerie motrice (main gauche, main droite, pieds), repos, états de concentration ou de relaxation, clignement volontaire, ouverture ou fermeture des yeux | Bloc casque (C2) |
| D-05 | Quels scénarios de démonstration pour le MVP ? (système cible tranché par D-72) | S1 : allumer ou éteindre un équipement via un microcontrôleur · S2 : déplacer ou sélectionner un élément à l'écran · S3 : supervision en direct (signal, intention, confiance, commande) · S4 : démonstration des garde-fous · S5 : évaluation sur des données enregistrées · S6 : robot simulé (extension) | S2 |
| D-08 | Quel sort pour les technologies non confirmées : Django, Flutter, Redis, gRPC, Docker ? L'ouverture du code (open source) est-elle décidée ? | — | — |
| D-09 | Quels droits détaillés par rôle (rôles : D-47), quelle durée de conservation, quel partage des données EEG ? (accès minimal et stockage local : D-74 ; base et fichiers : D-58) | — | S22 |
| D-10 | Quels seuils de réussite et quelles cibles de performance (précision, latence, commandes involontaires) ? | — | Bloc casque (C3) |
| D-11 | Robot et drone : extension en simulation, ou recherche et futur ? | — | — |
| D-12 | Quels participants pour le MVP, et comment identifier les besoins des personnes ayant des limitations motrices ? | Option évoquée : volontaires sans limitation motrice d'abord, public cible ensuite | S31 |
| D-13 | Quel budget, quelle configuration matérielle, quel financement ? | — | — |
| D-14 | Encadreurs, projet solo ou en équipe, échéance et dates des jalons (année universitaire : 2026-2027, indiquée par Eloge le 30/09/2026) | — | Dès qu'elle est connue |
| D-15 | Faut-il un persona illustratif dans la section 1 ? | — | — |
| D-18 | Faut-il distinguer un mode expérimentation et un mode utilisation ? (exécution des commandes en expérimentation : tranchée par D-98) | — | S7 |
| D-19 | Qui peut déclencher l'arrêt d'urgence (utilisateur, accompagnant, opérateur) ? Le mécanisme est fixé par D-57 | — | S9 |
| D-20 | L'association intention → commande est-elle configurable par profil dès le MVP ou en extension ? | — | S17 |
| D-21 | Quelle stratégie de repli si le casque est indisponible ou si la précision est insuffisante ? | Simulateur, données enregistrées, jeux de données publics | Appliquée ici ; à confirmer dans le Cahier des charges |
| D-23 | Qui peut modifier le seuil de confiance, et depuis quel rôle ? | Le seuil agit directement sur le risque de commande involontaire | — |
| D-25 | L'interface Web doit-elle être pilotable par intentions, et à quel horizon ? | Frontière entre CortexOS outil d'accessibilité et interface accessible | — |
| D-26 | Quel niveau d'accessibilité vise-t-on pour l'interface ? | Sans cible, ENF-09 n'est pas vérifiable | — |
| D-27 | Autorise-t-on le test manuel d'un système cible sans EEG ? | Utile au diagnostic, mais risque de fausser une démonstration | S17 |
| D-28 | Le repos est-il une intention sans commande, ou peut-il déclencher une action ? | Associer le repos à une action augmente le risque de commande involontaire | S7 |
| D-30 | Faut-il un délai minimal entre deux commandes, et lequel ? | Évite des répétitions involontaires d'une même commande | S7 |
| D-32 | Que fait la plateforme si l'interface Web est fermée ou déconnectée pendant une session active ? | Sans interface, plus personne ne voit l'état ni ne peut suspendre depuis l'écran | — |
| D-33 | Combien de temps une calibration reste-t-elle valable, et quand recommander une recalibration ? | Les signaux varient d'une session à l'autre | — |
| D-34 | Les données exportées ou partagées sont-elles pseudonymisées ? | Protection des données EEG, données personnelles sensibles | S25 |
| D-35 | Faut-il des alertes non visuelles (sonores) ? | Une personne peut ne pas regarder l'écran pendant l'utilisation | — |
| D-36 | Quelle(s) langue(s) pour l'interface ? | Compréhension des messages par tous les profils | — |
| D-37 | Durée maximale d'une session et pauses obligatoires ? | La fatigue dégrade le signal et le confort | — |
| D-38 | Que couvre la suppression des données : sessions, modèles, journal, résultats déjà exportés ? | Rendre le droit à l'effacement applicable concrètement | S25 |
| D-44 | Faut-il une version vectorielle (SVG) du logo, et qui la réalise ? | Les PNG actuels suffisent à l'écran ; le SVG est net à toutes tailles (favicon, impression) | S4 |
| D-67 | **État global** : perte du signal en *Préparation* → État sûr ou Arrêté ? Quelles conditions vérifier avant d'accepter l'activation (qualité, cible disponible) ? | — | S5 |
| D-68 | **Commande** : délai d'attente du résultat ; que faire si la cible devient indisponible entre la décision et l'envoi ? (action déjà envoyée : jamais rappelée, D-93) | — | S8 |
| D-69 | **Calibration** : une erreur d'entraînement donne « Interrompue » ou « Insuffisante » ? | — | S26 |
| D-70 | **Source de signal** : que se passe-t-il à la fin d'un enregistrement rejoué ? (reconnexion : tranchée par D-94) | — | S10 |
| D-71 | **Session** : une session interrompue peut-elle reprendre ? l'enregistrement continue-t-il pendant la pause ? (effet d'une perte du signal : tranché par D-95) | — | S22 |
| D-84 | **Mesures** : une détection « repos » est-elle comptée comme un rejet ou dans une catégorie à part (« aucune commande attendue ») ? | Proposition : catégorie à part, pour ne pas fausser le taux de rejet | S24 |

« À décider avant » : semaine du Planning MVP (S1 = 28/09/2026).
