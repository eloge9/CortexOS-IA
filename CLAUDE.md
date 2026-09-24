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
- `docs/diagrammes/` — diagrammes Mermaid

## Architecture (décidée)

- **Monolithe modulaire** : un seul backend découpé en modules à interfaces claires. Pas de microservices.
- L'**agent ordinateur** est un programme séparé.
- `core/` en **Python pur**, indépendant de FastAPI, testé seul.
- Interfaces interchangeables : `SourceEEG` (simulation, fichier, casque) et `Connecteur` (ordinateur, lampe simulée…).
- Stack : Python (EEG/IA : BrainFlow, MNE, scikit-learn), FastAPI, Next.js/TypeScript.
- Développement par **tranches verticales**, contrat d'API défini avant le code.
- Non tranché : D-05 (système cible), D-07 (langage du Core, Python recommandé), D-09 (stockage/auth). Ne décide pas à la place d'Eloge.

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

## Mise à jour des documents (règle 15)

- **Petits changements** (statut, précision, décision prise par Eloge en séance) : mets à jour `docs/` sans demander, dans le même commit que le code, et signale-le en fin de réponse (« Docs mises à jour : … »).
- **Gros changements** (périmètre, dates du planning, architecture, technologie, suppression de fonctionnalité) : demande avant.
- Point flou : note `[À DÉFINIR — D-xx]`, n'invente jamais une décision.
- `docs/` est la référence. Google Drive n'est qu'une copie de lecture.
- Diagrammes : en **Mermaid** dans `docs/diagrammes/`. draw.io seulement à la fin pour le rapport.
- Nouveau document : seulement quand le travail produit son contenu (voir « À venir » dans `docs/00-index.md`), puis l'ajouter à l'index.
