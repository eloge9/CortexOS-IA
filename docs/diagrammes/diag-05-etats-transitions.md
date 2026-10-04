# DIAG-5 — Diagrammes d'états-transitions

| | |
|---|---|
| **Réf.** | DIAG-5 (Planning MVP, S2, mode A — D-96) |
| **Sources** | Analyse des états fournie par Eloge · Spécification fonctionnelle 4.2 à 4.7 · DIAG-3 (classes du domaine) · Registre des décisions D-63 à D-71 |
| **Version** | 1.0 — 25 septembre 2026 |
| **Statut** | Validé par Eloge (25/09/2026) |

## Rôle de ces diagrammes

Un diagramme d'états-transitions décrit **le cycle de vie d'un objet** : les situations stables où il peut se trouver (**états**), les **événements** qui le font changer d'état, et les **conditions** (gardes, entre crochets) qui autorisent le changement.

Le diagramme de classes (DIAG-3) dit *ce qui existe* ; ces diagrammes disent *comment ça évolue dans le temps*. Ils serviront directement au code : chaque diagramme deviendra une énumération d'états et une fonction qui refuse les transitions non prévues.

Cinq diagrammes (D-63) :

| N° | Objet | Importance |
|---|---|---|
| ① | État global de CortexOS | Essentiel : c'est lui qui décide si une commande peut partir |
| ② | Commande, depuis la détection (D-64) | Essentiel : chaîne de sûreté détection → exécution |
| ③ | Session | Utile |
| ④ | Calibration (avec sous-état Entraînement, D-65) | Utile |
| ⑤ | Source de signal | Utile |

### Conventions de lecture

| Notation | Sens |
|---|---|
| `[*]` → état | État initial |
| état → `[*]` | État final |
| `événement [garde]` | La transition a lieu quand l'événement arrive **et** que la garde est vraie |
| **⚠ … [D-xx]** | Comportement **non tranché** : la transition est dessinée pour montrer la question, mais elle n'est pas décidée (D-66) |
| Note | Règle importante à respecter dans le code |

> Mermaid (`stateDiagram-v2`) ne permet pas de dessiner une flèche en pointillés. Les transitions non tranchées sont donc repérées par le symbole **⚠** et leur référence `[D-xx]` dans le libellé.

---

## ① État global de CortexOS

