# Contrat d'API v0 — CortexOS IA

| | |
|---|---|
| **Réf.** | API-0 (Planning MVP, S3, mode B — D-46) |
| **Sources** | Spécification fonctionnelle (§ 4, § 5 : F-01 à F-44, FW-01 à FW-51) · DIAG-3 (classes, énumérations) · DIAG-5 (états) · DIAG-6 (séquences) · ARCH-0 (§ 5, 7, 8) · Décisions D-24 à D-103 |
| **Version** | 0.1 — 4 octobre 2026 · **partie 1 sur 3** (conventions et objets) |
| **Statut** | Partie 1 à relire par Eloge · parties 2 (routes REST) et 3 (temps réel, canaux locaux) à venir |

## Rôle de ce document

Le contrat d'API dit **exactement** ce que l'interface Web (Next.js) peut demander au backend (FastAPI) et ce qu'elle reçoit. Il est écrit **avant le code** : chaque tranche vérifie que ses routes le respectent (Planning § 3.2).

| Partie | Contenu | Statut |
|---|---|---|
| **1** | Conventions communes · format des erreurs · objets échangés | **Ce document** |
| 2 | Routes REST `/api/v1/…`, domaine par domaine, avec traçabilité FW-xx → route | À venir |
| 3 | WebSocket `/ws/flux` · canaux locaux `/ws/agent` et `/api/local/arret-urgence` | À venir |

Les **propositions** (P-A1…) sont à valider par Eloge ; elles sont rassemblées en section 4. Tant qu'elles ne sont pas validées, elles restent « proposé ».

---

## 1. Conventions communes

### 1.1 Adresses (P2, D-101)

