# Contrat d'API v0 — CortexOS IA

| | |
|---|---|
| **Réf.** | API-0 (Planning MVP, S3, mode B — D-46) |
| **Sources** | Spécification fonctionnelle (§ 4, § 5 : F-01 à F-44, FW-01 à FW-51) · DIAG-3 (classes, énumérations) · DIAG-5 (états) · DIAG-6 (séquences) · ARCH-0 (§ 5, 7, 8) · Décisions D-24 à D-103 |
| **Version** | 0.3 — 5 octobre 2026 · **parties 1 à 3** (document complet) |
| **Statut** | Parties 1 et 2 validées (D-104, D-106) · partie 3 à relire par Eloge |

## Rôle de ce document

Le contrat d'API dit **exactement** ce que l'interface Web (Next.js) peut demander au backend (FastAPI) et ce qu'elle reçoit. Il est écrit **avant le code** : chaque tranche vérifie que ses routes le respectent (Planning § 3.2).

| Partie | Contenu | Statut |
|---|---|---|
| 1 | Conventions communes · format des erreurs · objets échangés (sections 1 à 3) | Validée (D-104) |
| 2 | Routes REST `/api/v1/…`, domaine par domaine, avec traçabilité FW-xx → route (section 4) | Validée (D-106) |
| **3** | WebSocket `/ws/flux` · canaux locaux `/ws/agent` et `/api/local/arret-urgence` (sections 5 et 6) | **À relire** |

Les **propositions** (P-A1…) sont rassemblées en section 7 : P-A1 à P-A10 validées (D-104, D-106), P-A11 à P-A16 à valider. Tant qu'elles ne sont pas validées, elles restent « proposé ».

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
| `NiveauQualite` | `suffisante`, `insuffisante` (D-104) ; niveaux plus fins `[À DÉFINIR — Documentation technique]` | DIAG-5 ⑤, F-02 |
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

## 4. Routes REST (partie 2)

### 4.0 Lire les tableaux

- Toutes les adresses commencent par `/api/v1` (sauf `/api/health`).
- **Rôle** : rôle minimal **proposé** (P-A10), en attendant D-09. `Connecté` = toute personne connectée · `U` utilisateur · `Acc` accompagnant · `Exp` expérimentateur · `Adm` administrateur.
- **Réponse** : objet de la section 3 renvoyé en cas de succès. `Page<X>` = liste paginée (1.6).
- **Erreurs** : codes propres à la route ; 401, 422 et 500 sont possibles partout et ne sont pas répétés.
- Une **action** qui change un état (DIAG-5) est un `POST` sur `…/{verbe}` (P-A7) : `POST /sessions/{id}/demarrer`. Si la machine à états l'interdit → **409 `transition_interdite`**, avec `details.etat_actuel`.

### 4.1 Santé et authentification (FW-42, D-74)

| Méthode | Route | Rôle | Entrée | Réponse | Erreurs |
|---|---|---|---|---|---|
| GET | `/api/health` | Public | — | `{ "statut": "ok", "version": "0.1.0" }` | — |
| POST | `/auth/connexion` | Public | `{ identifiant, mot_de_passe }` | 200 `Compte` + cookie | 401 `identifiants_invalides` (message volontairement vague : ne dit pas lequel est faux) |
| POST | `/auth/deconnexion` | Connecté | — | 204 | — |
| GET | `/auth/moi` | Connecté | — | `Compte` | — |
| POST | `/auth/mot-de-passe` | Connecté | `{ ancien, nouveau }` | 204 | 401 `identifiants_invalides` |
| GET | `/comptes` | Adm | — | `Compte[]` | 403 |
| POST | `/comptes` | Adm | `{ identifiant, nom_affiche, mot_de_passe, roles }` | 201 `Compte` | 403 · 409 `identifiant_deja_utilise` |
| PATCH | `/comptes/{id}` | Adm | `{ nom_affiche?, roles?, actif? }` | `Compte` | 403 · 404 |

`/api/health` reste hors de `/v1` : c'est une route technique (Planning S4), pas une partie du contrat métier (P-A9). Le premier administrateur est créé en ligne de commande, pas par l'API (P7, D-101).

### 4.2 Système et sûreté (FW-01, FW-25 à FW-27, F-19 ; DIAG-5 ①)

