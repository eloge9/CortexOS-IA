# Cahier des charges — CortexOS IA

| | |
|---|---|
| **Projet** | CortexOS IA |
| **Cadre** | PPE — Projet Professionnel Étudiant |
| **Parcours / Domaine / Spécialité** | Licence — Informatique — Génie Logiciel |
| **Pays** | République Togolaise |
| **Année universitaire** | `[À DÉFINIR — D-14]` |
| **Auteur** | GOMINA Eloge |
| **Encadreur junior / senior** | `[À DÉFINIR — D-14]` |
| **Version du document** | 2.0 — 23 septembre 2026 |

> Convention : `[À DÉFINIR — D-xx]` signale une décision non encore prise, détaillée dans [05-decisions.md](05-decisions.md). `[SOURCE À AJOUTER]` signale une affirmation factuelle qui doit être sourcée avant diffusion.

---

## Sommaire

1. Contexte et situation-problème
2. Étude de l'existant
3. Objectifs et critères de réussite
4. Périmètre
5. Public cible et acteurs
6. Besoins fonctionnels
7. Besoins non fonctionnels
8. Contraintes
9. Hypothèses et risques
10. Architecture de principe
11. Méthodologie
12. Jalons
13. Livrables
14. Budget
15. Conclusion

Annexe A — Glossaire · Annexe B — Décisions et questions ouvertes · Bibliographie

---

## 1. Contexte et situation-problème

### 1.1 Contexte

Les interfaces cerveau-ordinateur (*Brain-Computer Interface*, BCI) permettent de détecter certaines activités cérébrales et de les associer à des commandes numériques. Les approches non invasives, fondées sur l'électroencéphalographie (EEG), utilisent un casque posé sur le cuir chevelu, sans intervention chirurgicale. Elles sont utilisées en recherche, en rééducation, en neurofeedback et, de manière expérimentale, pour le contrôle de dispositifs `[SOURCE À AJOUTER]`.

Au Togo, les travaux et solutions fondés sur les BCI restent peu développés et reposent principalement sur des technologies importées `[SOURCE À AJOUTER]`.

### 1.2 Situation actuelle

L'interaction avec les ordinateurs et les systèmes connectés repose principalement sur des interfaces physiques (clavier, souris, écran tactile, boutons), qui supposent une capacité motrice suffisante. Des alternatives existent, comme la commande vocale, le suivi du regard ou les contacteurs adaptés, mais chacune impose ses propres conditions d'utilisation `[SOURCE À AJOUTER]`.

En parallèle, des casques EEG non invasifs et des bibliothèques logicielles d'acquisition et d'apprentissage automatique sont devenus plus accessibles `[SOURCE À AJOUTER]`. Il devient donc possible d'expérimenter, en dehors des grands laboratoires, la détection d'un petit nombre d'intentions à partir de l'activité cérébrale.

### 1.3 Difficultés constatées

- **Pour les personnes ayant des limitations motrices**, les interfaces physiques peuvent être difficiles, voire impossibles, à utiliser de manière autonome, et les alternatives existantes ne conviennent pas à toutes les situations ni à tous les profils `[SOURCE À AJOUTER]`.
- **Pour l'ensemble des utilisateurs**, l'interaction reste liée à un geste physique. Les solutions BCI accessibles au grand public sont surtout orientées vers le bien-être ou un usage unique, et restent souvent liées à un fabricant `[SOURCE À AJOUTER]`. Les solutions existantes sont détaillées en section 2.
- **Sur le plan technique**, les signaux EEG sont faibles, bruités et varient d'une personne à l'autre et d'une session à l'autre. Une erreur de détection peut se traduire par une commande involontaire. Les performances publiées sont le plus souvent obtenues en laboratoire, avec du matériel spécifique `[SOURCE À AJOUTER]`.

### 1.4 Problème identifié

On ne sait pas précisément, dans des conditions réalistes et avec du matériel EEG non invasif à coût accessible, quelles commandes peuvent être déclenchées de manière fiable et sûre à partir d'intentions détectées, ni dans quelles conditions ce canal peut compléter les interfaces classiques, que ce soit pour des personnes ayant des limitations motrices ou pour des utilisateurs sans handicap. Sans mesure de ses performances, sans visibilité sur ce qui est détecté et sans garde-fous, un tel canal ne peut pas être utilisé pour agir sur un système réel.