**Objet :** la plateforme dans son ensemble (porté par l'orchestrateur, D-54).
**Question à laquelle il répond :** « Est-ce qu'une commande a le droit de partir maintenant ? » → **uniquement dans l'état Actif.**

```mermaid
stateDiagram-v2
    direction LR
    state "Arrêté" as Arrete
    state "Préparation" as Preparation
    state "Calibration" as Calibration
    state "Prêt" as Pret
    state "Actif" as Actif
    state "Suspendu" as Suspendu
    state "État sûr" as EtatSur

    [*] --> Arrete
    Arrete --> Preparation : T1 démarrer [aucun modèle exploitable]
    Arrete --> Pret : T2 démarrer [modèle exploitable disponible]
    Preparation --> Calibration : T3 lancer la calibration [qualité OK et consentement]
    Preparation --> Pret : T4 sélectionner un modèle existant
    Calibration --> Pret : T5 calibration exploitable
    Calibration --> Preparation : T6 calibration insuffisante ou interrompue
    Pret --> Actif : T7 activer les commandes (action explicite)
    Actif --> Suspendu : T8 suspendre ou pause de session
    Suspendu --> Actif : T9 reprendre (action explicite)
    Pret --> Calibration : T10 recalibrer
    Suspendu --> Calibration : T11 recalibrer
    Actif --> EtatSur : T12 perte du signal ou erreur
    Suspendu --> EtatSur : T13 perte du signal ou erreur
    Preparation --> EtatSur : ⚠ T14 perte du signal [D-67]
    Actif --> EtatSur : ⚠ T15 arrêt d'urgence [D-19] ou interface perdue [D-32]
    EtatSur --> Suspendu : T16 incident résolu → Suspendu (D-29)
    Pret --> Arrete : T17 arrêter
    Suspendu --> Arrete : T17 arrêter
    EtatSur --> Arrete : T17 arrêter
    Arrete --> [*]

    note right of Actif
        Seul état où une commande
        peut être exécutée
    end note
    note left of EtatSur
        Aucune commande possible.
        Une reconnexion ne réactive
        jamais les commandes
    end note
```

### Lecture

| Élément | Ce qu'il faut retenir |
|---|---|
| **Actif** | Le seul état où l'orchestrateur transmet une commande. Dans tous les autres états, une intention détectée est affichée mais rejetée (voir ②). |
| **Prêt → Actif** | Toujours par une **action explicite** de l'utilisateur ou de l'accompagnant (parcours B). Le système ne s'active jamais tout seul. |
| **Suspendu** | Différent de *Prêt* : on garde la session et le modèle, on coupe seulement les commandes. La pause de session y mène aussi. |
| **État sûr** | État de repli après un incident. Après résolution, on passe en *Suspendu* (D-29) ; la reprise reste explicite et n'est possible que si signal, qualité, modèle et cible sont prêts (D-91). |
| **Qualité insuffisante** | Ne change **pas** l'état global : elle est signalée (diagramme ⑤) et la chaîne rejette les détections pendant ce temps. |
| **T14** | En *Préparation*, aucune commande n'est active : faut-il vraiment passer en État sûr, ou simplement revenir à *Arrêté* ? → D-67. |

---

## ② Cycle de vie d'une commande (depuis la détection)

**Objet :** une intention détectée qui devient (ou non) une commande. Option (a) retenue (D-64), fidèle à la spécification 4.5.
**Question à laquelle il répond :** « Qu'arrive-t-il à une intention entre la détection et l'action sur la cible ? »

```mermaid
stateDiagram-v2
    direction TB
    state "Détectée" as Detectee
    state "Rejetée" as Rejetee
    state "Acceptée" as Acceptee
    state "En attente de confirmation" as Attente
    state "Confirmée" as Confirmee
    state "Annulée" as Annulee
    state "Expirée" as Expiree
    state "Envoyée" as Envoyee
    state "Exécutée" as Executee
    state "Échouée" as Echouee

    [*] --> Detectee : C1 intention détectée par le modèle
    [*] --> Envoyee : ⚠ C0 commande manuelle de test [D-27]
    Detectee --> Rejetee : C2 [confiance < seuil ou état global ≠ Actif ou qualité insuffisante]
    Detectee --> Acceptee : C3 [confiance ≥ seuil et état Actif et qualité OK]
    Acceptee --> Envoyee : C4 [commande non sensible]
    Acceptee --> Attente : ⚠ C5 [commande sensible] [D-24]
    Attente --> Confirmee : ⚠ C6 confirmation reçue [D-24]
    Attente --> Annulee : C7 refus ou suspension
    Attente --> Expiree : ⚠ C8 délai dépassé [D-31]
    Confirmee --> Envoyee : C9 transmettre au connecteur
    Envoyee --> Executee : C10 résultat positif de la cible
    Envoyee --> Echouee : C11 erreur renvoyée par la cible
    Envoyee --> Echouee : ⚠ C12 pas de réponse dans le délai [D-68]
    Acceptee --> Echouee : ⚠ C13 cible indisponible [D-68]

    Rejetee --> [*]
    Annulee --> [*]
    Expiree --> [*]
    Executee --> [*]
    Echouee --> [*]

    note right of Rejetee
        Toujours journalisée
        (avec la raison du rejet)
    end note
```

### Lecture

| Élément | Ce qu'il faut retenir |
|---|---|
| **C2 / C3** | C'est ici que les **trois garde-fous** s'appliquent : seuil de confiance, état global *Actif* (diagramme ①), qualité du signal (diagramme ⑤). Il suffit d'un seul « non » pour rejeter. |
| **Rejetée** | Ce n'est pas une erreur : c'est le fonctionnement normal quand le système n'est pas sûr. Chaque rejet est journalisé avec sa raison (utile pour les mesures de l'expérimentateur). |
| **Commande sensible** | Passe par *En attente de confirmation*. Confirmation par l'accompagnant ou par l'intention EEG « oui » (D-24) ; délai réglable par profil (D-31). |
| **Cinq états finaux** | Rejetée, Annulée, Expirée, Exécutée, Échouée : une commande finit **toujours** dans l'un d'eux, ce qui garantit une trace complète dans le journal. |
| **C0** | Le test manuel d'une cible sans EEG (D-27) entrerait directement en *Envoyée*, sans passer par la détection. |