| Méthode | Route | Transition | Rôle | Entrée | Réponse | Erreurs |
|---|---|---|---|---|---|---|
| GET | `/systeme/etat` | — | Connecté | — | `EtatSysteme` | — |
| POST | `/systeme/demarrer` | T1 / T2 | Acc | `{ profil_id }` | `EtatSysteme` (Préparation ou Prêt selon le modèle) | 409 `transition_interdite` · 503 `source_indisponible` · 403 `consentement_absent` |
| POST | `/systeme/selectionner-modele` | T4 | Acc | `{ version }` | `EtatSysteme` | 404 · 409 |
| POST | `/systeme/activer` | T7 | U, Acc | — | `EtatSysteme` | 409 `transition_interdite` · 409 `conditions_non_reunies` `[D-67]` |
| POST | `/systeme/suspendre` | T8 | **Connecté** (P-A10) | — | `EtatSysteme` | 409 si déjà non actif |
| POST | `/systeme/reprendre` | T9 | U, Acc | — | `EtatSysteme` | 409 `conditions_reprise_non_reunies` (D-91 ; `details.conditions` liste celles qui manquent) |
| POST | `/systeme/arreter` | T17 | Acc | — | `EtatSysteme` | 409 |
| GET | `/parametres-surete` | — | Connecté | — | `ParametresSurete` | — |
| PATCH | `/parametres-surete` | — | `[D-23]` | `{ seuil_confiance?, delai_minimal_ms?, delai_confirmation_s? }` | `ParametresSurete` (journalisé avec l'auteur, FW-50) | 403 · 422 |

- **L'arrêt d'urgence n'est pas ici** : il passe par `/api/local/arret-urgence` (partie 3), justement pour fonctionner sans l'interface (F-20, D-75). « Suspendre » (FW-26) ne le remplace pas.
- Le passage en **état sûr** (T12 à T15) et la sortie vers **Suspendu** (T16) ne sont pas des routes : ce sont des réactions automatiques, annoncées sur `/ws/flux`.

### 4.3 Source et signal (FW-02, FW-08 à FW-11, FW-15 ; F-01 à F-05 ; DIAG-5 ⑤)

| Méthode | Route | Transition | Rôle | Entrée | Réponse | Erreurs |
|---|---|---|---|---|---|---|
| GET | `/source` | — | Connecté | — | `Source` ou `null` | — |
| GET | `/source/enregistrements` | — | Acc, Exp | — | `[{ id, nom, origine, duree_s, frequence_hz }]` (fichiers de `data/`) | — |
| POST | `/source/connecter` | E1 | Acc | `{ type, enregistrement_id? }` | **202** `Source` (`connexion_en_cours`) | 409 déjà connectée · 404 enregistrement |
| POST | `/source/reconnecter` | E5 | Acc | — | **202** `Source` | 409 si le signal n'est pas perdu |
| POST | `/source/deconnecter` | E6 / E9 | Acc | — | `Source` | 409 |

- **202** (P-A8) : la connexion prend du temps (BrainFlow, Bluetooth). La route répond tout de suite ; la réussite (E2) ou l'échec (E3) arrive sur `/ws/flux`. L'interface n'est jamais bloquée.
- La **qualité** (FW-10, FW-11) et les **échantillons** du signal (FW-12) changent en continu : ils passent **uniquement** par `/ws/flux` (partie 3). `GET /source` en donne l'état au moment de la lecture (utile au chargement d'une page).

### 4.4 Calibration et modèles (FW-16 à FW-18 ; F-06 à F-09 ; DIAG-5 ④)

| Méthode | Route | Transition | Rôle | Entrée | Réponse | Erreurs |
|---|---|---|---|---|---|---|
| POST | `/calibrations` | K2 (+ T3, T10, T11) | Acc | `{ profil_id }` | 201 `Calibration` | 409 `qualite_insuffisante` (F-07, K1) · 403 `consentement_absent` · 409 `transition_interdite` |
| GET | `/calibrations/{id}` | — | Connecté | — | `Calibration` | 404 |
| POST | `/calibrations/{id}/interrompre` | K5 | U, Acc | — | `Calibration` | 409 |
| GET | `/profils/{id}/modeles` | — | Connecté | — | `ModeleResume[]` | 404 |