### 1.5 Problématique

> **Dans quelle mesure un ensemble limité d'intentions, détectées à partir de signaux EEG non invasifs et interprétées par l'IA, peut-il être traduit en commandes fiables, sûres et observables pour des systèmes informatiques ou connectés, en complément ou en alternative aux interfaces classiques, et quelles sont les limites de cette chaîne ?**

### 1.6 Vision

**Vision courte**

> CortexOS IA est une plateforme expérimentale qui détecte, à partir de signaux EEG, un ensemble limité d'intentions préalablement définies et détectables par le système, et les traduit en commandes supervisées pour des systèmes informatiques ou connectés. Elle vise à explorer une nouvelle forme d'interaction homme-machine, au service de l'accessibilité comme de l'usage général, et à en mesurer objectivement les capacités et les limites.

**Vision développée**

CortexOS IA est une plateforme expérimentale de supervision et de contrôle. Le système ne « lit » pas les pensées : il reconnaît uniquement un ensemble limité d'intentions préalablement définies et détectables par le système. La méthode de détection et d'apprentissage est précisée dans les documents techniques.

Le projet repose sur trois dimensions, qui sont des objectifs à part entière :

- **Accessibilité** : offrir aux personnes ayant des limitations motrices une modalité d'interaction complémentaire, ou alternative, au clavier, à la souris, à l'écran tactile et aux autres interfaces physiques, pour interagir avec un ordinateur ou des systèmes connectés.
- **Nouvelle forme d'interaction homme-machine** : permettre à toute personne, avec ou sans handicap, d'explorer une interaction fondée sur des intentions détectées. L'objectif n'est pas de remplacer de manière générale les interfaces classiques, mais d'explorer leur complémentarité, ou leur remplacement, pour certaines commandes et certains usages.
- **Plateforme expérimentale** : mesurer objectivement les capacités et les limites de la chaîne EEG → traitement → IA → intention détectée → commande → système, en particulier la précision, la latence, la fiabilité, les erreurs, les conditions expérimentales et l'efficacité des mécanismes qui empêchent les commandes involontaires.

Ces dimensions ont des rôles complémentaires. L'accessibilité et l'interaction homme-machine définissent pour qui et pour quels usages CortexOS est conçu. La dimension expérimentale définit comment ses capacités et ses limites sont évaluées et comprises. Sans elle, les deux usages ne pourraient pas être évalués de façon objective.

Les trois dimensions partagent une même chaîne technique. La **supervision** en temps réel (qualité du signal, intention détectée, niveau de confiance, commande exécutée) leur est commune : elle permet à l'utilisateur de comprendre ce que fait le système, elle permet de sécuriser l'exécution des commandes, et elle fournit les données nécessaires à la mesure.

CortexOS est conçu pour pouvoir s'étendre progressivement de l'ordinateur aux objets connectés, aux systèmes embarqués, puis à la robotique et aux drones. Seule une partie de cette trajectoire relève du MVP (section 4).

---

## 2. Étude de l'existant

### 2.1 Tableau comparatif

| Solution | Type | Usage principal | Ouverture logicielle | Limite au regard de ce projet |
|---|---|---|---|---|
| Neuralink | Implant (invasif) | Recherche clinique, contrôle d'ordinateur | Propriétaire | Chirurgie, accès très restreint |
| Synchron | Implant endovasculaire (invasif) | Paralysie sévère, communication assistée | Propriétaire | Chirurgie, usage strictement médical |
| OpenBCI | Casque et cartes EEG non invasifs | Recherche, développement BCI | Ouverte (matériel et logiciel) | Pas de chaîne intégrée jusqu'à l'action |
| Emotiv | Casque EEG non invasif | Recherche, applications commerciales | Propriétaire (SDK sous licence) | Personnalisation et licence |
| Muse | Bandeau EEG non invasif | Méditation, neurofeedback | Propriétaire | Peu de capteurs, orienté bien-être |
| **CortexOS IA** | Logiciel, casque EEG non invasif | Les trois dimensions de la section 1.6 | `[À DÉFINIR — D-08]` | Visé : MVP / extension / futur (section 4) |

Les caractéristiques du tableau sont à vérifier et à sourcer `[SOURCE À AJOUTER]`.

