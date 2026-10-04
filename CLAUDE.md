# CLAUDE.md — Règles pour Claude Code sur CortexOS IA

Réponds toujours en **français**. Sois concis : pas de texte inutile.

## Le projet

CortexOS IA : plateforme qui interprète des signaux EEG avec l'IA pour superviser et commander des systèmes (ordinateur, objets connectés, puis robot/drone en extension).
Chaîne : intention → détection → décision → commande → action → résultat.
Projet de fin d'études (PPE) d'Eloge, en apprentissage. Pas encore de casque EEG : tout se développe d'abord en simulation.

Documents de référence (à lire avant toute modification importante) :

- `docs/00-index.md` — point d'entrée, liste des documents existants et à venir
- `docs/01-cahier-des-charges.md` — besoin, périmètre, exigences BF-xx / ENF-xx
- `docs/02-specification-fonctionnelle.md` — fonctions F-xx, fonctionnalités web FW-xx
- `docs/03-planning-mvp.md` — tranches, semaines, jalons (planning de référence)
- `docs/04-developpement-pc-seul.md` — ce qui se fait sans casque
- `docs/05-decisions.md` — **registre unique** des décisions D-xx (à mettre à jour quand Eloge tranche)
- `docs/06-charte-graphique.md` — couleurs, polices, composants ; **aucune couleur en dur dans le frontend**, toujours les variables de la charte
- `docs/diagrammes/` — diagrammes Mermaid

## Architecture (décidée)

