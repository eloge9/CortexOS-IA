# CortexOS IA

Plateforme qui interprète des signaux EEG grâce à l'IA pour superviser et commander des systèmes connectés : ordinateur, objets connectés, et à terme robots ou drones.

**Chaîne :** intention → détection → décision → commande → action → résultat.

> Statut : conception (tranche 0). Le code arrive à partir de la semaine 4 (squelette FastAPI + Next.js).

## Documentation

| Document | Contenu |
|---|---|
| [Index](docs/00-index.md) | Point d'entrée de toute la documentation |
| [Cahier des charges](docs/01-cahier-des-charges.md) | Besoin, périmètre, exigences |
| [Spécification fonctionnelle](docs/02-specification-fonctionnelle.md) | Fonctions F-xx et fonctionnalités web FW-xx |
| [Planning MVP](docs/03-planning-mvp.md) | Tranches, semaines, jalons |
| [Développement sur PC seul](docs/04-developpement-pc-seul.md) | Ce qui se développe sans casque EEG |
| [Décisions](docs/05-decisions.md) | Registre des décisions |
| [Diagrammes](docs/diagrammes/) | Diagrammes de conception (Mermaid) |

## Stack prévue

- **EEG / IA :** Python (BrainFlow, MNE-Python, scikit-learn)
- **Backend :** FastAPI (monolithe modulaire)
- **Frontend :** Next.js / TypeScript
- **Extensions :** MQTT / ESP32, ROS 2, Core C++

## Installation

À venir avec le squelette du projet.

## Auteur

Eloge — projet de fin d'études (PPE).