### 2.2 Limites des solutions existantes

- Les solutions les plus performantes sont invasives et réservées au cadre médical.
- Les solutions non invasives sont principalement des outils d'acquisition, ou sont orientées vers un usage unique (bien-être, neurofeedback).
- Les logiciels sont souvent liés à un fabricant de casque.
- Peu de solutions rendent visible, pour l'utilisateur ou son accompagnant, ce qui a été détecté, avec quelle confiance, et pourquoi une commande a été exécutée `[SOURCE À AJOUTER]`.

### 2.3 Positionnement

CortexOS IA ne développe pas de matériel EEG. Il s'appuie sur des casques non invasifs existants et se concentre sur la chaîne logicielle qui va de l'intention détectée jusqu'à la commande exécutée, avec trois éléments : la supervision de cette chaîne, des garde-fous contre les commandes involontaires, et la mesure de ses performances. Ce positionnement sert à la fois l'accessibilité, l'exploration d'une nouvelle interaction et l'expérimentation.

---

## 3. Objectifs et critères de réussite

### 3.1 Objectif général

Concevoir et évaluer une plateforme qui traduit un ensemble limité d'intentions détectées par EEG en commandes fiables, sûres et observables pour des systèmes informatiques ou connectés, afin d'explorer cette modalité d'interaction pour l'accessibilité et pour l'usage général, et d'en mesurer objectivement les capacités et les limites.

### 3.2 Objectifs spécifiques

| ID | Dimension | Objectif |
|---|---|---|
| OS-A1 | Accessibilité | Concevoir une modalité d'interaction utilisable sans geste physique fin |
| OS-A2 | Accessibilité | Identifier les besoins et conditions d'usage des personnes ayant des limitations motrices `[À DÉFINIR — D-12 : méthode]` |
| OS-I1 | Interaction | Permettre de déclencher certaines commandes à partir d'intentions détectées, en complément ou à la place des interfaces classiques |
| OS-I2 | Interaction | Identifier les commandes et les usages pour lesquels ce canal est pertinent |
| OS-E1 | Expérimentation | Mesurer la précision, la latence, la fiabilité et les erreurs de la chaîne |
| OS-E2 | Expérimentation | Documenter les conditions expérimentales et les limites observées, y compris les résultats négatifs |
| OS-E3 | Expérimentation | Évaluer l'efficacité des mécanismes contre les commandes involontaires |
| OS-T1 | Transversal | N'exécuter aucune commande sans niveau de confiance suffisant, et permettre l'arrêt à tout moment |
| OS-T2 | Transversal | Rendre observable chaque étape de la chaîne |
| OS-T3 | Transversal | Permettre l'ajout de nouveaux systèmes cibles sans refonte du Core |
| OS-T4 | Transversal | Protéger les données EEG et les données personnelles |

### 3.3 Critères de réussite du MVP

| Critère | Ce que le MVP mesure | Seuil |
|---|---|---|
| Précision de détection | Taux de détection correcte des intentions, comparé au hasard | `[À DÉFINIR — D-10]` |
| Latence | Délai de bout en bout entre la fenêtre de signal analysée et la commande exécutée | `[À DÉFINIR — D-10]` |
| Commandes involontaires | Taux de commandes exécutées sans intention correspondante | `[À DÉFINIR — D-10]` |
| Réussite du scénario | Taux de réussite du ou des scénarios de démonstration | `[À DÉFINIR — D-05, D-10]` |
| Accessibilité de la boucle de contrôle | Le scénario peut être réalisé sans geste physique fin | Oui / Non |
| Documentation | Résultats, conditions et limites consignés dans le rapport d'expérimentation | Rapport livré |

Le bénéfice réel pour des personnes en situation de handicap **n'est pas un critère du MVP**, sauf si des tests avec ce public sont décidés (`[À DÉFINIR — D-12]`, voir 4.3).

---

## 4. Périmètre

### 4.1 Trajectoire des systèmes cibles

| Système cible | Horizon |
|---|---|
| Ordinateur | MVP ou extension `[À DÉFINIR — D-05]` |
| Objets connectés | MVP ou extension `[À DÉFINIR — D-05]` |
| Systèmes embarqués | Extension |
| Robotique | Extension en simulation ou recherche et futur `[À DÉFINIR — D-11]` |
| Drones | Recherche et futur `[À DÉFINIR — D-11]` |

