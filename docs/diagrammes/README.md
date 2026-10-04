# Diagrammes

Chaque diagramme est un fichier `.md` contenant un bloc Mermaid (GitHub l'affiche directement).
DIAG-1 à DIAG-8 (et ARCH-0) sont en **mode A** (D-96) : Eloge rédige l'analyse textuelle, Claude écrit le Mermaid, complète et corrige. DIAG-9 à DIAG-14 restent en mode B (D-46) : Claude les écrit, Eloge les relit. Dans les deux cas, Eloge doit savoir les expliquer et répond à une question de compréhension.
Nommage : `diag-01-contexte.md`, `diag-02a-…`, `diag-02b-…`, etc.
Cas d'utilisation : acteurs humains à gauche, non humains à droite (D-48).

| Réf. | Diagramme | Semaine | Mode | Statut |
|---|---|---|---|---|
| DIAG-1 | [Contexte](diag-01-contexte.md) | S1 | A | Validé (24/09/2026, v1.1 le 25/09) |
| DIAG-2 | Cas d'utilisation : [par acteur](diag-02a-cas-utilisation-par-acteur.md) · [vue d'ensemble](diag-02b-cas-utilisation-vue-ensemble.md) | S1 | A | Validé (25/09/2026) |
| DIAG-3 | [Classes de domaine](diag-03-classes-domaine.md) | S2 | A | Validé (25/09/2026) |
| DIAG-4 | [Composants](diag-04-composants.md) | S2 | A | Validé (25/09/2026) |
| DIAG-5 | [États-transitions](diag-05-etats-transitions.md) (5 diagrammes, D-63) | S2 | A | Validé (25/09/2026) |
| DIAG-6 | [Séquences](diag-06-sequences.md) (4 scénarios) | S3 | A | (a) à (d) à relire par Eloge |
| DIAG-7 | [Activités](diag-07-activites.md) (8 diagrammes : A1, A1-b, A2 à A7, D-97) | S3 | A | v0.1 à relire par Eloge |
| DIAG-8 | Déploiement : [tout sur le PC](diag-08a-deploiement-pc.md) · [ajouts matériels](diag-08b-deploiement-materiel.md) | S3 | A | v0.1 à relire par Eloge |
| DIAG-9 | Classes de conception du Core | S5 | B | À faire |
| DIAG-10 | Classes de l'interface SourceEEG | S10 | B | À faire |
| DIAG-11 | Séquence temps réel | S11 | B | À faire |
| DIAG-12 | Classes de l'interface Connecteur | S15 | B | À faire |
| DIAG-13 | Modèle de données | S22 | B | À faire |
| DIAG-14 | Séquence de la calibration | S26 | B | À faire |