- **Recommencer** (F-08) = créer une nouvelle calibration (`POST /calibrations`). L'ancienne reste dans l'historique, avec son statut.
- Les **essais** s'enchaînent tout seuls côté backend : la consigne de l'essai suivant et la progression arrivent sur `/ws/flux`. Aucune route « essai suivant ».

### 4.5 Commandes et confirmation (FW-03, FW-04, FW-28 ; F-14, F-18, F-22 ; DIAG-5 ②)

| Méthode | Route | Transition | Rôle | Entrée | Réponse | Erreurs |
|---|---|---|---|---|---|---|
| GET | `/commandes` | — | Connecté | filtres : `session_id`, `statut`, `limite`, `decalage` | `Page<Commande>` | — |
| GET | `/commandes/{id}` | — | Connecté | — | `Commande` | 404 |
| POST | `/commandes/{id}/confirmer` | C6 | Acc (U en développement, D-24) | — | `Commande` | **409 `deja_traitee`** (D-87 ; `details.statut`) · 503 `cible_indisponible` `[D-68]` |
| POST | `/commandes/{id}/annuler` | C7 | Acc, U | — | `Commande` | **409 `deja_traitee`** |
| POST | `/commandes/manuelles` | C0 | `[D-27]` | `{ cible_id, type_commande }` | 201 `Commande` (`origine: manuelle`) | `[À DÉFINIR — D-27]` |

- La confirmation **par intention EEG « oui »** ne passe pas par REST : c'est une détection, qui suit la chaîne normale (DIAG-6 (c)).
- **Contrôle « encore en attente ? »** (D-87) : le Core vérifie le statut **et** le change dans la même opération, protégée par un verrou. Si deux demandes arrivent ensemble, une seule trouve `en_attente_confirmation` ; l'autre reçoit 409.
- Les **détections et décisions** n'ont pas de route de lecture : en direct elles arrivent sur `/ws/flux`, après coup elles sont dans le journal (4.9) et dans les mesures (4.7).

### 4.6 Systèmes cibles et correspondance (FW-05, FW-20 à FW-24 ; F-14, F-23, F-24)

| Méthode | Route | Rôle | Entrée | Réponse | Erreurs |
|---|---|---|---|---|---|
| GET | `/cibles` | Connecté | — | `Cible[]` | — |
| GET | `/cibles/{id}` | Connecté | — | `Cible` | 404 |
| GET | `/profils/{id}/correspondance` | Connecté | — | `Correspondance` active | 404 |
| PUT | `/profils/{id}/correspondance` | `[D-20]` | `{ regles }` | `Correspondance` (nouvelle version, journalisée) | `[À DÉFINIR — D-20]` |

- La **disponibilité** d'une cible change toute seule (l'agent se connecte ou se déconnecte, P5) : elle est annoncée sur `/ws/flux`.
- **FW-24** (types de cibles selon la trajectoire MVP / Ext. / Futur) est un contenu fixe de l'interface : pas de route.

### 4.7 Sessions et mesures (FW-07, FW-29 à FW-34 ; F-27 à F-35 ; DIAG-5 ③)

| Méthode | Route | Transition | Rôle | Entrée | Réponse | Erreurs |
|---|---|---|---|---|---|---|
| POST | `/sessions` | S1 | Exp, Acc | `{ type, profil_id, scenario?, conditions?, notes? }` | 201 `Session` | 403 `consentement_absent` (portée) · 409 `modele_absent` (D-99 : calibrer d'abord) |
| GET | `/sessions` | — | Connecté | filtres : `type`, `profil_id`, `type_source`, `statut`, `limite`, `decalage` | `Page<Session>` (FW-32) | — |
| GET | `/sessions/{id}` | — | Connecté | — | `Session` | 404 |
| PATCH | `/sessions/{id}` | — | Exp, Acc | `{ conditions?, notes? }` | `Session` | 404 |
| POST | `/sessions/{id}/demarrer` | S2 | Exp, Acc | — | `Session` | 409 · 503 `source_indisponible` |
| POST | `/sessions/{id}/pause` | S3 | Connecté | — | `Session` | 409 |
| POST | `/sessions/{id}/reprendre` | S4 | Exp, Acc | — | `Session` | 409 |
| POST | `/sessions/{id}/terminer` | S5 / S6 | Exp, Acc | — | `Session` | 409 |
| GET | `/sessions/{id}/essais` | — | Connecté | — | `Essai[]` | 404 |
| GET | `/sessions/{id}/mesures` | — | Connecté | — | `Mesures` | 409 `session_en_cours` (mesures calculées à la fin ; en direct : `Session.compteurs`) |
| GET | `/sessions/{id}/export` | — | Exp | `format` `[À DÉFINIR — Doc. technique]` | Fichier (téléchargement) | 403 `consentement_absent` (portée `export`, F-35) |

- **Mettre en pause** suspend aussi les commandes (F-28) : la route fait les deux, dans cet ordre.
- **Protocole** de l'expérimentation (nombre d'essais, ordre des intentions) : `[À DÉFINIR — Q2 de DIAG-7]` ; il s'ajoutera à l'entrée de `POST /sessions`.
- **Reprendre une session interrompue** : `[À DÉFINIR — D-71]`, pas de route pour l'instant.