Au moins un système cible réel fait partie du MVP. La progression suit un principe : **aucun système plus complexe n'est intégré tant que la chaîne EEG → IA → commande n'a pas été validée sur un système plus simple.**

### 4.2 MVP : ce qui est implémenté

- La chaîne complète : acquisition EEG, traitement, détection d'intention avec niveau de confiance, association intention → commande, exécution.
- Les garde-fous : rejet des détections de confiance insuffisante, confirmation des commandes sensibles, arrêt `[À DÉFINIR — D-19]`.
- Un connecteur vers au moins un système cible réel `[À DÉFINIR — D-05]`.
- La supervision en temps réel de la chaîne.
- L'enregistrement des sessions et le calcul des mesures de la section 3.3.
- La journalisation des détections, commandes et erreurs.
- La gestion des profils et du consentement `[À DÉFINIR — D-09 : forme]`.

Les intentions retenues (`[À DÉFINIR — D-03]`) et le casque utilisé (`[À DÉFINIR — D-04]`) ne sont pas encore décidés.

### 4.3 MVP : ce qui est démontré et ce qui ne l'est pas

| Le MVP vise à démontrer | Le MVP ne démontre pas |
|---|---|
| Qu'un ensemble limité d'intentions peut être détecté avec une précision mesurée, dans les conditions du projet | Que CortexOS est utile aux personnes en situation de handicap, sauf si des tests avec ce public sont réalisés `[À DÉFINIR — D-12]` |
| Qu'une intention détectée peut déclencher une commande réelle sur un système cible | Que l'interaction par EEG est plus efficace que les interfaces classiques |
| Que les garde-fous limitent les commandes involontaires | Des résultats généralisables à une large population |
| Que la chaîne est observable et mesurable | Un usage autonome et prolongé hors cadre expérimental |

L'accessibilité reste une dimension du projet et une contrainte de conception du MVP. Elle n'est pas un résultat démontré par le MVP.

### 4.4 Extensions

Extensions envisagées après validation du MVP, sans ordre ni engagement :

- d'autres systèmes cibles (ordinateur ou objets connectés, selon D-05) ;
- des systèmes embarqués ;
- le robot en simulation `[À DÉFINIR — D-11]` ;
- l'adaptation progressive du modèle à l'utilisateur ;
- une association intention → commande configurable par profil `[À DÉFINIR — D-20]`.

### 4.5 Recherche et futur

- Robotique physique et drones.
- Combinaison de l'EEG avec d'autres modalités (regard, voix).
- Évaluation avec des publics ayant différents types de handicap, dans un cadre éthique adapté.

### 4.6 Hors périmètre

- Le pilotage d'applications précises (traitement de texte, messagerie, environnements de développement, jeux).
- Les extensions ou plugins pour des logiciels tiers.
- Le pilotage d'un drone réel dans le cadre du MVP.
- Les robots humanoïdes ou quadrupèdes.
- Le développement de matériel EEG.

### 4.7 Statut du projet

- CortexOS IA **n'est pas un dispositif médical**. Il ne doit pas être présenté comme tel sans les validations et certifications nécessaires.
- CortexOS IA **ne lit pas les pensées**. Il détecte uniquement un ensemble limité d'intentions préalablement définies et détectables par le système.
- CortexOS IA **ne remplace pas** les accompagnants ni les professionnels de l'assistance.

---

## 5. Public cible et acteurs

### 5.1 Public potentiel

| Public | Dimension | Attente |
|---|---|---|
| Personnes ayant des limitations motrices | Accessibilité | Disposer d'une modalité d'interaction complémentaire ou alternative |
| Utilisateurs avec ou sans handicap | Interaction | Explorer une interaction fondée sur des intentions détectées |
| Chercheurs, étudiants, développeurs | Expérimentation | Disposer de résultats mesurés et d'une base de travail réutilisable |

Les familles, accompagnants et professionnels de la réadaptation constituent un public indirect.

### 5.2 Acteurs du système