| Canal | Adresse | Usage |
|---|---|---|
| REST | `http://localhost:8000/api/v1/…` | Actions ponctuelles : lire, créer, modifier, déclencher |
| WebSocket | `ws://localhost:8000/ws/flux` | Tout ce qui change en continu ; **le backend pousse** |
| Local | `/api/local/…`, `/ws/agent` | Programmes du PC seulement (arrêt d'urgence, agent), jamais le navigateur |

`v1` est la **version du contrat** : si un jour un changement casse la compatibilité, on crée `/api/v2` sans casser `v1`.

### 1.2 Format des données (P-A1 à P-A3)

| Sujet | Règle | Exemple |
|---|---|---|
| Format | JSON, encodage UTF-8 | — |
| **Noms des champs** (P-A1) | Français, `snake_case`, **sans accents** | `niveau_confiance`, `version_modele` |
| **Valeurs d'énumération** (P-A1) | Même règle | `"en_attente_confirmation"`, `"etat_sur"` |
| **Identifiants** (P-A2) | UUID (texte de 36 caractères) | `"3f2b8c1e-…"` |
| **Dates et heures** (P-A3) | ISO 8601 en **UTC**, à la milliseconde, suffixe `Z` | `"2026-10-04T19:30:00.123Z"` |
| Durées | Nombre entier suffixé par l'unité dans le nom | `latence_ms: 182`, `delai_confirmation_s: 10` |
| Confiance, taux | Nombre décimal entre 0 et 1 | `niveau_confiance: 0.72` (l'interface affiche « 72 % ») |
| Valeur absente | `null` (le champ est toujours présent) | `"motif_rejet": null` |

Pourquoi pas d'accents dans les noms : ils deviennent des noms de variables en Python (Pydantic) et en TypeScript ; sans accents, aucun risque d'encodage ni de faute de frappe. Les **textes affichés** (messages, libellés) gardent leurs accents.

### 1.3 Authentification (D-74, ARCH-0 § 7)

- Après `POST /api/v1/auth/connexion`, le backend pose un **cookie de session** `HttpOnly` et `SameSite=Strict`. Le navigateur le renvoie tout seul à chaque requête et à l'ouverture du WebSocket ; le code JavaScript ne le voit jamais.
- Sans session valide → **401**. Session valide mais rôle insuffisant → **403**.
- Droits détaillés par rôle : `[À DÉFINIR — D-09]`. En attendant, chaque route de la partie 2 indiquera le rôle **proposé**.
- Seule exception : la page vitrine (D-76) n'appelle aucune route protégée.

### 1.4 Codes HTTP utilisés

| Code | Nom | Quand |
|---|---|---|
| 200 | OK | Lecture ou action réussie, avec une réponse |
| 201 | Created | Création réussie (session, profil…) ; la réponse contient l'objet créé |
| 202 | Accepted | Demande acceptée mais **pas encore terminée** (ex. « Reconnecter », DIAG-6 (d)) : le résultat arrivera par `/ws/flux` |
| 204 | No Content | Action réussie, rien à renvoyer (ex. déconnexion) |
| 400 | Bad Request | Requête mal formée (JSON illisible) |
| 401 | Unauthorized | Pas connecté |
| 403 | Forbidden | Connecté, mais rôle insuffisant (D-09) ou consentement absent (F-40) |
| 404 | Not Found | L'objet demandé n'existe pas |
| **409** | Conflict | **L'action est interdite dans l'état actuel** : transition refusée par DIAG-5 (ex. « activer » alors que l'état est Préparation), commande **déjà traitée** (D-87), conditions de reprise non réunies (D-91) |
| 422 | Unprocessable Entity | Données invalides (champ manquant, valeur hors liste) — produit automatiquement par FastAPI/Pydantic |
| 500 | Internal Server Error | Erreur imprévue du backend |
| 503 | Service Unavailable | Ressource indisponible : source non connectée, cible indisponible |

> **Une détection rejetée n'est pas une erreur HTTP.** Le rejet est une décision normale du Core (DIAG-6 (b)) : il arrive par `/ws/flux` comme objet `Decision` (section 3.5), pas comme un code 4xx.

### 1.5 Format unique des erreurs (P-A4)

Toute réponse 4xx ou 5xx a la même forme, pour que l'interface affiche toujours **la cause et l'action possible** (FW-37) :

```json
{
  "erreur": {
    "code": "transition_interdite",
    "message": "Impossible d'activer les commandes : CortexOS est en Préparation.",
    "action_possible": "Lancez une calibration ou sélectionnez un modèle existant.",
    "details": { "etat_actuel": "preparation", "action": "activer" }
  }
}
```

| Champ | Type | Rôle |
|---|---|---|
| `code` | texte (snake_case) | Stable, pour le code de l'interface (`if (code === "deja_traitee")`) |
| `message` | texte | Lisible par une personne, en français |
| `action_possible` | texte ou `null` | Ce que la personne peut faire (FW-37) |
| `details` | objet ou `null` | Informations utiles au débogage ou à l'affichage |

Codes d'erreur prévus (liste complétée en partie 2) : `non_connecte`, `role_insuffisant`, `consentement_absent`, `introuvable`, `donnees_invalides`, `transition_interdite`, `deja_traitee`, `conditions_reprise_non_reunies`, `source_indisponible`, `cible_indisponible`, `erreur_interne`.

### 1.6 Listes et pagination (P-A5)

Les listes longues (journal, sessions) se lisent par pages :

`GET /api/v1/journal?limite=50&decalage=100`

```json
{ "elements": [ … ], "total": 1342, "limite": 50, "decalage": 100 }
```

`limite` : 50 par défaut, 200 au maximum. Les filtres (session, type, gravité…) seront précisés en partie 2.

---

## 2. Énumérations

Valeurs exactes reprises de DIAG-3 et DIAG-5, écrites selon P-A1.

| Énumération | Valeurs | Source |
|---|---|---|
| `EtatGlobal` | `arrete`, `preparation`, `calibration`, `pret`, `actif`, `suspendu`, `etat_sur` | DIAG-5 ① |
| `TypeSource` | `casque`, `simulation`, `enregistrement` | F-03, F-32 |
| `EtatConnexionSource` | `deconnectee`, `connexion_en_cours`, `connectee`, `signal_perdu`, `fin_enregistrement` | DIAG-5 ⑤ |
| `NiveauQualite` | `suffisante`, `insuffisante` **(proposé, P-A6)** ; niveaux plus fins `[À DÉFINIR — Documentation technique]` | DIAG-5 ⑤, F-02 |
| `IssueDecision` | `acceptee`, `rejetee`, `en_attente_confirmation`, `confirmation` (« oui » qui confirme, D-85), `repos` (aucune commande attendue) | DIAG-3, DIAG-7 A2 |
| `MotifRejet` | `systeme_non_actif`, `qualite_insuffisante`, `confiance_insuffisante`, `cible_indisponible`, `commande_non_autorisee`, `delai_non_ecoule`, `confirmation_en_attente` | Spéc. 4.5, D-79, D-85 |
| `StatutCommande` | `acceptee`, `en_attente_confirmation`, `confirmee`, `annulee`, `expiree`, `envoyee`, `executee`, `echouee` | DIAG-5 ② |
| `OrigineCommande` | `eeg`, `manuelle` `[D-27]` | DIAG-3 |
| `IssueConfirmation` | `confirmee`, `annulee`, `expiree` | DIAG-3 |
| `TypeAuteur` | `compte`, `intention_eeg`, `systeme` | D-89 |
| `Disponibilite` | `disponible`, `indisponible` | DIAG-3 |
| `TypeCible` | `ordinateur`, `lampe_simulee` | D-72 |
| `TypeSession` | `utilisation`, `experimentation` | D-50, `[D-18]` |
| `StatutSession` | `creee`, `en_cours`, `en_pause`, `terminee`, `interrompue` | DIAG-5 ③ |
| `StatutCalibration` | `non_commencee`, `en_cours`, `entrainement`, `exploitable`, `insuffisante`, `interrompue` | DIAG-5 ④ |
| `PorteeConsentement` | `utilisation`, `experimentation`, `export` | DIAG-3, `[D-09, D-34]` |
| `Role` | `utilisateur`, `accompagnant`, `experimentateur`, `administrateur` | D-47 |
| `TypeEvenement` | `connexion`, `qualite_signal`, `calibration`, `detection`, `decision`, `commande`, `resultat`, `etat_global`, `session`, `erreur`, `modification_parametre` | DIAG-3, F-36 |
| `Gravite` | `information`, `avertissement`, `erreur`, `critique` | F-37 |
| `StatutAlerte` | `active`, `traitee` | DIAG-3 |

**Écarts avec DIAG-3, corrigés ici** (DIAG-3 est un modèle du domaine, le contrat suit les diagrammes d'états, plus précis) :

| Énumération | DIAG-3 | Contrat | Pourquoi |
|---|---|---|---|
| `MotifRejet` | 6 motifs | + `confirmation_en_attente` | D-85 (ajouté après DIAG-3) |
| `IssueDecision` | 3 issues | + `confirmation`, `repos` | DIAG-7 A2 : trois sorties sans rejet ; classement du repos dans les mesures `[D-84]` |
| `StatutCommande` | sans `acceptee`, `confirmee` | avec | DIAG-5 ② |
| `StatutCalibration` | `terminee` | `entrainement`, `exploitable`, `insuffisante` | DIAG-5 ④ (D-65) |
| `TypeEvenement` | « suspension / reprise / arrêt » | `etat_global` | Un seul type pour tous les changements d'état global, la transition est dans `details` |

---

## 3. Objets échangés

Notation : `champ : type` ; `?` = peut valoir `null` ; `[]` = liste. Chaque objet deviendra un **modèle Pydantic** dans `backend/app/` et un **type TypeScript** dans `frontend/`.

### 3.1 `EtatSysteme` — ce que CortexOS fait maintenant (FW-01, FW-02, FW-25)

| Champ | Type | Sens |
|---|---|---|
| `etat` | `EtatGlobal` | État global (DIAG-5 ①) |
| `commandes_actives` | booléen | `true` seulement si `etat = actif` (raccourci pour l'affichage) |
| `depuis` | date | Heure du dernier changement d'état |
| `motif` | texte ? | Raison de l'état sûr ou de la suspension (ex. « perte du signal ») |
| `source` | `Source` ? | Source actuelle (3.2) |
| `modele_actif` | `ModeleResume` ? | Modèle utilisé pour décider (3.11) |
| `session_en_cours_id` | UUID ? | Session active, si elle existe (D-77 : facultative) |
| `commande_en_attente_id` | UUID ? | Commande qui attend une confirmation (D-85 : une seule à la fois) |

### 3.2 `Source` et `Canal` — le signal (FW-02, FW-08 à FW-11)

**`Source`**

| Champ | Type | Sens |
|---|---|---|
| `type` | `TypeSource` | Casque, simulation ou enregistrement — **toujours affiché** (F-03, F-32) |
| `nom` | texte | Ex. « Carte synthétique BrainFlow », « ADS1299 8 canaux » |
| `etat_connexion` | `EtatConnexionSource` | DIAG-5 ⑤ |
| `nb_canaux` | entier | Ex. 8 |
| `frequence_hz` | entier | Fréquence d'échantillonnage, ex. 250 |
| `qualite` | `Qualite` ? | `null` si non connectée |

**`Qualite`**

| Champ | Type | Sens |
|---|---|---|
| `globale` | `NiveauQualite` | F-02 |
| `par_canal` | `Canal`[] | Vide si la source ne le permet pas `[D-04]` |
| `horodatage` | date | Moment de l'évaluation (FW-49 : fraîcheur) |

**`Canal`** : `nom` (texte, ex. « C3 ») · `qualite` (`NiveauQualite`).

Le **signal lui-même** (échantillons pour les courbes, FW-12) n'est pas un objet REST : il passe par `/ws/flux` (partie 3).

### 3.3 `Detection` — ce que l'IA a reconnu (FW-03, F-11)

| Champ | Type | Sens |
|---|---|---|
| `id` | UUID | — |
| `intention` | texte | Nom de l'intention `[D-03]` (ex. `"main_gauche"`, `"repos"`, `"oui"`) |
| `niveau_confiance` | décimal 0–1 | Confiance du modèle |
| `fin_fenetre` | date | **t0** : fin de la fenêtre de signal analysée |
| `horodatage` | date | **t1** : moment de la détection |
| `type_source` | `TypeSource` | Recopié depuis la source (F-32) |
| `version_modele` | texte | Modèle qui a détecté (F-27) |
| `session_id` | UUID ? | Session en cours, sinon `null` (D-77) |

### 3.4 `ParametresSurete` — les réglages qui protègent (FW-27, FW-28)

| Champ | Type | Sens |
|---|---|---|
| `seuil_confiance` | décimal 0–1 | Valeur `[À DÉFINIR — D-10]` ; qui peut la modifier `[D-23]` |
| `delai_minimal_ms` | entier ? | Entre deux commandes `[D-30]` ; `null` si non retenu |
| `delai_confirmation_s` | entier | Par défaut `[À DÉFINIR — D-31]`, réglable par profil |

### 3.5 `Decision` — ce que le Core a décidé (FW-03, F-15, F-21)

| Champ | Type | Sens |
|---|---|---|
| `id` | UUID | — |
| `detection_id` | UUID | Détection jugée |
| `issue` | `IssueDecision` | Acceptée, rejetée, en attente, confirmation, repos |
| `motif_rejet` | `MotifRejet` ? | Premier garde-fou qui a échoué (D-79), `null` si non rejetée |
| `niveau_confiance` | décimal | Recopié pour l'affichage (D-81) |
| `seuil` | décimal | Seuil appliqué **à ce moment-là** (D-81 ; le seuil peut changer ensuite) |
| `commande_id` | UUID ? | Commande créée, si l'issue est `acceptee` ou `en_attente_confirmation` |
| `horodatage` | date | — |

### 3.6 `Commande` — ce qui a été envoyé, et ce qu'il en est advenu (FW-04, FW-28)

| Champ | Type | Sens |
|---|---|---|
| `id` | UUID | Créé par le Core ; **transmis jusqu'à l'agent** et renvoyé avec le résultat (DIAG-6 (a)) |
| `decision_id` | UUID ? | `null` pour une commande manuelle `[D-27]` |
| `origine` | `OrigineCommande` | — |
| `cible_id` | UUID | Système cible visé |
| `type_commande` | texte | Nom dans la liste fermée de la cible (ex. `"clic"`, `"allumer"`) |
| `sensible` | booléen | Demande une confirmation (F-18) |
| `statut` | `StatutCommande` | DIAG-5 ② |
| `echeance_confirmation` | date ? | Fin du délai, si en attente ; **le Core fait foi** (D-86), l'interface calcule le compte à rebours |
| `confirmation` | `Confirmation` ? | 3.7 |
| `resultat` | `Resultat` ? | 3.8 |
| `horodatages` | `Horodatages` | t0 à t3 (ci-dessous) |
| `latence_ms` | entier ? | t3 − t0, **hors temps humain** (D-88) ; `null` tant que non exécutée |
| `session_id` | UUID ? | — |

**`Horodatages`** : `t0_fin_fenetre` · `t1_detection` · `t2_envoi` ? · `t3_resultat` ? (tous en date). Tous viennent de **la même horloge** (le PC, DIAG-8a) : c'est ce qui rend la latence fiable.

### 3.7 `Confirmation` (D-24, D-87 à D-89)

| Champ | Type | Sens |
|---|---|---|
| `issue` | `IssueConfirmation` | — |
| `auteur` | `Auteur` | Qui a confirmé ou annulé ; `systeme` pour une expiration ou une annulation automatique (état sûr) |
| `horodatage` | date | — |
| `temps_decision_ms` | entier | Temps humain, mesuré à part (D-88) |

**`Auteur`** : `type` (`TypeAuteur`) · `compte_id` (UUID ?) · `nom_affiche` (texte ?). Pour une confirmation par EEG : `type = intention_eeg`, `compte_id = null` (D-89).

### 3.8 `Resultat` — le retour du système cible (F-22)

| Champ | Type | Sens |
|---|---|---|
| `succes` | booléen | — |
| `message` | texte ? | Message de la cible, ou cause de l'échec (ex. « délai dépassé » `[D-68]`) |
| `horodatage` | date | **t3** |
| `recu_en_etat_sur` | booléen | `true` si le résultat arrive après un passage en état sûr (D-93) |

### 3.9 `Cible` et `TypeCommande` (FW-05, FW-22, F-23, F-24)

**`Cible`** : `id` · `nom` (« Ordinateur », « Lampe simulée ») · `type` (`TypeCible`) · `disponibilite` (`Disponibilite`) · `commandes` (`TypeCommande`[]) · `etat_simule` (objet ?, ex. `{"allumee": true}` pour la lampe, affiché dans l'interface, P4).

**`TypeCommande`** : `nom` (ex. `"curseur_gauche"`) · `libelle` (« Déplacer le curseur à gauche ») · `sensible` (booléen ; liste `[À DÉFINIR — D-24]`).

### 3.10 `Correspondance` et `Regle` (FW-20, F-14)

**`Correspondance`** : `version` (entier) · `active` (booléen) · `regles` (`Regle`[]).
**`Regle`** : `intention` (texte) · `cible_id` (UUID) · `type_commande` (texte).
Modification par l'interface : `[D-20]`.

### 3.11 `Calibration` et `Modele` (FW-16 à FW-18)

**`Calibration`**

| Champ | Type | Sens |
|---|---|---|
| `id` | UUID | — |
| `profil_id` | UUID | — |
| `statut` | `StatutCalibration` | DIAG-5 ④ |
| `essai_en_cours` | `Essai` ? | Consigne et intention à produire (FW-16) |
| `progression` | objet | `{ "essais_faits": 12, "essais_total": 40 }` |
| `precision_estimee` | décimal ? | Après entraînement |
| `modele_version` | texte ? | Modèle produit, si exploitable (F-09) |
| `debut`, `fin` | date, date ? | — |

**`ModeleResume`** : `version` · `date` · `calibration_id` · `actif` (booléen).

### 3.12 `Session`, `Essai`, `Mesures` (FW-07, FW-29 à FW-32)

**`Session`**

| Champ | Type | Sens |
|---|---|---|
| `id` | UUID | — |
| `type` | `TypeSession` | `[D-18]` |
| `statut` | `StatutSession` | DIAG-5 ③ |
| `profil_id` | UUID | — |
| `scenario` | texte ? | `[D-05]` |
| `conditions`, `notes` | texte ? | Saisies par l'expérimentateur |
| `type_source` | `TypeSource` | Enregistré automatiquement (F-27) |
| `version_modele` | texte | Enregistrée automatiquement (F-27) |
| `debut`, `fin` | date ?, date ? | — |
| `compteurs` | objet | En direct (FW-07) : `{ "detections": 120, "rejets": 31, "commandes": 42 }` |

**`Essai`** : `id` · `rang` · `intention_attendue` · `debut` · `fin` ? · `session_id` ? · `calibration_id` ? (l'un ou l'autre, D-51).

**`Mesures`** (FW-31 ; calculées à la fin, pas de précision en direct, spéc. 4.7)

| Champ | Type | Sens |
|---|---|---|
| `session_id` | UUID | — |
| `type_source` | `TypeSource` | **Jamais mélanger réel et simulé** (F-32) |
| `precision` | décimal ? | Expérimentation seulement |
| `niveau_hasard` | décimal ? | Ex. 0,33 pour 3 intentions |
| `matrice_confusion` | objet ? | `{ "intentions": [...], "valeurs": [[...]] }` |
| `latence_ms` | objet ? | `{ "moyenne", "mediane", "max" }` |
| `taux_rejet` | objet | `{ "total": 0.21, "par_motif": { "confiance_insuffisante": 0.15, … } }` |
| `taux_commandes_involontaires` | décimal ? | Expérimentation seulement |
| `periodes_exclues_s` | entier | Pauses et pertes de signal exclues (D-95) |
| `calculees_le` | date | — |

### 3.13 `Profil`, `Consentement`, `Compte` (FW-39 à FW-42)

**`Profil`** : `id` · `pseudonyme` · `cree_le` · `consentement` (`Consentement` ?) · `modele_actif` (`ModeleResume` ?) · `preferences` (objet : délai de confirmation du profil, D-31 ; affichage).
**`Consentement`** : `portees` (`PorteeConsentement`[]) · `recueilli_le` · `retire_le` ?.
**`Compte`** : `id` · `identifiant` · `nom_affiche` · `roles` (`Role`[]) · `profil_id` ? (D-53). **Le mot de passe n'apparaît jamais dans une réponse**, même haché.

### 3.14 `EvenementJournal` et `Alerte` (FW-06, FW-36, FW-50)

**`EvenementJournal`**

| Champ | Type | Sens |
|---|---|---|
| `id` | UUID | — |
| `horodatage` | date | — |
| `type` | `TypeEvenement` | — |
| `gravite` | `Gravite` | — |
| `message` | texte | Lisible (ex. « Commande « clic » exécutée en 182 ms ») |
| `session_id` | UUID ? | Pour filtrer (FW-36) |
| `auteur` | `Auteur` ? | Pour les actions humaines et les modifications de paramètres (F-36, FW-50) |
| `references` | objet | Liens vers les objets concernés : `{ "commande_id": …, "decision_id": … }` |
| `details` | objet ? | Ex. `{ "ancienne_valeur": 0.7, "nouvelle_valeur": 0.75 }`, `{ "de": "actif", "vers": "etat_sur" }` |

**`Alerte`** : `id` · `evenement_id` · `gravite` · `message` · `action_proposee` ? · `statut` (`StatutAlerte`).

---

## 4. Propositions à valider (partie 1)

| N° | Proposition | Alternative écartée | Pourquoi |
|---|---|---|---|
| **P-A1** | Noms de champs et valeurs en **français, snake_case, sans accents** | Anglais (`confidence`), ou français avec accents | Cohérent avec `docs/` et le code (`etats.py`, `commande.py`) ; sans accents, pas de problème en Python ni en TypeScript |
| **P-A2** | Identifiants **UUID** | Entiers auto-incrémentés par PostgreSQL | Le Core (Python pur, sans base) crée l'`id` d'une commande **avant** qu'elle soit enregistrée, et le transmet à l'agent (DIAG-6 (a)) |
| **P-A3** | Dates ISO 8601 **UTC** avec millisecondes ; l'interface convertit en heure locale | Heure locale dans l'API | Une seule référence ; les millisecondes sont nécessaires pour la latence |
| **P-A4** | **Format unique des erreurs** `{ erreur: { code, message, action_possible, details } }`, y compris pour les 422 de FastAPI | Format par défaut de FastAPI (`{ detail }`, qui change selon l'erreur) | FW-37 : toujours la cause et l'action possible ; une seule fonction d'affichage dans l'interface |
| **P-A5** | Pagination `limite` / `decalage` | Pagination par curseur | Suffisant pour un seul PC et quelques milliers d'événements ; plus simple à comprendre |
| **P-A6** | `NiveauQualite` = `suffisante` / `insuffisante` au MVP | 3 ou 5 niveaux, ou un score 0–100 | DIAG-5 ⑤ ne distingue que ces deux états ; un score pourra s'ajouter sans casser le contrat |

## 5. Points ouverts de la partie 1

- **D-03** : noms des intentions (`"main_gauche"`… sont des exemples).
- **D-09** : droits par rôle (partie 2 : rôle proposé par route).
- **D-10, D-23, D-30, D-31** : valeurs des paramètres de sûreté.
- **D-24** : liste des commandes sensibles · **D-27** : commandes manuelles · **D-68** : délai du résultat.
- **D-84** : la décision `repos` compte-t-elle comme un rejet dans les mesures ?
- Calcul de la qualité et niveaux fins : Documentation technique.