---

## ③ Session

**Objet :** une session d'utilisation ou d'expérimentation (classe `Session`, attribut `type`, D-50).

```mermaid
stateDiagram-v2
    direction LR
    state "Créée" as Creee
    state "En cours" as EnCours
    state "En pause" as EnPause
    state "Terminée" as Terminee
    state "Interrompue" as Interrompue

    [*] --> Creee : S1 créer la session (profil, type, protocole)
    Creee --> EnCours : S2 démarrer [source connectée]
    EnCours --> EnPause : S3 mettre en pause
    EnPause --> EnCours : S4 reprendre
    EnCours --> Terminee : S5 terminer
    EnPause --> Terminee : S6 terminer
    EnCours --> Interrompue : S7 erreur bloquante ou arrêt d'urgence
    EnCours --> EnPause : S8 perte du signal (D-95)
    EnCours --> EnPause : ⚠ S9 durée maximale atteinte [D-37]
    Interrompue --> EnCours : ⚠ S10 reprendre une session interrompue ? [D-71]
    Terminee --> [*]
    Interrompue --> [*]

    note right of Terminee
        Données conservées :
        journal, mesures, exports
    end note
```

### Lecture

| Élément | Ce qu'il faut retenir |
|---|---|
| **Pause ≠ Suspendu** | Mettre la session en pause suspend aussi les commandes (① T8), mais l'inverse n'est pas vrai : on peut suspendre les commandes sans mettre la session en pause. |
| **Terminée / Interrompue** | Toutes deux sont des fins de session ; la différence est la **cause** (volontaire ou incident), enregistrée dans le journal. |
| **D-95 / D-71** | Perte du signal → *En pause* automatiquement, période exclue des mesures (S8, D-95). Restent ouverts : reprise d'une session interrompue, enregistrement pendant la pause (D-71). |

---

## ④ Calibration