| Acteur | Rôle |
|---|---|
| Utilisateur | Porte le casque, réalise la calibration et déclenche des commandes par ses intentions |
| Accompagnant / opérateur | Suit la session via la supervision et peut interrompre le système `[À DÉFINIR — D-09, D-19]` |
| Expérimentateur | Prépare et conduit les sessions, analyse les mesures |
| Casque EEG | Fournit le signal |
| Système cible | Reçoit et exécute les commandes |

### 5.3 Participants aux expérimentations

Le profil des participants au MVP est `[À DÉFINIR — D-12]`. Toute expérimentation respecte le consentement éclairé, la possibilité de se retirer à tout moment et la confidentialité des données. Une expérimentation avec des personnes en situation de handicap exige des précautions éthiques renforcées et un encadrement adapté `[SOURCE À AJOUTER : règles applicables]`.

---

## 6. Besoins fonctionnels

Priorités : **MVP**, **Ext.** (extension), **Futur**.

### 6.1 Acquisition du signal

| ID | Exigence | Priorité |
|---|---|---|
| BF-01 | Se connecter à un casque EEG compatible et s'en déconnecter | MVP |
| BF-02 | Indiquer la qualité du signal et l'état de connexion du casque | MVP |
| BF-03 | Démarrer et arrêter une session d'acquisition | MVP |

### 6.2 Traitement du signal

| ID | Exigence | Priorité |
|---|---|---|
| BF-04 | Réduire le bruit et les artefacts du signal avant la détection | MVP |
| BF-05 | Extraire du signal les éléments nécessaires à la détection des intentions | MVP |

### 6.3 Calibration et détection

| ID | Exigence | Priorité |
|---|---|---|
| BF-06 | Guider l'utilisateur pendant une session de calibration | MVP |
| BF-07 | Construire un modèle de détection propre à l'utilisateur | MVP |
| BF-08 | Détecter les intentions retenues `[À DÉFINIR — D-03]` et associer à chaque détection un niveau de confiance | MVP |
| BF-09 | Adapter progressivement le modèle à l'utilisateur | Ext. |

### 6.4 Décision et sûreté (CortexOS Core)

| ID | Exigence | Priorité |
|---|---|---|
| BF-10 | Associer une intention détectée à une commande | MVP |
| BF-11 | Rejeter toute détection dont le niveau de confiance est inférieur au seuil défini | MVP |
| BF-12 | Demander une confirmation avant l'exécution d'une commande sensible | MVP |
| BF-13 | Permettre l'arrêt immédiat de toute exécution `[À DÉFINIR — D-19]` | MVP |
| BF-14 | Permettre de configurer l'association intention → commande par profil | `[À DÉFINIR — D-20]` |

### 6.5 Systèmes cibles

| ID | Exigence | Priorité |
|---|---|---|
| BF-15 | Transmettre une commande à au moins un système cible réel et recevoir son résultat `[À DÉFINIR — D-05]` | MVP |
| BF-16 | Ajouter un nouveau type de système cible sous forme de connecteur | Ext. |

### 6.6 Supervision

| ID | Exigence | Priorité |
|---|---|---|
| BF-17 | Afficher en temps réel le signal, l'intention détectée, son niveau de confiance et la commande exécutée | MVP |
| BF-18 | Signaler les erreurs, rejets et interruptions | MVP |

### 6.7 Expérimentation

| ID | Exigence | Priorité |
|---|---|---|
| BF-19 | Enregistrer les sessions : signal, détections, commandes, conditions | MVP |
| BF-20 | Calculer les mesures de la section 3.3 | MVP |
| BF-21 | Exporter les données et résultats d'une session | MVP |

### 6.8 Profils, données et journalisation

| ID | Exigence | Priorité |
|---|---|---|
| BF-22 | Gérer les profils utilisateurs et leurs modèles de détection `[À DÉFINIR — D-09]` | MVP |
| BF-23 | Recueillir et enregistrer le consentement de l'utilisateur | MVP |
| BF-24 | Permettre à l'utilisateur de supprimer ses données | MVP |
| BF-25 | Journaliser les détections, commandes, confirmations, rejets et erreurs | MVP |

---

## 7. Besoins non fonctionnels

