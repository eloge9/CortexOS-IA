# DIAG-2b — Cas d'utilisation : vue d'ensemble

| | |
|---|---|
| **Réf.** | DIAG-2 (Planning MVP, S1, mode B — D-46) |
| **Sources** | Catalogue des cas d'utilisation (section 2 de `diag-02a-cas-utilisation-par-acteur.md`) |
| **Version** | 1.0 — 25 septembre 2026 |

## Notation

| Élément | Représentation | Sens |
|---|---|---|
| Acteur humain | 👤 **à gauche** | Personne qui déclenche le cas (ou y participe) |
| Acteur non humain | ⚙️ **à droite** | Système externe sollicité par le cas (casque, jeu de données, système cible) |
| Cas d'utilisation | forme arrondie, dans le cadre « CortexOS IA » | Objectif d'un acteur ; le cadre est la frontière du système (DIAG-1) |
| Cas interne | forme arrondie **grisée, bord pointillé** | Jamais déclenché seul : n'existe que par un `«include»` (UC-42 à UC-47) |
| `«include»` | flèche pointillée du cas de base **vers** le cas inclus | Le cas inclus est **toujours** exécuté |
| `«extend»` | flèche pointillée du cas optionnel **vers** le cas étendu, avec la condition | Le cas optionnel s'ajoute **seulement si** la condition est vraie |
| Généralisation | flèche pleine vers le parent | L'enfant hérite de tous les cas du parent |
| 📝 | après le nom du cas | Le cas **inclut UC-47 « Enregistrer dans le journal »** (flèche non dessinée pour garder les diagrammes lisibles) |
| `[D-xx]` | dans le nom du cas | Le cas ou son acteur dépend d'une décision non prise (`05-decisions.md`) |

> Mermaid n'a pas de type « cas d'utilisation » UML : les diagrammes utilisent un `flowchart` qui respecte les conventions ci-dessus. La version draw.io du rapport utilisera les symboles UML standards (D-39).

---

## Diagramme

Un seul cadre « CortexOS IA » contenant **tous les cas** (UC-01 à UC-46, plus les 2 cas hors MVP) ; **acteurs humains à gauche** avec leurs généralisations vers *Personne authentifiée* ; **acteurs non humains à droite** avec les spécialisations de *Système cible* ; toutes les relations `include`, `extend` et les spécialisations de UC-12. Seul UC-47 (journal) reste noté 📝.

Ce diagramme sert à **voir le système entier d'un coup d'œil** (vue de synthèse pour le rapport). Pour **lire** un cas précis, utiliser les diagrammes par acteur (`diag-02a-cas-utilisation-par-acteur.md`), plus lisibles. Il utilise la disposition `elk` de Mermaid, adaptée aux grands diagrammes.