### 4.8 Profils, consentement et données (FW-39 à FW-41 ; F-39 à F-41)

| Méthode | Route | Rôle | Entrée | Réponse | Erreurs |
|---|---|---|---|---|---|
| GET | `/profils` | Acc, Exp | — | `Profil[]` | — |
| POST | `/profils` | Acc, Exp | `{ pseudonyme }` | 201 `Profil` | 409 `pseudonyme_deja_utilise` |
| GET | `/profils/{id}` | Connecté `[D-09]` | — | `Profil` | 404 |
| PATCH | `/profils/{id}` | U (le sien), Acc | `{ pseudonyme?, preferences? }` | `Profil` | 404 |
| PUT | `/profils/{id}/consentement` | U (le sien), Acc | `{ portees }` | `Consentement` (journalisé) | 404 · 422 |
| DELETE | `/profils/{id}/consentement` | U (le sien), Acc | — | `Consentement` (`retire_le` rempli) | 404 |
| DELETE | `/profils/{id}/donnees` | U (le sien), Adm | — | 204 | `[À DÉFINIR — D-38]` (ce qui est supprimé) |

- **Retirer le consentement** (F-41) arrête tout nouvel enregistrement **tout de suite** : une session en cours passe **Interrompue** (DIAG-7 A1). Ce que deviennent les données déjà enregistrées : `[À DÉFINIR — D-38]`.
- « Le sien » suppose qu'un compte soit lié au profil (D-53) ; les droits exacts restent `[D-09]`.

### 4.9 Journal et alertes (FW-06, FW-36, FW-37, FW-50 ; F-36, F-37)

| Méthode | Route | Rôle | Entrée | Réponse | Erreurs |
|---|---|---|---|---|---|
| GET | `/journal` | Connecté | filtres : `session_id`, `type`, `gravite`, `depuis`, `jusqua`, `limite`, `decalage` | `Page<EvenementJournal>` | — |
| GET | `/alertes` | Connecté | filtre : `statut` | `Alerte[]` | — |
| POST | `/alertes/{id}/traiter` | Connecté | — | `Alerte` (auteur journalisé) | 404 · 409 `deja_traitee` |

**Historique des paramètres** (FW-50) : `GET /journal?type=modification_parametre`, pas de route à part.

### 4.10 Traçabilité : chaque fonctionnalité Web du MVP a ses routes