| ID | Catégorie | Exigence |
|---|---|---|
| ENF-01 | Performance | La latence de bout en bout reste inférieure à la cible fixée `[À DÉFINIR — D-10]` |
| ENF-02 | Fiabilité et robustesse | Une perte de signal ou de connexion n'entraîne aucune commande ; le système le signale et revient à un état sûr |
| ENF-03 | Sûreté | En cas de doute (confiance insuffisante, erreur, déconnexion), le système n'exécute aucune commande |
| ENF-04 | Sûreté | Un moyen d'arrêt indépendant de la détection EEG est disponible `[À DÉFINIR — D-19 : mécanisme et déclencheur]` |
| ENF-05 | Sécurité | L'accès aux données et aux fonctions d'administration est contrôlé `[À DÉFINIR — D-09]` ; les échanges réseau sont protégés |
| ENF-06 | Confidentialité | Les données EEG ne sont collectées et utilisées qu'avec le consentement explicite de l'utilisateur ; leur lieu de stockage et leur durée de conservation sont définis `[À DÉFINIR — D-09]` |
| ENF-07 | Maintenabilité et évolutivité | L'architecture est modulaire : un nouveau système cible ou une nouvelle méthode de détection s'ajoute sans modifier le Core |
| ENF-08 | Compatibilité | La plateforme fonctionne avec le casque `[À DÉFINIR — D-04]` et le système d'exploitation `[À DÉFINIR — D-06]` retenus |
| ENF-09 | Utilisabilité et accessibilité | L'état du système et la raison de chaque rejet ou commande sont compréhensibles par l'utilisateur et l'accompagnant ; les fonctions d'usage ne nécessitent pas de geste physique fin |
| ENF-10 | Traçabilité | Toute session peut être rejouée ou analysée à partir de ses enregistrements |

---

## 8. Contraintes

### 8.1 Techniques

- Le signal EEG est de très faible amplitude et sensible aux mouvements, aux clignements, aux interférences électriques et au positionnement des électrodes.
- Une même intention produit des signaux différents selon les personnes et les sessions (fatigue, concentration). Une calibration par utilisateur est nécessaire.
- La détection doit rester compatible avec une utilisation en temps réel.

### 8.2 Matérielles

Le casque doit être non invasif, permettre une acquisition en temps réel et disposer d'un SDK ou d'une compatibilité avec une bibliothèque d'acquisition ouverte. Le modèle et le nombre de canaux sont `[À DÉFINIR — D-04]`.

### 8.3 Temporelles et humaines

Le projet est réalisé dans le cadre d'un PPE de Licence. L'échéance et la composition de l'équipe sont `[À DÉFINIR — D-14]`. Le périmètre du MVP est dimensionné en conséquence (section 4).

### 8.4 Budgétaires

Le matériel est limité aux éléments indispensables au MVP. Les technologies open source sont privilégiées (section 14, `[À DÉFINIR — D-13]`).

### 8.5 Éthiques et juridiques

- Les données EEG sont des données personnelles sensibles. Elles relèvent du cadre juridique applicable à la protection des données personnelles au Togo `[SOURCE À AJOUTER]`.
- Toute expérimentation repose sur un consentement éclairé (section 5.3).
- Aucune commande ne doit pouvoir mettre en danger l'utilisateur ou son environnement (ENF-03, ENF-04).

---

## 9. Hypothèses et risques

### 9.1 Hypothèses

| ID | Hypothèse |
|---|---|
| H1 | Un casque compatible est disponible à temps pour le développement de la chaîne |
| H2 | Un petit nombre d'intentions peut être détecté au-dessus du hasard avec le matériel retenu |
| H3 | Des participants volontaires sont disponibles pour les sessions d'expérimentation |
| H4 | Le matériel informatique disponible suffit pour entraîner et exécuter les modèles `[À DÉFINIR — D-13]` |

### 9.2 Risques