```mermaid
---
config:
  layout: elk
  elk:
    nodePlacementStrategy: NETWORK_SIMPLEX
---
flowchart LR
    PA["👤 Personne authentifiée<br/><i>abstrait</i>"]
    U["👤 Utilisateur"]
    A["👤 Accompagnant /<br/>opérateur"]
    X["👤 Expérimentateur"]
    AD["👤 Administrateur"]
    P["👤 Utilisateur participant"]
    subgraph SYS["CortexOS IA"]
        UC01(["UC-01 S'authentifier [D-09]"])
        UC02(["UC-02 Gérer les comptes et les rôles"])
        UC03(["UC-03 Consulter l'aide contextuelle"])
        UC04(["UC-04 Régler ses préférences d'affichage"])
        UC05(["UC-05 Créer ou sélectionner un profil"])
        UC06(["UC-06 Donner son consentement"])
        UC07(["UC-07 Consulter son consentement"])
        UC08(["UC-08 Retirer son consentement"])
        UC09(["UC-09 Supprimer ses données 📝 [D-38]"])
        UC10(["UC-10 Consulter ses données"])
        UC11(["UC-11 Exporter des données [D-34]"])
        UC12(["UC-12 Connecter / déconnecter la source de signal"])
        UC12a(["UC-12a Connecter le casque EEG"])
        UC12b(["UC-12b Utiliser la simulation"])
        UC12c(["UC-12c Rejouer un enregistrement"])
        UC13(["UC-13 Surveiller la qualité du signal"])
        UC14(["UC-14 Visualiser le signal"])
        UC15(["UC-15 Analyser le signal"])
        UC16(["UC-16 Réaliser la calibration 📝"])
        UC17(["UC-17 Interrompre / recommencer la calibration"])
        UC18(["UC-18 Consulter les calibrations et choisir le modèle"])
        UC19(["UC-19 Être averti d'une recalibration [D-33]"])
        UC20(["UC-20 Superviser la chaîne en direct"])
        UC21(["UC-21 Activer / reprendre les commandes 📝 [D-29]"])
        UC22(["UC-22 Suspendre les commandes 📝"])
        UC23(["UC-23 Commander un système cible par intention 📝"])
        UC24(["UC-24 Consulter l'état des systèmes cibles"])
        UC25(["UC-25 Confirmer une commande sensible [D-24]"])
        UC26(["UC-26 Tester manuellement un système cible 📝 [D-27]"])
        UC27(["UC-27 Consulter la correspondance"])
        UC28(["UC-28 Modifier la correspondance 📝 [D-20]"])
        UC29(["UC-29 Consulter le seuil de confiance"])
        UC30(["UC-30 Modifier le seuil 📝 [D-23]"])
        UC31(["UC-31 Déclencher l'arrêt d'urgence 📝 [D-19]"])
        UC32(["UC-32 Gérer un incident 📝"])
        UC33(["UC-33 Consulter le journal"])
        UC34(["UC-34 Recevoir une alerte [D-35]"])
        UC35(["UC-35 Consulter l'état technique des services"])
        UC36(["UC-36 Créer une session"])
        UC37(["UC-37 Conduire une session 📝 [D-37]"])
        UC38(["UC-38 Suivre le protocole d'essais"])
        UC39(["UC-39 Consulter les mesures"])
        UC40(["UC-40 Comparer des sessions"])
        UC41(["UC-41 Rejouer une session"])
        UC42(["UC-42 Détecter une intention"]):::interne
        UC43(["UC-43 Appliquer les garde-fous"]):::interne
        UC44(["UC-44 Transmettre la commande et recevoir le résultat"]):::interne
        UC45(["UC-45 Vérifier le consentement"]):::interne
        UC46(["UC-46 Enregistrer les données de session"]):::interne
        HM1(["Piloter l'interface par intentions<br/>hors MVP [D-25]"]):::horsmvp
        HM2(["Commander un robot / drone simulé<br/>hors MVP [D-11]"]):::horsmvp
    end
    E["⚙️ Casque EEG"]
    J["⚙️ Jeu de données EEG public"]
    T["⚙️ Système cible<br/><i>abstrait</i>"]
    O["⚙️ Ordinateur"]
    OC["⚙️ Objet connecté<br/>[D-05]"]
    R["⚙️ Robot / drone simulé<br/>[D-11]"]
    %% Généralisations d'acteurs
    U --> PA
    A --> PA
    X --> PA
    AD --> PA
    O --> T
    OC --> T
    R -.-> T
    %% Associations
    PA --- UC01
    PA --- UC03
    PA --- UC04
    PA --- UC27
    PA --- UC29
    U --- UC05
    U --- UC06
    U --- UC07
    U --- UC10
    U --- UC12
    U --- UC13
    U --- UC14
    U --- UC16
    U --- UC18
    U --- UC20
    U --- UC21
    U --- UC22
    U --- UC23
    U --- UC25
    U --- UC31
    U --- UC32
    U --- UC34
    A --- UC13
    A --- UC20
    A --- UC21
    A --- UC22
    A --- UC24
    A --- UC25
    A --- UC31
    A --- UC32
    A --- UC33
    A --- UC34
    X --- UC05
    X --- UC12
    X --- UC14
    X --- UC18
    X --- UC20
    X --- UC24
    X --- UC26
    X --- UC33
    X --- UC35
    X --- UC36
    X --- UC37
    X --- UC38
    X --- UC39
    X --- UC41
    AD --- UC02
    AD --- UC35
    P --- UC37
    P --- UC38
    U -.- HM1
    U -.- HM2
    UC12a --- E
    UC13 --- E
    UC16 --- E
    UC23 --- E
    UC12c --- J
    UC23 --- T
    UC24 --- T
    UC26 --- T
    UC31 --- T
    %% Spécialisations de cas
    UC12a --> UC12
    UC12b --> UC12
    UC12c --> UC12
    %% include
    UC02 -. "«include»" .-> UC01
    UC05 -. "«include»" .-> UC01
    UC11 -. "«include»" .-> UC45
    UC16 -. "«include»" .-> UC05
    UC16 -. "«include»" .-> UC13
    UC16 -. "«include»" .-> UC45
    UC21 -. "«include»" .-> UC13
    UC21 -. "«include»" .-> UC24
    UC23 -. "«include»" .-> UC42
    UC23 -. "«include»" .-> UC43
    UC23 -. "«include»" .-> UC44
    UC26 -. "«include»" .-> UC44
    UC36 -. "«include»" .-> UC05
    UC36 -. "«include»" .-> UC45
    UC37 -. "«include»" .-> UC46
    UC41 -. "«include»" .-> UC12c
    %% extend
    UC08 -. "«extend» veut le retirer" .-> UC07
    UC09 -. "«extend» suppression" .-> UC08
    UC11 -. "«extend» export" .-> UC10
    UC11 -. "«extend» export" .-> UC39
    UC15 -. "«extend» analyse" .-> UC14
    UC17 -. "«extend» interruption" .-> UC16
    UC19 -. "«extend» calibration ancienne" .-> UC20
    UC25 -. "«extend» commande sensible" .-> UC23
    UC28 -. "«extend» modif. autorisée" .-> UC27
    UC30 -. "«extend» modif. autorisée" .-> UC29
    UC34 -. "«extend» qualité insuffisante" .-> UC13
    UC34 -. "«extend» événement" .-> UC20
    UC32 -. "«extend» action requise" .-> UC34
    UC22 -. "«extend» gravité" .-> UC32
    UC31 -. "«extend» danger" .-> UC32
    UC21 -. "«extend» incident résolu" .-> UC32
    UC38 -. "«extend» expérimentation" .-> UC37
    UC40 -. "«extend» comparaison" .-> UC39
    classDef interne stroke-dasharray: 4 3,fill:#f2f2f2
    classDef horsmvp stroke-dasharray: 2 4,fill:#ffffff,color:#777777
```

---

Descriptions des cas, acteurs et relations : voir le catalogue dans `diag-02a-cas-utilisation-par-acteur.md`, section 2.