| FW | Fonctionnalité | REST | `/ws/flux` (partie 3) |
|---|---|---|---|
| FW-01 | État de la chaîne | `GET /systeme/etat`, `GET /source`, `GET /cibles` | oui |
| FW-02 | Source des données | `GET /source` | oui |
| FW-03 | Détection, confiance, décision | — | oui |
| FW-04 | Commande et résultat | `GET /commandes` | oui |
| FW-05 | État de la cible | `GET /cibles` | oui |
| FW-06 | Événements récents | `GET /journal?limite=20` | oui |
| FW-07 | Indicateurs de session | `GET /sessions/{id}` | oui |
| FW-08 à FW-11 | Connexion, caractéristiques, qualité | 4.3 | oui |
| FW-12 | Signal en temps réel | — | oui |
| FW-15 | Alertes sur le signal | `GET /alertes` | oui |
| FW-16 à FW-18 | Calibration, modèle actif | 4.4 | oui |
| FW-20 | Correspondance | `GET /profils/{id}/correspondance` | — |
| FW-22 | Liste des cibles | `GET /cibles` | oui |
| FW-25, FW-26 | État global, suspendre / reprendre | 4.2 | oui |
| FW-27 | Seuil | `GET /parametres-surete` | — |
| FW-28 | Confirmation | `POST /commandes/{id}/confirmer`, `…/annuler` | oui (demande, compte à rebours) |
| FW-29 à FW-32 | Sessions, mesures | 4.7 | oui |
| FW-34 | Export | `GET /sessions/{id}/export` | — |
| FW-36, FW-37 | Journal, messages d'erreur | `GET /journal` ; format d'erreur 1.5 | oui |
| FW-39 à FW-41 | Profil, consentement | 4.8 | — |
| FW-42 | Comptes | 4.1 | — |
| FW-44, FW-49 | Accessibilité, fraîcheur | — (interface) | horodatage des messages |
| FW-50 | Historique des paramètres | `GET /journal?type=modification_parametre` | — |
| FW-51 | Page vitrine | aucune (publique) | — |

Hors MVP ou non décidés, sans route : FW-13, FW-14, FW-19, FW-21 `[D-20]`, FW-23 `[D-27]`, FW-24 (contenu fixe), FW-33, FW-35, FW-38, FW-43, FW-45 à FW-48.

### 4.11 Codes d'erreur (liste complète)

`non_connecte` · `identifiants_invalides` · `role_insuffisant` · `consentement_absent` · `introuvable` · `donnees_invalides` · `transition_interdite` · `conditions_non_reunies` · `conditions_reprise_non_reunies` · `deja_traitee` · `qualite_insuffisante` · `modele_absent` · `session_en_cours` · `source_indisponible` · `cible_indisponible` · `identifiant_deja_utilise` · `pseudonyme_deja_utilise` · `jeton_invalide` (canaux locaux, section 6) · `erreur_interne`.

---

## 5. Temps réel : WebSocket `/ws/flux` (partie 3)

### 5.1 Principe

**Photo de départ, puis changements.** Au chargement d'une page, l'interface lit l'état actuel par REST (`GET /systeme/etat`, `GET /source`…). Ensuite, elle ne redemande plus rien : le backend **pousse** chaque changement sur `/ws/flux`. Pas de *polling*, donc pas de requêtes répétées « y a-t-il du nouveau ? ».

| Étape | Ce qui se passe |
|---|---|
| Ouverture | Le navigateur ouvre `ws://localhost:8000/ws/flux` ; le cookie de session est vérifié (D-74). Sans session valide → fermeture avec le code **4401** |
| En service | Le backend envoie les messages de 5.3 ; le navigateur peut seulement s'abonner ou se désabonner (5.4, P-A12) |
| Coupure | Le navigateur se reconnecte seul (attente 1 s, 2 s, puis 5 s entre les essais), puis **relit la photo de départ** en REST. Les messages manqués ne sont pas renvoyés : l'historique est dans le journal |

**Les actions ne passent jamais par `/ws/flux`** (P-A12) : activer, confirmer, créer une session… restent des routes REST (section 4). Une seule voie pour les actions = un seul endroit pour vérifier les droits et renvoyer les erreurs (1.5).

### 5.2 Enveloppe commune (P-A11)

Chaque message a la même forme :

```json
{
  "type": "decision",
  "horodatage": "2026-10-04T19:30:00.412Z",
  "sequence": 18342,
  "donnees": { "…": "objet de la section 3" }
}
```

| Champ | Rôle |
|---|---|
| `type` | Dit à l'interface quoi faire du message (tableau 5.3) |
| `horodatage` | Heure d'envoi : sert à l'indicateur de fraîcheur (FW-49) |
| `sequence` | Compteur qui augmente de 1 à chaque message : un trou (18342 puis 18345) révèle des messages perdus → relire la photo de départ |
| `donnees` | Un objet de la section 3, sans changement de format |

### 5.3 Messages du backend vers le navigateur