| ID | Risque | Impact | Piste de maîtrise |
|---|---|---|---|
| R1 | Dilution du périmètre entre les trois dimensions | MVP trop large | Une chaîne commune ; les extensions sont séparées (section 4) |
| R2 | Promesse non démontrée sur l'accessibilité | Perte de crédibilité, problème éthique | Distinction entre implémenté et démontré (section 4.3) |
| R3 | Présentation de l'EEG comme remplaçant général des interfaces classiques | Objectif intenable | Vocabulaire « certaines commandes », « en complément » |
| R4 | Perception de « lecture des pensées » | Malentendu, crainte | Section 4.7, glossaire |
| R5 | Précision faible hors laboratoire | Démonstration difficile | Seuils définis (D-10) ; les résultats négatifs sont documentés ; repli `[À DÉFINIR — D-21]` |
| R6 | Commandes involontaires | Action non voulue sur un système réel | BF-11 à BF-13, ENF-03, ENF-04 |
| R7 | Fatigue et durée de calibration | Résultats instables | Sessions courtes ; effets mesurés et documentés |
| R8 | Contraintes éthiques sur les données et les participants | Expérimentation retardée ou limitée | Consentement, D-09, D-12 |
| R9 | Indisponibilité ou retard du casque | Retard du projet | D-04 ; repli `[À DÉFINIR — D-21]` |
| R10 | Charge de travail | Retard | Priorisation stricte du MVP ; D-14 |

---

## 10. Architecture de principe

```
                        ┌────────────────────────────────────┐
                        │      Supervision (transversale)    │
                        └──────────────────┬─────────────────┘
                                           │ observe chaque étape
 Casque EEG → Acquisition → Traitement → Détection d'intention → CortexOS Core → Connecteur → Système cible
                                          (+ niveau de confiance)  (décision et sûreté)
                                           │
                        Enregistrement des sessions et mesures (expérimentation)
```

| Module | Rôle |
|---|---|
| Acquisition | Recevoir le signal du casque et contrôler sa qualité |
| Traitement | Réduire le bruit et préparer le signal pour la détection |
| Détection d'intention | Reconnaître les intentions retenues et estimer un niveau de confiance |
| CortexOS Core | Décider : associer l'intention à une commande, appliquer les garde-fous, router la commande |
| Connecteurs | Traduire une commande pour un type de système cible |
| Supervision | Rendre la chaîne observable en temps réel |
| Enregistrement et mesures | Conserver les sessions et calculer les indicateurs |

| Domaine | Technologie | Statut |
|---|---|---|
| Traitement EEG et IA | Python | Retenu |
| Backend / API | FastAPI | Retenu |
| Interface de supervision | Next.js (TypeScript) | Retenu |
| CortexOS Core | C++ | Retenu à terme ; langage du MVP `[À DÉFINIR — D-07]` |
| Acquisition EEG | Bibliothèque compatible avec plusieurs casques (ex. BrainFlow) | Envisagé `[À DÉFINIR — D-04]` |
| Base de données | PostgreSQL | Envisagé |
| Objets connectés / embarqué | MQTT, ESP32 | Envisagé `[À DÉFINIR — D-05]` |
| Robotique | ROS 2 | Extension `[À DÉFINIR — D-11]` |
| Autres (Django, Flutter, Redis, gRPC, Docker) | — | `[À DÉFINIR — D-08]` |

Le détail de l'architecture (composants, protocoles, modèle de données, diagrammes UML, déploiement) est traité dans le document **Architecture technique**.

---

## 11. Méthodologie

- **Démarche incrémentale** : la chaîne est d'abord construite et validée de bout en bout sur un système cible simple, puis étendue. Cet ordre suit les niveaux d'interaction : numérique, puis environnement, puis robotique.
- **Conception centrée sur l'utilisateur** : les commandes et les scénarios sont choisis en fonction de leur utilité réelle et de leur faisabilité, avant tout ajout de fonctionnalité.
- **Tests** : tests unitaires et d'intégration des modules, mesures de performance, et évaluation expérimentale selon la section 3.3. Le protocole est décrit dans la **Documentation technique**.
- **Suivi** : l'organisation détaillée du travail est décrite dans le **Planning MVP**.

---

## 12. Jalons

| Jalon | Résultat attendu | Date |
|---|---|---|
| J1 | Cahier des charges validé | `[À DÉFINIR — D-14]` |
| J2 | Architecture de principe validée | `[À DÉFINIR — D-14]` |
| J3 | Acquisition EEG fonctionnelle et signal exploitable | `[À DÉFINIR — D-14]` |
| J4 | Première détection d'intention mesurée | `[À DÉFINIR — D-14]` |
| J5 | Première commande réelle exécutée avec garde-fous | `[À DÉFINIR — D-14]` |
| J6 | Supervision opérationnelle | `[À DÉFINIR — D-14]` |
| J7 | Campagne d'évaluation terminée et résultats documentés | `[À DÉFINIR — D-14]` |
| J8 | Soutenance | `[À DÉFINIR — D-14]` |