- **Monolithe modulaire** : un seul backend découpé en modules à interfaces claires. Pas de microservices.
- L'**agent ordinateur** est un programme séparé.
- `core/` en **Python pur**, indépendant de FastAPI, testé seul.
- Interfaces interchangeables : `SourceEEG` (simulation, fichier, casque) et `Connecteur` (ordinateur, lampe simulée…).
- Stack : Python (EEG/IA : BrainFlow, MNE, scikit-learn), FastAPI, Next.js/TypeScript.
- Développement par **tranches verticales**, contrat d'API défini avant le code.
- Décidé : Core en Python (D-62), PostgreSQL + fichiers (D-58), orchestrateur de la chaîne (D-54), agent en WebSocket local (D-55), arrêt d'urgence par raccourci global (D-57) avec route HTTP locale + jeton (D-75), cible ordinateur + lampe simulée (D-72), Windows (D-73), comptes locaux + cookie, tout sur le PC (D-74), page vitrine publique (D-76). Architecture : `docs/07-architecture-technique.md` (propositions P1–P8 à valider).
- Décidé aussi (DIAG-6, 25-26/09) : D-24 (confirmation par accompagnant ou EEG « oui »), D-29 (après incident → Suspendu), D-31 (délai réglable par profil), D-77 à D-95 (séquences). Diagrammes DIAG-1 à DIAG-8 et ARCH-0 en mode A (D-96) : Eloge rédige l'analyse, Claude écrit le Mermaid, complète et corrige.
- Décidé le 27/09–03/10 : D-04 casque = module ADS1299 8 canaux (compatibilité BrainFlow à vérifier, commande bloquée) ; D-97 à D-100 (DIAG-7) ; D-101 = P1 à P8 d'ARCH-0 validées ; D-102 = draw.io/PDF sur Drive seulement.
- Décidé le 04/10 : D-103 = DIAG-1 à DIAG-8 terminés et validés (DIAG-6, 7, 8 relus par Eloge). Prochaine étape : API-0 (contrat d'API) et maquettes.
- Non tranché : D-05 (scénarios de démonstration), D-09 (droits par rôle, conservation, partage), D-19 (qui déclenche l'arrêt), D-30, D-67 à D-71 (parties restantes), D-84. Ne décide pas à la place d'Eloge.

## Règles de travail

1. **Analyse le code existant avant de modifier.** Respecte les conventions. Ne réécris pas ce qui marche.
2. **Pas de nouvelle dépendance ou technologie sans raison expliquée.** Pas de sur-ingénierie.
3. **Deux modes** :
   - **Mode A** (Eloge code) : explique, découpe, donne une étape, laisse-le coder, corrige ses erreurs. Ne donne pas toute la solution.
   - **Mode B** (tu codes) : avant, dis quoi, pourquoi, quels fichiers. Après, compte rendu : ce que j'ai fait · pourquoi · comment ça fonctionne (entrée → traitement → sortie) · fichiers · code important · comment tester · proposition de commit.
4. Tout code produit doit pouvoir être expliqué à Eloge. Explique les notions nouvelles.
5. **Débogage** : cause → explication → correction (si demandée) → comment l'éviter → test.
6. **Git** : commits petits et clairs, en français, format `type(portée): description` (ex. `feat(core): ajoute la machine à états`). Ne pousse (`git push`) qu'après accord d'Eloge.
7. Pose parfois une question courte pour vérifier qu'Eloge a compris.
8. Priorités : compréhension > fonctionnement correct > qualité > architecture > sécurité > maintenabilité > performance > vitesse.
9. **Consignes et validations d'Eloge** : dès qu'Eloge donne une consigne, valide un livrable ou tranche un point (même en passant, dans une demande), le reporter aussitôt dans `docs/` : registre `05-decisions.md`, retrait des `[À DÉFINIR]` concernés, documents touchés (cahier des charges, spécification, planning, diagrammes). Le signaler en fin de réponse. En cas de doute (« est-ce une décision ou une demande ponctuelle ? »), lui demander : « Je note ça comme décision ? ». Les propositions de Claude restent « proposé » tant qu'Eloge ne les a pas validées.

## Mise à jour des documents (règle 15)

- **Petits changements** (statut, précision, décision prise par Eloge en séance) : mets à jour `docs/` sans demander, dans le même commit que le code, et signale-le en fin de réponse (« Docs mises à jour : … »).
- **Gros changements** (périmètre, dates du planning, architecture, technologie, suppression de fonctionnalité) : demande avant.
- Point flou : note `[À DÉFINIR — D-xx]`, n'invente jamais une décision.
- `docs/` est la référence. Google Drive n'est qu'une copie de lecture.
- Diagrammes : en **Mermaid** dans `docs/diagrammes/`. draw.io seulement à la fin pour le rapport.
- Nouveau document : seulement quand le travail produit son contenu (voir « À venir » dans `docs/00-index.md`), puis l'ajouter à l'index.

## Synchronisation Git / Drive (D-45)

- **« début de séance »** : liste les fichiers Drive (dossier « CortexOS IA ») modifiés depuis la dernière synchronisation et les nouveaux commentaires ; résume-les à Eloge.
- **Avant de modifier ou de resynchroniser un fichier Drive**, vérifie s'il a été modifié depuis la dernière synchronisation (par Eloge ou l'encadreur). Si oui : ne l'écrase jamais, montre les changements, reporte-les dans `docs/` après accord d'Eloge.
- **« fin de séance »** : récapitulatif, message de commit proposé (Eloge lance `git`), republication sur Drive des docs modifiées, mise à jour du classeur « Tableau de suivi — CortexOS IA » (dossier « 03 - Planning et suivi », seule source du suivi, n'existe que sur Drive).
- **Claude tient à jour toutes les copies Drive** : un doc modifié dans `docs/` est republié (nouvelle version au même endroit, titre « Nom — CortexOS IA (vX.Y) », ancienne version dans « 99 - Archives »), puis la carte ci-dessous est mise à jour.
- L'encadreur commente les documents plutôt que de les modifier (sauf le classeur de suivi).

### Carte des documents Drive (dossier « CortexOS IA »)

| `docs/` (référence) | Copie Drive (dossier · ID) |
|---|---|
| `01-cahier-des-charges.md` | 01 - Cadrage · `1tHzF8DdNUWbAJ-kPWVEuUf7T0hJWwSpFlYzL7OANav0` (v2.2) |
| `02-specification-fonctionnelle.md` | 02 - Spécification · `1lJXpGcBWE7rpFpuZLZfkkuALarsU8HAh_hm0tCma9mM` (v1.2) |
| `03-planning-mvp.md` | 03 - Planning et suivi · `1hGN7P2YGQbNMl-_45y2eH8PtNMJDpwSb37qXyVpldlg` (v3.2) |
| `04-developpement-pc-seul.md` | 03 - Planning et suivi · `1FlKusLkbv9Atd55LNp2xuuV8Ua4wyKwDP6FXJOoDiGU` |
| `05-decisions.md` | 01 - Cadrage · `1ILRW3AvUgK1LBlSJgUyQUEKKV5bGZd2P9IEG0-81yYw` |
| `06-charte-graphique.md` | 04 - Conception · `1DlstuQh8dYu96B44tQtIUgXbnecW-2sc_PQTaO7J7NQ` |
| `07-architecture-technique.md` | 04 - Conception · `16gwNcjf1BzQr_74IftWb-eQb-zdby5SrfVMQNg8h0pc` (v0.1) |
| `assets/logo/` | 04 - Conception / Logos (8 PNG) |
| `diagrammes/` | 04 - Conception / Diagrammes : index `1XNCbW8S5n_u9Vet889Nii10oTBCyyD6n5wdOwUHgoOI` · DIAG-1 `1LYxH206B9t9mgweGoeuZzek5SesaN6wFLwzGSPvyac8` · DIAG-2a `1PNMJTIoe_EGRitKnJV8nZwiyCDImPSi5Um508OMRCs0` · DIAG-2b `10kSdyLFnPVIyyLKP9RqWV1zxNhpAU03FBQJ6g-Hb9bU` · DIAG-3 `19Vd_5fExEVNblAzQDRDLKEyaZw8GU-co6byDql2xkdM` · DIAG-4 `1WwCnmmD9O0lwlGxPt7R_LqfMIOln_A_mbSR7M0kct00` (v1.3) · DIAG-5 `16oG9d59pviD414TwFma-zvEMllk33kPthfR14RrJysY` · DIAG-6 `1dTo1BETsm5joyZXyfOhQLqblw1nOzcr76MUvGMBNarA` · DIAG-7 `189HiZYkkPbB3BGw6NUiuhOkXceqwIoEwho9bApuYULk` · DIAG-8a `1NAgtUZCfeTSrLHE2DtPBABsa6TnKseZl70ezgvV21z4` · DIAG-8b `1CYrI8mF8s9k3CEOX06dVKrudSX9L7zEcyFCaWdm__os` · sous-dossier « V1 (pdf et drawio) » `1BDlRD01uSE2VztrCLMd3ajGeU_WSRsJp` (draw.io + PDF d'Eloge, Drive seulement, D-102) (texte et tableaux ; le Mermaid reste dans Git) |
| — (Drive seul) | Racine : « 00 - À lire d'abord » `1zOaxVEo3XOv4yvHkZoBdBARSzRcUCTPYVo3WXRImivE` |
| — (Drive seul) | 03 - Planning et suivi : « Tableau de suivi » `1YYopxwHtpu-pPZV-6wcIV2saAVnL8ywu5iKx6Q3veBE` |

Dossiers : 05 - Documentation technique et 06 - Rapport et soutenance (vides), 99 - Archives (anciennes versions). Nouvelle version d'un doc Drive : l'ancienne va dans 99 - Archives. Tenir cette carte à jour. Dernière republication complète : 03/10/2026 (fs : 01, 02, 03, 05, 07, « 00 - À lire d'abord », diagrammes index, DIAG-4 v1.3, DIAG-7, DIAG-8a, DIAG-8b ; classeur non mis à jour : navigateur déconnecté de Google). Classeur de suivi : saisie par la barre de formule (sélection via la zone de nom, clic sur la barre de formule, Ctrl+A, texte, Entrée, attendre ~2 s entre chaque étape ; au-delà de 2 cellules par lot, des saisies se perdent). Le navigateur intégré doit être connecté au compte Google (sinon : Eloge remplit lui-même). Vérifier ensuite par lecture CSV (`/gviz/tq?tqx=out:csv&gid=87982260&range=…`).