| `type` | `donnees` | Envoyé quand | FW |
|---|---|---|---|
| `etat_systeme` | `EtatSysteme` | Chaque changement d'état global (DIAG-5 ①), y compris le passage automatique en état sûr | FW-01, FW-25 |
| `source` | `Source` | Changement de connexion (DIAG-5 ⑤ : connexion réussie, échec, signal perdu…) | FW-02, FW-08, FW-09 |
| `qualite` | `Qualite` | Changement de niveau, et au moins une fois par seconde tant que la source est connectée | FW-10, FW-11 |
| `signal` | `PaquetSignal` (ci-dessous) | En continu, **seulement aux abonnés** (5.4, P-A13) | FW-12 |
| `detection` | `Detection` | Chaque détection (en utilisation, sans le repos : D-78) | FW-03 |
| `decision` | `Decision` | Chaque décision du Core | FW-03 |
| `commande` | `Commande` | **Chaque changement de statut** : acceptée, en attente (avec `echeance_confirmation`), confirmée, envoyée, exécutée… | FW-04, FW-28 |
| `cible` | `Cible` | Changement de disponibilité (agent connecté ou non) ou de l'état simulé de la lampe | FW-05, FW-22 |
| `calibration` | `Calibration` | Nouvel essai (consigne), progression, changement de statut | FW-16, FW-17 |
| `session` | `Session` | Changement de statut, et `compteurs` toutes les 2 secondes pendant la session | FW-07, FW-30 |
| `essai` | `Essai` | Début et fin de chaque essai d'expérimentation (intention attendue) | FW-30 |
| `evenement` | `EvenementJournal` | Chaque événement du journal de gravité ≥ `information` | FW-06, FW-36 |
| `alerte` | `Alerte` | Nouvelle alerte, ou alerte traitée | FW-06, FW-15 |
| `battement` | `{}` | Toutes les **5 secondes**, même quand rien ne change (P-A14) | FW-49 |

**`PaquetSignal`**

