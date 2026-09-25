# DIAG-4 — Diagramme de composants

| | |
|---|---|
| **Réf.** | DIAG-4 (Planning MVP, S2, mode B — D-46) |
| **Sources** | Analyse des composants v1.0 (25/09/2026) · Cahier des charges § 10 · Spécification § 4 · Planning § 2 · Décisions D-54 à D-62 · DIAG-1, DIAG-2, DIAG-3 |
| **Version** | 1.1 — 25 septembre 2026 (relecture : détections soumises par l'orchestrateur, canal de l'arrêt d'urgence, passerelle temps réel) |

## Rôle du diagramme

Il montre **les blocs logiciels** de CortexOS IA (composants), **ce que chacun offre et utilise** (interfaces) et **comment ils communiquent** (protocoles). Il répond à : *de quoi le système est-il fait, et qui parle à qui ?*

Il ne montre ni les classes (DIAG-3), ni les tables, ni les routes une par une, ni les écrans, ni les bibliothèques (BrainFlow, MNE, scikit-learn…), ni le matériel comme composant logiciel.

**Décisions qui cadrent ce diagramme :** monolithe modulaire FastAPI · agent ordinateur séparé · Core en Python pur, indépendant de FastAPI (D-62) · interfaces `ISourceEEG` et `IConnecteur` · orchestrateur distinct du Core (D-54) · agent en WebSocket local (D-55) · lampe simulée en appel direct (D-56) · arrêt d'urgence par raccourci global (D-57) · PostgreSQL + fichiers (D-58) · entraînement en tâche de fond (D-59) · authentification minimale en S25 (D-60) · contrôle qualité séparé (D-61).

## Notation

| Élément | Représentation | Sens |
|---|---|---|
| Composant | rectangle (📦) | Bloc logiciel avec une responsabilité claire |
| Interface **fournie** | cercle `○ INom` relié au composant par un trait plein | Ce que le composant **offre** aux autres (« sucette » UML) |
| Interface **requise** | flèche pointillée d'un composant vers le cercle | Ce que le composant **utilise** (« demi-lune » UML) |
| «datastore» | cylindre | Stockage |
| Système externe | rectangle gris, hors du cadre CortexOS IA | Matériel ou logiciel qui n'est pas construit dans le projet |
| Pointillés | trait ou bordure pointillés | À confirmer (D-05) ou extension (D-11) |
| Libellé d'une liaison | texte sur la flèche | Protocole, quand la liaison sort du processus du backend |

Tout ce qui est dans le cadre **Backend** tourne dans **un seul processus** (monolithe) : les liaisons internes sont des appels de fonctions Python. Seules les liaisons qui **sortent** du backend ont un protocole réseau.

---

## 1. Diagramme principal (niveau 1)

```mermaid
---
config:
  layout: elk
---
flowchart LR
    %% ===== Systèmes externes (gauche) =====
    E1["🌐 Navigateur"]:::ext
    E2["🧠 Casque EEG<br/>[D-04]"]:::ext
    E3["📁 Jeu de données<br/>EEG public"]:::ext

    subgraph CX["CortexOS IA"]
        C1["📦 C1 Application Web<br/><i>Next.js / TypeScript</i>"]

        subgraph BE["Backend — monolithe modulaire (FastAPI, un seul processus)"]
            C2["📦 C2 API REST"]
            C3["📦 C3 Passerelle temps réel"]
            C4["📦 C4 Orchestrateur<br/>de la chaîne"]
            MOD["📦 Modules métier<br/>C5 Accès et comptes · C6 Profils et consentement<br/>C7 Sessions et mesures · C8 Journal et alertes<br/>C9 Paramètres de sûreté · C10 Persistance"]
            subgraph EEG["Traitement EEG"]
                C11["📦 C11 Acquisition"]
                C12["📦 C12 Contrôle qualité"]
                C13["📦 C13 Prétraitement"]
            end
            subgraph IA["IA"]
                C14["📦 C14 Détection d'intention"]
                C15["📦 C15 Calibration<br/>et entraînement"]
                C16["📦 C16 Gestion des modèles"]
            end
            C17["📦 C17 CortexOS Core<br/><i>Python pur</i>"]
            C19["📦 C19–C20 Registre des cibles<br/>et connecteurs"]

            I_REST(("IApiRest"))
            I_WS(("IFluxTempsRéel"))
            I_SRC(("ISourceEEG"))
            I_QUA(("IQualitéSignal"))
            I_SIG(("ISignalTraité"))
            I_DET(("IDétecteur"))
            I_CORE(("IContrôleCore"))
            I_CON(("IConnecteur"))
            I_PER(("IPersistance"))
            I_FIC(("IStockageFichiers"))
        end

        subgraph ST["Stockage"]
            C21[("🗄️ C21 Base de données<br/>PostgreSQL")]
            C22[("🗄️ C22 Stockage de fichiers<br/>signal · modèles · exports")]
        end

        subgraph PS["Programmes séparés"]
            C18["📦 C18 Arrêt d'urgence<br/><i>raccourci global</i>"]
            C23["📦 C23 Agent ordinateur"]
            C24["📦 C24 Lampe simulée<br/>«simulator»"]
            I_AG(("IAgentOrdinateur"))
        end
    end

    %% ===== Systèmes externes (droite) =====
    E4["💻 Système d'exploitation<br/>[D-06]"]:::ext
    E5["Broker MQTT<br/>[D-05]"]:::extopt
    E6["ESP32<br/>[D-05]"]:::extopt
    E7["Simulateur robot ROS 2<br/>[D-11]"]:::extopt

    %% ===== Interface Web =====
    E1 -- "exécute" --> C1
    C2 --- I_REST
    C3 --- I_WS
    C1 -. "HTTP / REST (JSON)" .-> I_REST
    C1 -. "WebSocket" .-> I_WS

    %% ===== Chaîne EEG =====
    E2 -- "BrainFlow<br/>liaison [D-04]" --> C11
    E3 -- "téléchargé à l'avance" --> C22
    C11 --- I_SRC
    C12 --- I_QUA
    C13 --- I_SIG
    C14 --- I_DET
    C4 -.-> I_SRC
    C12 -.-> I_SRC
    C13 -.-> I_SRC
    C14 -.-> I_SIG
    C15 -.-> I_SIG
    C15 -.-> I_QUA
    C14 -. "modèle actif" .-> C16
    C15 -. "nouveau modèle" .-> C16
    C4 -.-> I_DET

    %% ===== Core =====
    C17 --- I_CORE
    C4 -. "détections" .-> I_CORE
    C2 -.-> I_CORE
    C17 -.-> I_QUA
    C17 -.-> C19
    C18 -. "ordre d'arrêt<br/>canal local [À DÉFINIR]" .-> I_CORE

    %% ===== Cibles =====
    C19 --- I_CON
    C23 --- I_AG
    C19 -. "WebSocket local" .-> I_AG
    C23 -- "API du système" --> E4
    C19 -- "appel direct" --> C24
    C19 -. "MQTT" .-> E5
    E5 -.-> E6
    C19 -. "ROS 2" .-> E7

    %% ===== Modules, événements, stockage =====
    C2 -.-> MOD
    C4 -. "enregistre la session" .-> MOD
    C17 -. "événements" .-> MOD
    C3 -. "abonnement aux événements" .-> MOD
    C3 -. "qualité" .-> I_QUA
    C3 -. "signal à afficher" .-> I_SIG
    MOD --- I_PER
    C22 --- I_FIC
    I_PER -- "SQL" --> C21
    C11 -.-> I_FIC
    C16 -.-> I_FIC
    MOD -.-> I_FIC

    classDef ext fill:#eeeeee,stroke:#777777,color:#222222
    classDef extopt fill:#ffffff,stroke:#999999,stroke-dasharray: 4 3,color:#555555
```

---

## 2. Sous-diagrammes (niveau 2)

### 2a — Backend : points d'entrée et modules métier

```mermaid
flowchart LR
    C1["📦 C1 Application Web"]

    subgraph BE["Backend (FastAPI)"]
        C2["📦 C2 API REST"]
        C3["📦 C3 Passerelle temps réel"]
        C4["📦 C4 Orchestrateur de la chaîne"]
        C5["📦 C5 Accès et comptes<br/><i>S25 · D-60</i>"]
        C6["📦 C6 Profils et consentement"]
        C7["📦 C7 Sessions et mesures"]
        C8["📦 C8 Journal et alertes"]
        C9["📦 C9 Paramètres de sûreté<br/>et correspondance"]
        C10["📦 C10 Persistance"]

        I_REST(("IApiRest"))
        I_WS(("IFluxTempsRéel"))
        I_HEALTH(("IÉtatServices"))
        I_CH(("IChaîne"))
        I_AUTH(("IAuth"))
        I_PROF(("IProfils ·<br/>IConsentement"))
        I_SES(("ISessions"))
        I_JOU(("IJournal"))
        I_EMI(("IÉmetteurÉvénements"))
        I_ABO(("IAbonnementÉvénements"))
        I_PAR(("IParamètresSûreté"))
        I_PER(("IPersistance"))
    end
    C21[("🗄️ PostgreSQL")]
    C22[("🗄️ Fichiers")]
    CORE["📦 C17 Core"]

    C2 --- I_REST
    C2 --- I_HEALTH
    C3 --- I_WS
    C4 --- I_CH
    C5 --- I_AUTH
    C6 --- I_PROF
    C7 --- I_SES
    C8 --- I_JOU
    C8 --- I_EMI
    C8 --- I_ABO
    C9 --- I_PAR
    C10 --- I_PER

    C1 -. "HTTP / REST" .-> I_REST
    C1 -. "WebSocket" .-> I_WS
    C2 -.-> I_AUTH
    C2 -.-> I_PROF
    C2 -.-> I_SES
    C2 -.-> I_PAR
    C2 -.-> I_JOU
    C2 -.-> I_CH
    C2 -. "activer, suspendre,<br/>confirmer" .-> CORE
    C3 -.-> I_ABO
    C4 -. "enregistre" .-> I_SES
    C4 -.-> I_EMI
    C7 -.-> I_PROF
    C7 -. "pause = suspension" .-> CORE
    C9 -. "trace l'auteur" .-> I_JOU
    C9 -.-> I_AUTH
    CORE -.-> I_EMI
    CORE -.-> I_PAR
    C5 -.-> I_PER
    C6 -.-> I_PER
    C7 -.-> I_PER
    C8 -.-> I_PER
    C9 -.-> I_PER
    I_PER -- "SQL" --> C21
    C6 -. "suppression" .-> C22
    C7 -. "enregistrements, exports" .-> C22
```

### 2b — Traitement EEG et IA

```mermaid
flowchart LR
    E2["🧠 Casque EEG"]:::ext
    C22[("🗄️ Fichiers<br/>enregistrements · modèles")]

    subgraph EEG["Traitement EEG"]
        subgraph C11["📦 C11 Acquisition"]
            S1["Source casque<br/><i>BrainFlow</i>"]
            S2["Source simulée<br/><i>carte synthétique BrainFlow</i>"]
            S3["Source enregistrement<br/><i>lecture de fichier</i>"]
        end
        I_SRC(("ISourceEEG"))
        C12["📦 C12 Contrôle qualité"]
        I_QUA(("IQualitéSignal"))
        C13["📦 C13 Prétraitement<br/><i>filtrage</i>"]
        I_SIG(("ISignalTraité"))
    end

    subgraph IA["IA"]
        subgraph C14["📦 C14 Détection d'intention"]
            D1["Détecteur IA<br/><i>fenêtres → CSP → classifieur</i>"]
            D2["Détecteur de test<br/><i>signalé comme simulé</i>"]
        end
        I_DET(("IDétecteur"))
        C15["📦 C15 Calibration et entraînement<br/><i>tâche de fond · D-59</i>"]
        I_CAL(("ICalibration"))
        C16["📦 C16 Gestion des modèles"]
        I_MOD(("IRegistreModèles"))
    end

    E2 -- "liaison [D-04]" --> S1
    S3 -.-> C22
    S1 --- I_SRC
    S2 --- I_SRC
    S3 --- I_SRC
    C12 --- I_QUA
    C13 --- I_SIG
    D1 --- I_DET
    D2 --- I_DET
    C15 --- I_CAL
    C16 --- I_MOD

    C12 -.-> I_SRC
    C13 -.-> I_SRC
    D1 -.-> I_SIG
    D1 -. "modèle actif" .-> I_MOD
    C15 -.-> I_SIG
    C15 -.-> I_QUA
    C15 -. "nouveau modèle" .-> I_MOD
    C16 -.-> C22

    classDef ext fill:#eeeeee,stroke:#777777,color:#222222
```

Les trois **sources** réalisent la même interface `ISourceEEG` : le reste de la chaîne ne sait pas si le signal vient du casque, de la simulation ou d'un fichier. Même principe pour les deux **détecteurs** avec `IDétecteur`. Fenêtres, CSP et classifieur sont **internes** au Détecteur IA.

### 2c — CortexOS Core

```mermaid
flowchart LR
    C4["📦 C4 Orchestrateur"]
    C2["📦 C2 API REST"]
    C7["📦 C7 Sessions"]
    C15["📦 C15 Calibration"]
    C18["📦 C18 Arrêt d'urgence<br/><i>programme local · D-57</i>"]

    subgraph CORE["📦 C17 CortexOS Core — Python pur, sans FastAPI ni base de données"]
        ETAT["Gestionnaire d'état global<br/><i>7 états · reprise explicite</i>"]
        DEC["Moteur de décision<br/><i>correspondance · seuil</i>"]
        GF["Garde-fous<br/><i>état Actif · qualité · cible · délai · commande autorisée</i>"]
        CONF["Gestionnaire de confirmation<br/><i>commandes sensibles · expiration</i>"]
        ROUT["Routeur de commandes<br/><i>envoi · attente du résultat · échec après délai</i>"]
        DEC --> GF --> CONF --> ROUT
        ETAT -. "état courant" .-> GF
    end
    I_CORE(("IContrôleCore"))

    I_QUA(("IQualitéSignal"))
    I_PAR(("IParamètresSûreté"))
    I_REG(("IRegistreCibles"))
    I_EMI(("IÉmetteurÉvénements"))

    CORE --- I_CORE
    C2 -. "activer, suspendre, reprendre,<br/>confirmer, annuler" .-> I_CORE
    C7 -. "pause de session" .-> I_CORE
    C15 -. "état Calibration" .-> I_CORE
    C18 -. "arrêt prioritaire<br/>canal local [À DÉFINIR]" .-> I_CORE
    C4 -. "soumettre une détection" .-> I_CORE
    I_CORE -. "transmise au" .-> DEC

    GF -.-> I_QUA
    DEC -.-> I_PAR
    GF -.-> I_PAR
    CONF -.-> I_PAR
    GF -.-> I_REG
    ROUT -.-> I_REG
    CORE -. "décisions, commandes,<br/>résultats, changements d'état" .-> I_EMI
```

Le Core **ne lit pas lui-même le détecteur** : c'est l'Orchestrateur (C4, D-54) qui lit `IDétecteur` et **soumet** chaque détection au Core par `IContrôleCore`. Le Core ne fait que décider.

Le Core **n'écrit pas lui-même** dans le journal ni dans la base : il **émet** des événements (`IÉmetteurÉvénements`) que le module Journal enregistre. C'est ce qui le garde indépendant de FastAPI et de PostgreSQL, donc testable seul.

### 2d — Intégration des systèmes cibles

```mermaid
flowchart LR
    CORE["📦 C17 Core<br/>(Routeur)"]

    subgraph INT["Intégration des systèmes cibles"]
        C19["📦 C19 Registre des systèmes cibles<br/><i>disponibilité · liste fermée · commandes sensibles</i>"]
        I_REG(("IRegistreCibles"))
        I_CON(("IConnecteur"))
        K1["📦 Connecteur ordinateur"]
        K2["📦 Connecteur lampe"]
        K3["📦 Connecteur MQTT<br/>[D-05]"]:::opt
        K4["📦 Connecteur robot ROS 2<br/>[D-11]"]:::opt
    end

    subgraph PS["Programmes séparés"]
        C23["📦 C23 Agent ordinateur"]
        I_AG(("IAgentOrdinateur"))
        C24["📦 C24 Lampe simulée<br/>«simulator»"]
        I_SIMU(("ICibleSimulée"))
    end

    E4["💻 Système d'exploitation<br/>[D-06]"]:::ext
    E5["Broker MQTT"]:::extopt
    E6["ESP32"]:::extopt
    E7["Simulateur robot ROS 2"]:::extopt

    C19 --- I_REG
    CORE -.-> I_REG
    K1 --- I_CON
    K2 --- I_CON
    K3 --- I_CON
    K4 --- I_CON
    C19 -. "1..* connecteurs" .-> I_CON

    C23 --- I_AG
    C24 --- I_SIMU
    K1 -. "WebSocket local (D-55)" .-> I_AG
    C23 -- "API du système" --> E4
    K2 -. "appel direct (D-56)" .-> I_SIMU
    K3 -. "MQTT" .-> E5
    E5 -.-> E6
    K4 -. "ROS 2" .-> E7

    classDef opt stroke-dasharray: 4 3
    classDef ext fill:#eeeeee,stroke:#777777,color:#222222
    classDef extopt fill:#ffffff,stroke:#999999,stroke-dasharray: 4 3,color:#555555
```

Ajouter un type de cible = ajouter un **connecteur** qui réalise `IConnecteur`, **sans modifier le Core** (F-25).

---

## 3. Liste des composants

| # | Composant | Sous-système | Responsabilité | Statut |
|---|---|---|---|---|
| C1 | Application Web | Interface | Tous les écrans ; respecte la charte et l'accessibilité | Confirmé |
| C2 | API REST | Backend | Exposer les opérations, contrôler l'accès | Confirmé |
| C3 | Passerelle temps réel | Backend | Pousser état, qualité, signal, détections, décisions, commandes, résultats, alertes | Confirmé |
| C4 | Orchestrateur de la chaîne | Backend | Boucle temps réel source → qualité → prétraitement → détection → Core ; enregistrement dans la session | Décidé (D-54) |
| C5 | Accès et comptes | Backend | Authentification, comptes, rôles | Décidé (D-60) · détail D-09 |
| C6 | Profils et consentement | Backend | Profils, consentements, suppression des données | Déduit |
| C7 | Sessions et mesures | Backend | Sessions, essais, mesures, rejeu, export | Déduit |
| C8 | Journal et alertes | Backend | Journal unique, gravité, alertes, diffusion | Confirmé |
| C9 | Paramètres de sûreté et correspondance | Backend | Seuil, délais, correspondance ; trace l'auteur des modifications | Déduit · rattachement D-20, D-23 |
| C10 | Persistance | Backend | Accès à PostgreSQL pour tous les modules | Décidé (D-58) |
| C11 | Acquisition | Traitement EEG | Lire une source interchangeable (casque, simulation, enregistrement) | Confirmé |
| C12 | Contrôle qualité | Traitement EEG | Qualité globale et par canal, perte du signal | Décidé (D-61) |
| C13 | Prétraitement | Traitement EEG | Filtrage, réduction du bruit | Confirmé |
| C14 | Détection d'intention | IA | Inférence (détecteur IA) ou détecteur de test | Confirmé |
| C15 | Calibration et entraînement | IA | Essais guidés, entraînement en tâche de fond, évaluation | Décidé (D-59) |
| C16 | Gestion des modèles | IA | Versions, modèle actif | Déduit |
| C17 | CortexOS Core | Core | État global, décision, garde-fous, confirmation, routage | Confirmé · Python (D-62) |
| C18 | Arrêt d'urgence | Programme séparé | Raccourci clavier global → ordre d'arrêt au Core par un canal local, sans passer par l'interface Web | Décidé (D-57) · canal local `[À DÉFINIR]` · déclencheur D-19 |
| C19–C20 | Registre des cibles et connecteurs | Intégration | Cibles, disponibilité, liste fermée ; traduction des commandes | Confirmé · MQTT D-05, robot D-11 |
| C21 | Base de données | Stockage | Données métier | Décidé : PostgreSQL (D-58) |
| C22 | Stockage de fichiers | Stockage | Signal enregistré, modèles, exports | Décidé (D-58) |
| C23 | Agent ordinateur | Programme séparé | Exécuter les commandes sur le système d'exploitation | Confirmé · liaison D-55 |
| C24 | Lampe simulée | Programme séparé | Cible simulée | Confirmé · appel direct D-56 |

**Systèmes externes :** E1 Navigateur · E2 Casque EEG (D-04) · E3 Jeu de données EEG public · E4 Système d'exploitation (D-06) · E5 Broker MQTT et E6 ESP32 (D-05, à confirmer) · E7 Simulateur robot ROS 2 (extension, D-11).

## 4. Interfaces principales

| Interface | Fournie par | Utilisée par | Protocole |
|---|---|---|---|
| `IApiRest` | C2 | C1 | HTTP / REST (JSON) |
| `IFluxTempsRéel` | C3 | C1 | WebSocket |
| `IÉtatServices` | C2 | C1, outils | HTTP (`GET /api/health`) |
| `IChaîne` | C4 | C2 | en processus |
| `ISourceEEG` | C11 (3 sources) | C4, C12, C13 | en processus |
| `IQualitéSignal` | C12 | C17, C15, C3 | en processus |
| `ISignalTraité` | C13 | C14, C15, C3 | en processus |
| `IDétecteur` | C14 (IA, test) | C4 (qui soumet les détections au Core) | en processus |
| `ICalibration` | C15 | C2 | en processus |
| `IRegistreModèles` | C16 | C14, C15 | en processus |
| `IContrôleCore` (soumettre une détection, activer, suspendre, reprendre, état sûr, confirmer, annuler, lire l'état) | C17 | C4, C2, C7, C15, C18 | en processus ; C18 : canal local `[À DÉFINIR]` |
| `IParamètresSûreté` | C9 | C17, C2 | en processus |
| `IRegistreCibles` | C19 | C17, C2 | en processus |
| `IConnecteur` | connecteurs (C20) | C19 | en processus |
| `IAgentOrdinateur` | C23 | connecteur ordinateur | WebSocket local (D-55) |
| `ICibleSimulée` | C24 | connecteur lampe | appel direct (D-56) |
| `IÉmetteurÉvénements` · `IAbonnementÉvénements` · `IJournal` | C8 | C17, C4, C11, C12, C15 · C3 · C2, C6, C7, C9 | en processus |
| `IAuth` · `IProfils` · `IConsentement` · `ISessions` | C5 · C6 · C6 · C7 | C2, C9 · C2, C7, C15 · C2, C7, C15 · C2, C4 | en processus |
| `IPersistance` | C10 | modules métier | en processus → SQL (PostgreSQL) |
| `IStockageFichiers` | C22 | C6, C7, C11, C16 | système de fichiers |

## 5. Flux principaux

1. **Acquisition** : casque / simulation / enregistrement → C11 → C12 et C13 → C14 → détection.
2. **Décision** : détection → C17 (moteur de décision → garde-fous → confirmation si sensible) → commande.
3. **Exécution** : C17 routeur → C19 → connecteur → C23 (→ OS) ou C24 → résultat → C17 → événement → C8.
4. **Supervision** : C17, C12, C13, C8 → C3 → C1 (le Core passe par les événements, jamais directement par le Web).
5. **Calibration** : C1 → C2 → C15 (consentement, qualité, Core en état Calibration) → C11 → C13 → entraînement → C16 → fichiers.
6. **Session d'expérimentation** : C1 → C2 → C7 → C4 démarre la chaîne et enregistre → mesures → export.
7. **Arrêt d'urgence** : C18 → C17 (suspendu / état sûr) → C8 → C3 → C1, **sans dépendre de C1** pour l'ordre d'arrêt.

## 6. Points encore ouverts

- **D-04** : casque, donc liaison physique et pilote éventuel.
- **D-05** : objet connecté réel (MQTT, ESP32) dans le MVP ou non.
- **D-06** : système d'exploitation, donc API utilisée par l'agent.
- **D-09** : authentification précise, droits par rôle, conservation.
- **D-11** : robot et drone.
- **D-19** : qui peut déclencher l'arrêt d'urgence.
- **Canal de l'arrêt d'urgence** : C18 est un programme séparé ; il ne peut donc pas appeler le Core « en direct ». Il faut choisir un canal local vers le backend (par exemple une route HTTP réservée à `localhost`, ou un WebSocket local comme l'agent). `[À DÉFINIR]` — à trancher avec D-19, avant S9.
- **D-20 / D-23** : paramètres et correspondance globaux ou par profil.
- **D-32** : que fait le Core si l'interface Web est perdue pendant une session (chien de garde ?).