**Objet :** une calibration (série d'essais guidés puis entraînement du modèle). Sous-état *Entraînement* retenu (D-65). Chaque essai suit la règle `{xor}` de D-51.

```mermaid
stateDiagram-v2
    direction LR
    state "Non commencée" as NonCommencee
    state "En cours" as EnCours
    state "Entraînement" as Entrainement
    state "Exploitable" as Exploitable
    state "Insuffisante" as Insuffisante
    state "Interrompue" as Interrompue

    [*] --> NonCommencee
    NonCommencee --> NonCommencee : K1 lancer [qualité insuffisante] / refus affiché
    NonCommencee --> EnCours : K2 lancer [qualité OK et consentement]
    EnCours --> EnCours : K3 essai terminé [essais restants] / essai suivant
    EnCours --> Entrainement : K4 dernier essai terminé
    EnCours --> Interrompue : K5 arrêt par l'utilisateur ou perte du signal
    Entrainement --> Exploitable : ⚠ K6 [critère atteint] [D-10]
    Entrainement --> Insuffisante : ⚠ K7 [critère non atteint] [D-10]
    Entrainement --> Interrompue : ⚠ K8 erreur d'entraînement [D-69]
    Exploitable --> [*]
    Insuffisante --> [*]
    Interrompue --> [*]

    note right of Exploitable
        Le modèle produit peut être
        sélectionné (① T5 → Prêt)
    end note
```

### Lecture

| Élément | Ce qu'il faut retenir |
|---|---|
| **K1** | Auto-transition : on reste en *Non commencée*, mais le refus est affiché avec la raison (qualité). |
| **K3** | Auto-transition sur *En cours* : un essai après l'autre, sans changer d'état. |
| **Entraînement** | Le modèle est calculé à partir des essais. Cela peut prendre quelques secondes : l'interface doit montrer que le système travaille. |
| **D-10** | Le seuil qui distingue *Exploitable* d'*Insuffisante* n'est pas encore fixé (bloc casque). |
| **Lien avec ①** | Exploitable → ① T5 (Prêt) ; Insuffisante ou Interrompue → ① T6 (Préparation). |

---

## ⑤ Source de signal

**Objet :** la source EEG active (casque, carte synthétique BrainFlow ou fichier rejoué, interface `ISourceEEG`).

```mermaid
stateDiagram-v2
    direction LR
    state "Déconnectée" as Deconnectee
    state "Connexion en cours" as Connexion
    state "Connectée" as Connectee {
        state "Qualité suffisante" as QOk
        state "Qualité insuffisante" as QKo
        [*] --> QKo
        QKo --> QOk : Q1 qualité ≥ seuil
        QOk --> QKo : Q2 qualité < seuil
    }
    state "Signal perdu" as Perdu
    state "Fin d'enregistrement" as Fin

    [*] --> Deconnectee
    Deconnectee --> Connexion : E1 connecter
    Connexion --> Connectee : ⚠ E2 connexion établie [D-04]
    Connexion --> Deconnectee : E3 échec de connexion
    Connectee --> Perdu : E4 plus de données reçues
    Perdu --> Connexion : E5 reconnexion auto ou bouton (D-94)
    Perdu --> Deconnectee : E6 abandonner
    Connectee --> Fin : ⚠ E7 fin du fichier rejoué [D-70]
    Fin --> Deconnectee : E8 fermer la source
    Connectee --> Deconnectee : E9 déconnecter
```

### Lecture

| Élément | Ce qu'il faut retenir |
|---|---|
| **État composite** | *Connectée* contient deux sous-états : la qualité peut changer sans que la source se déconnecte. On démarre en *insuffisante* par prudence tant que la qualité n'est pas mesurée. |
| **Signal perdu → ①** | Déclenche ① T12/T13 (État sûr) si les commandes étaient actives. |
| **Fin d'enregistrement** | N'existe que pour un fichier rejoué (PhysioNet…). Que fait-on ensuite : boucler, s'arrêter ? → D-70. |
| **D-04** | La connexion réelle dépend du casque choisi (Bluetooth, USB, dongle). |

---

## Vérification croisée entre les diagrammes

| Événement | ① Global | ② Commande | ③ Session | ④ Calibration | ⑤ Source |
|---|---|---|---|---|---|
| Qualité insuffisante | pas de changement | C2 rejet | — | K1 refus | Q2 |
| Perte du signal | T12/T13 État sûr, ⚠ T14 | — | S8 En pause (D-95) | K5 Interrompue | E4 |
| Suspendre / pause | T8 Suspendu | C7 Annulée si en attente | S3 (pause) | — | — |
| Arrêt d'urgence | ⚠ T15 | action envoyée non rappelée (D-93) | S7 Interrompue | — | — |
| Calibration terminée | T5 ou T6 | — | — | Exploitable / Insuffisante | — |

Ce tableau vérifie qu'un même événement a un effet cohérent partout. Il servira de base aux tests de la machine à états.

## Objets écartés

Pas de diagramme pour le **système cible** ni pour l'**alerte** (D-63) : leurs états sont simples (disponible / indisponible ; émise / acquittée) et déjà visibles dans ② et dans la supervision. Ils pourront être ajoutés si le code montre qu'ils se compliquent.

## Points à définir

- **D-19** : qui peut déclencher l'arrêt d'urgence.
- **D-24** : liste des commandes sensibles (moyen de confirmation tranché).
- **D-27** : test manuel d'une cible sans EEG.
- **D-31** : valeur par défaut du délai de confirmation (réglable par profil, tranché).
- **D-32** : interface Web fermée pendant une session active.
- **D-37** : durée maximale d'une session.
- **D-04 / D-10** : casque ; critère de calibration exploitable.
- **D-67** : perte du signal en Préparation ; conditions d'activation.
- **D-68** : délai du résultat, cible indisponible (action envoyée jamais rappelée : D-93).
- **D-69** : erreur d'entraînement.
- **D-70** : fin d'un enregistrement rejoué (reconnexion : D-94).
- **D-71** : reprise d'une session interrompue, enregistrement en pause (perte du signal : D-95).