Le découpage détaillé en tâches et en itérations est décrit dans le **Planning MVP**.

---

## 13. Livrables

| Type | Livrable |
|---|---|
| Logiciel | Code source de la plateforme (chaîne complète, supervision, enregistrement et mesures) |
| Démonstrateur | Le ou les scénarios de démonstration du MVP `[À DÉFINIR — D-05]` |
| Scientifique | Rapport d'expérimentation : protocole, conditions, mesures, erreurs, limites, y compris les résultats négatifs ; données de sessions si le consentement le permet `[À DÉFINIR — D-09]` |
| Documentaire | Cahier des charges, Fonctionnalités Web, Architecture technique, Documentation technique, Planning MVP, manuel d'installation et d'utilisation, rapport final, support de soutenance |

---

## 14. Budget

| Poste | Estimation |
|---|---|
| Casque EEG | `[À DÉFINIR — D-04, D-13]` |
| Matériel de démonstration pour le système cible (microcontrôleur, composants) | `[À DÉFINIR — D-05, D-13]` |
| Ordinateur de développement | `[À DÉFINIR — D-13 : matériel existant ou achat]` |
| Logiciels | Open source ou gratuits |
| **Total** | `[À DÉFINIR — D-13]` |

Le financement est également `[À DÉFINIR — D-13]`.

---

## 15. Conclusion

CortexOS IA vise à traduire un ensemble limité d'intentions détectées par EEG en commandes fiables, sûres et observables, au service de trois dimensions : l'accessibilité, une nouvelle forme d'interaction homme-machine et l'expérimentation. Le MVP se concentre sur une chaîne complète, supervisée et mesurée, appliquée à au moins un système cible réel. Il distingue clairement ce qu'il implémente de ce qu'il démontre. La trajectoire vers les objets connectés, les systèmes embarqués, la robotique et les drones reste ouverte, sous réserve que chaque étape soit validée avant la suivante.

---

## Annexe A — Glossaire

| Terme | Définition |
|---|---|
| **Intention** | État ou activité mentale, préalablement défini et détectable par le système, que l'utilisateur produit volontairement |
| **Détection** | Résultat produit par le système à partir du signal : une intention reconnue et son niveau de confiance |
| **Niveau de confiance** | Estimation, par le système, de la fiabilité d'une détection |
| **Commande** | Instruction décidée par le Core à partir d'une détection et destinée à un système cible |
| **Action** | Effet produit par le système cible après exécution d'une commande |
| **Commande involontaire** | Commande exécutée sans que l'utilisateur ait produit l'intention correspondante |
| **Système cible** | Tout système qui reçoit les commandes : ordinateur, objet connecté, système embarqué, robot, drone |
| **Calibration** | Session guidée permettant de construire ou d'ajuster le modèle de détection d'un utilisateur |
| **Session** | Période continue d'utilisation ou d'expérimentation, du démarrage à l'arrêt de l'acquisition |
| **Supervision** | Affichage en temps réel de l'état de la chaîne : signal, détection, confiance, commande, erreurs |
| **CortexOS IA** | La plateforme dans son ensemble |
| **CortexOS Core** | Le module de décision : association intention → commande et garde-fous |
| **MVP / Extension / Recherche et futur** | Horizons du périmètre définis en section 4 |

---

## Annexe B — Décisions et questions ouvertes

Le registre des décisions est désormais tenu dans [05-decisions.md](05-decisions.md).

---

## Bibliographie

- Nicolas-Alonso, L. F., & Gomez-Gil, J. (2012). *Brain Computer Interfaces, a Review*. Sensors, 12(2), 1211–1279.
- Lotte, F., Bougrain, L., Cichocki, A., Clerc, M., Congedo, M., Rakotomamonjy, A., & Yger, F. (2018). *A review of classification algorithms for EEG-based brain–computer interfaces: a 10 year update*. Journal of Neural Engineering, 15(3), 031005.
- LeCun, Y., Bengio, Y., & Hinton, G. (2015). *Deep learning*. Nature, 521, 436–444.
- `[SOURCE À AJOUTER]` : références pour chaque affirmation marquée dans le document (sections 1, 2, 5.3 et 8.5).