| Champ | Type | Sens |
|---|---|---|
| `debut` | date | Heure du premier échantillon du paquet |
| `frequence_hz` | entier | Pour placer chaque point sur l'axe du temps |
| `canaux` | texte[] | Canaux présents (ceux demandés à l'abonnement) |
| `echantillons` | décimal[][] | Un tableau par canal, en microvolts ; environ 0,1 s de signal par paquet (P-A13) |

**Fraîcheur (FW-49, P-A14)** : si l'interface ne reçoit **aucun message pendant 10 secondes** (pas même un `battement`), elle affiche « données périmées — dernière mise à jour il y a X s » au lieu de laisser croire que l'écran est à jour.

### 5.4 Messages du navigateur vers le backend

Les deux seuls messages possibles (P-A12) :

```json
{ "type": "abonner", "flux": "signal", "canaux": ["C3", "Cz", "C4"] }
{ "type": "desabonner", "flux": "signal" }
```

Le signal est le seul flux à abonnement, parce que c'est le seul lourd (8 canaux × 250 échantillons par seconde). La page « Signal » s'abonne en s'ouvrant et se désabonne en se fermant ; les autres pages ne le reçoivent pas.

### 5.5 Codes de fermeture

| Code | Sens |
|---|---|
| 1000 | Fermeture normale (page fermée) |
| 1001 | Le backend s'arrête |
| 4401 | Pas de session valide : se reconnecter (`/connexion`) |
| 4400 | Message du navigateur incompréhensible |

Les codes 4000 à 4999 sont réservés aux applications : on y reprend les codes HTTP connus (4401 ≈ 401).

---

## 6. Canaux locaux (partie 3)

Ces canaux servent aux **programmes du PC**, jamais au navigateur. Ils n'acceptent que les connexions venant de `127.0.0.1`, avec un **jeton local** généré au démarrage du backend dans `data/` (P8, D-101 ; ARCH-0 § 7).

### 6.1 Agent ordinateur : WebSocket `/ws/agent` (D-55, P5)

**Connexion** : l'agent se connecte à `ws://127.0.0.1:8000/ws/agent` et envoie aussitôt sa présentation (P-A15) :

```json
{ "type": "presentation", "jeton": "…contenu de data/jeton-agent.txt…", "version_agent": "0.1.0",
  "commandes": ["curseur_gauche", "curseur_droite", "clic"] }
```

| Réponse du backend | Effet |
|---|---|
| `{ "type": "bienvenue" }` | La cible **Ordinateur** devient **disponible** (message `cible` sur `/ws/flux`) |
| Fermeture **4401** | Jeton faux, ou connexion venant d'une autre adresse que `127.0.0.1` |

**Échanges**

| Sens | `type` | Contenu | Rôle |
|---|---|---|---|
| Backend → agent | `executer` | `{ commande_id, type_commande, parametres }` | Ordre d'exécution ; `commande_id` est l'identifiant de la commande (3.6) |
| Agent → backend | `resultat` | `{ commande_id, succes, message }` | Résultat ; le backend en fait un `Resultat` (3.8), heure t3 |
| Les deux | *ping / pong* | (automatique, bibliothèque `websockets`) | Si l'agent ne répond plus, la cible passe **indisponible** et les commandes vers elle sont rejetées (F-23) |

- **Liste fermée côté agent** : une commande inconnue n'est **pas exécutée** ; l'agent répond `resultat` avec `succes: false`, `message: "commande inconnue"`. C'est le deuxième contrôle de la liste fermée (DIAG-6 (a)).
- Pas de réponse dans le délai → commande **Échouée** `[À DÉFINIR — D-68]`.
- Un résultat qui arrive après un passage en état sûr est enregistré avec `recu_en_etat_sur: true` (D-93).

### 6.2 Arrêt d'urgence : `POST /api/local/arret-urgence` (D-57, D-75)

| | |
|---|---|
| Appelant | Le programme `arret_urgence/` (C18), quand le raccourci clavier global est pressé |
| En-tête | `X-Jeton-Local: <contenu de data/jeton-arret.txt>` (P-A15) |
| Entrée | `{ "declencheur": "raccourci_clavier" }` |
| Réponse | **200** `EtatSysteme` (`etat: "etat_sur"`) |
| Erreur | **403** `jeton_invalide` (jeton faux, ou requête venant d'une autre adresse que `127.0.0.1`) |

**Effets, dans cet ordre** : passage en **État sûr** (T15) · commande en attente annulée (auteur `systeme`) · session en cours **Interrompue** (S7) · alerte **critique** · événement du journal · message `etat_systeme` sur `/ws/flux`.

**Toujours 200, même si CortexOS est déjà en état sûr ou arrêté** (P-A16) : un arrêt d'urgence ne doit jamais « échouer » parce que le système est déjà arrêté. Le programme C18 n'a qu'une chose à vérifier : la réponse est 200.

Qui peut déclencher l'arrêt : `[À DÉFINIR — D-19]`.

---

## 7. Propositions

### 7.1 Partie 1 — validées le 04/10/2026 (D-104)

| N° | Proposition | Alternative écartée | Pourquoi |
|---|---|---|---|
| **P-A1** | Noms de champs et valeurs en **français, snake_case, sans accents** | Anglais (`confidence`), ou français avec accents | Cohérent avec `docs/` et le code (`etats.py`, `commande.py`) ; sans accents, pas de problème en Python ni en TypeScript |
| **P-A2** | Identifiants **UUID** | Entiers auto-incrémentés par PostgreSQL | Le Core (Python pur, sans base) crée l'`id` d'une commande **avant** qu'elle soit enregistrée, et le transmet à l'agent (DIAG-6 (a)) |
| **P-A3** | Dates ISO 8601 **UTC** avec millisecondes ; l'interface convertit en heure locale | Heure locale dans l'API | Une seule référence ; les millisecondes sont nécessaires pour la latence |
| **P-A4** | **Format unique des erreurs** `{ erreur: { code, message, action_possible, details } }`, y compris pour les 422 de FastAPI | Format par défaut de FastAPI (`{ detail }`, qui change selon l'erreur) | FW-37 : toujours la cause et l'action possible ; une seule fonction d'affichage dans l'interface |
| **P-A5** | Pagination `limite` / `decalage` | Pagination par curseur | Suffisant pour un seul PC et quelques milliers d'événements ; plus simple à comprendre |
| **P-A6** | `NiveauQualite` = `suffisante` / `insuffisante` au MVP | 3 ou 5 niveaux, ou un score 0–100 | DIAG-5 ⑤ ne distingue que ces deux états ; un score pourra s'ajouter sans casser le contrat |

### 7.2 Partie 2 — validées le 05/10/2026 (D-106)

| N° | Proposition | Alternative écartée | Pourquoi |
|---|---|---|---|
| **P-A7** | Une transition d'état = un **`POST` sur un verbe** (`/sessions/{id}/demarrer`) | `PATCH /sessions/{id}` avec `{ "statut": "en_cours" }` | Une transition n'est pas une simple modification de champ : elle est **vérifiée** par la machine à états et a des effets (journal, suspension des commandes). Un verbe par transition rend chaque règle de DIAG-5 visible et testable |
| **P-A8** | **202** pour connecter et reconnecter la source ; le résultat arrive par `/ws/flux` | Attendre la fin de la connexion avant de répondre | Une connexion peut durer plusieurs secondes ou échouer ; l'interface ne doit jamais rester figée (FW-49) |
| **P-A9** | `/api/health` hors de `/v1` | `/api/v1/health` | Route technique, prévue telle quelle au Planning (S4) ; elle ne dépend pas de la version du contrat |
| **P-A10** | Rôles minimaux par route (colonnes « Rôle ») en attendant D-09 ; **suspendre** (système et session) permis à **toute personne connectée** | Tout réservé à l'accompagnant | Principe de sûreté : n'importe qui doit pouvoir arrêter, seuls certains peuvent (ré)activer |

### 7.3 Partie 3 — à valider

| N° | Proposition | Alternative écartée | Pourquoi |
|---|---|---|---|
| **P-A11** | **Enveloppe commune** `{ type, horodatage, sequence, donnees }` pour tous les messages de `/ws/flux` | Un format différent par message | Un seul code de réception dans l'interface ; `sequence` révèle les messages perdus |
| **P-A12** | `/ws/flux` **sert à recevoir** : le navigateur n'envoie que `abonner` / `desabonner` ; **toutes les actions passent par REST** | Envoyer aussi les actions (confirmer, activer) par WebSocket | Un seul chemin pour les actions = droits, erreurs (1.5) et tests au même endroit |
| **P-A13** | Signal envoyé **seulement aux abonnés**, par paquets d'environ 0,1 s ; canaux choisis à l'abonnement (FW-12) | Envoyer tout le signal à toutes les pages | C'est le seul flux lourd ; inutile de le calculer et de l'envoyer à une page qui ne l'affiche pas. Taille exacte des paquets : Documentation technique |
| **P-A14** | **Battement** toutes les 5 s ; « données périmées » affiché après 10 s sans aucun message | Pas de battement : impossible de distinguer « rien ne change » et « connexion morte » | FW-49 : ne jamais afficher un état périmé comme actuel |
| **P-A15** | Jeton de l'agent dans son **premier message** `presentation` ; jeton de l'arrêt d'urgence dans l'**en-tête** `X-Jeton-Local` | Jeton dans l'adresse (`?jeton=…`) | Une adresse peut finir dans des journaux techniques ; un message ou un en-tête, non |
| **P-A16** | Arrêt d'urgence **toujours 200**, même si CortexOS est déjà en état sûr ou arrêté | 409 si déjà arrêté | Le programme C18 doit être le plus simple et le plus fiable possible : appuyer = résultat garanti |



## 8. Points ouverts

- **D-03** : noms des intentions (`"main_gauche"`… sont des exemples).
- **D-09** : droits par rôle (partie 2 : rôle proposé par route).
- **D-10, D-23, D-30, D-31** : valeurs des paramètres de sûreté.
- **D-24** : liste des commandes sensibles · **D-27** : commandes manuelles · **D-68** : délai du résultat.
- **D-84** : la décision `repos` compte-t-elle comme un rejet dans les mesures ?
- Calcul de la qualité et niveaux fins : Documentation technique.
- **D-20** : modification de la correspondance (`PUT …/correspondance`) · **D-23** : qui modifie le seuil · **D-38** : périmètre de la suppression · **D-67** : conditions d'activation · **D-71** : reprise d'une session interrompue.
- **Protocole** d'expérimentation (Q2 de DIAG-7) : champs à ajouter à `POST /sessions`.
- **Formats d'export** : Documentation technique.
- **D-19** : qui peut déclencher l'arrêt d'urgence · **D-68** : délai d'attente du résultat de l'agent.
- **Extension possible (non retenue au MVP)** : `GET /sessions/{id}/decisions`, liste des objets `Decision` d'une session pour un écran d'analyse détaillée (idée d'Eloge, 05/10) ; en attendant, l'historique est dans `GET /journal?type=decision` et dans l'export.
