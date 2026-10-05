# Charte graphique — CortexOS IA

| | |
|---|---|
| **Version** | 1.1 — 05/10/2026 (section 9 « Ce que CortexOS IA ne fait pas », D-109) |
| **Choix validés** | Variables CSS + Tailwind v4 · thème sombre par défaut + thème clair · Exo 2 + Inter · ambiance pro et sobre (D-43) |
| **Fichiers** | `docs/assets/logo/` |

> À partir du squelette frontend (S4), les valeurs de la section 7 vivent dans `frontend/app/globals.css`. Ce fichier-là fait alors foi pour les valeurs ; ce document garde les règles d'usage.

---

## 1. Principes

1. **La lisibilité d'abord.** CortexOS sert à superviser une chaîne qui commande des systèmes réels : on doit comprendre l'état en un coup d'œil.
2. **Les couleurs du logo en accent, pas partout.** Le cyan et le violet signalent ce qui est important ; le reste est neutre.
3. **Jamais l'information par la couleur seule.** Chaque état a une couleur **+ une icône + un texte**.
4. **Le rouge est réservé à l'arrêt et aux erreurs.** Aucun autre élément n'utilise le rouge.

---

## 2. Logo

### 2.1 Versions

| Fichier | Contenu | Usage |
|---|---|---|
| `horizontal-nuit-transparent.png` | Icône + « CortexOS IA », bleu nuit | **Principal** sur fond clair (rapport, documents) |
| `horizontal-blanc-transparent.png` | Icône cyan + texte blanc | **Principal** sur fond sombre (interface, diapositives) |
| `horizontal-nuit-fond-blanc.png` | Idem, fond blanc intégré | Quand la transparence n'est pas gérée |
| `horizontal-blanc-fond-nuit.png` | Idem, fond bleu nuit intégré | Couverture, réseaux, vidéo |
| `icone-nuit-transparent.png` | Icône seule, bleu nuit | Petits espaces sur fond clair |
| `icone-cyan-transparent.png` | Icône seule, cyan | En-tête de l'interface (thème sombre), petits espaces sur fond sombre |
| `icone-nuit-fond-blanc.png` · `icone-cyan-fond-nuit.png` | Icône avec fond | Favicon, icône d'application, avatar |

### 2.2 Règles

- **Taille minimale :** icône 24 px ; logo horizontal 120 px de large. En dessous, utiliser l'icône seule.
- **Zone de protection :** autour du logo, un espace libre égal au quart de la hauteur de l'icône.
- **Graphie :** « CortexOS IA » — C, O et S en majuscules, « IA » séparé (D-42).
- **Interdit :** déformer, pivoter, changer les couleurs, ajouter une ombre ou un effet lumineux, poser le logo sur une photo chargée, recolorer le badge « IA ».

> Les fichiers actuels sont des images (PNG). Une version vectorielle (SVG) sera nécessaire pour un rendu net à toutes les tailles `[À DÉFINIR — D-44]`.

---

## 3. Couleurs

### 3.1 Couleurs de marque (relevées sur le logo)

| Nom | Valeur | Rôle |
|---|---|---|
| Bleu nuit | `#031430` | Fond du thème sombre, texte du thème clair |
| Cyan | `#19C8FB` | Couleur primaire : actions principales, signal EEG, focus |
| Violet | `#5A12E9` | Accent : IA, détection, éléments secondaires mis en avant |

### 3.2 Thème sombre (par défaut)

| Rôle | Variable | Valeur |
|---|---|---|
| Fond de page | `--cx-fond` | `#031430` |
| Surface (cartes, panneaux) | `--cx-surface` | `#0B1E3F` |
| Surface surélevée (menus, fenêtres) | `--cx-surface-2` | `#13294F` |
| Bordure décorative | `--cx-bordure` | `#2A4570` |
| Texte principal | `--cx-texte` | `#E8EEF7` |
| Texte secondaire | `--cx-texte-2` | `#9FB0CA` |
| Primaire | `--cx-primaire` | `#19C8FB` |
| Texte sur primaire | `--cx-sur-primaire` | `#031430` |
| Accent (texte, icônes) | `--cx-accent` | `#A78BFA` |
| Accent (fonds) | `--cx-accent-fond` | `#5A12E9` |

### 3.3 Thème clair

| Rôle | Variable | Valeur |
|---|---|---|
| Fond de page | `--cx-fond` | `#F5F8FC` |
| Surface | `--cx-surface` | `#FFFFFF` |
| Surface surélevée | `--cx-surface-2` | `#FFFFFF` (avec ombre) |
| Bordure décorative | `--cx-bordure` | `#D5DDEA` |
| Texte principal | `--cx-texte` | `#031430` |
| Texte secondaire | `--cx-texte-2` | `#4A5B78` |
| Primaire (texte, liens, boutons) | `--cx-primaire` | `#0077A8` |
| Texte sur primaire | `--cx-sur-primaire` | `#FFFFFF` |
| Accent | `--cx-accent` | `#5A12E9` |
| Accent (fonds) | `--cx-accent-fond` | `#5A12E9` |

> Pourquoi un cyan plus foncé en thème clair ? Le cyan du logo sur fond blanc n'a qu'un contraste de 1,96 : illisible. `#0077A8` garde la même teinte avec un contraste de 5,00.

### 3.4 Couleurs d'état

Les états reprennent ceux de la Spécification fonctionnelle (7 états globaux et cycle de vie d'une commande).

| État | Sombre | Clair | Icône | Exemple de texte |
|---|---|---|---|---|
| Actif / succès | `#34D399` | `#047857` | ✓ cercle | « Commandes actives » |
| Suspendu / attention | `#FBBF24` | `#B45309` | ‖ pause | « Commandes suspendues » |
| Arrêt / erreur | `#F87171` | `#B91C1C` | ■ carré | « Arrêt d'urgence activé » |
| Information / en cours | `--cx-primaire` | `--cx-primaire` | i | « Calibration en cours » |
| Détection IA | `--cx-accent` | `--cx-accent` | onde | « Intention détectée : gauche » |
| Inactif / hors ligne | `--cx-texte-2` | `--cx-texte-2` | cercle vide | « Casque déconnecté » |

**Bouton d'arrêt d'urgence :** fond `#B91C1C`, texte blanc (contraste 6,47), dans les deux thèmes.

### 3.5 Contrastes vérifiés

Seuils WCAG 2.1 : texte normal ≥ 4,5 · grand texte et éléments d'interface ≥ 3.

| Combinaison | Sombre | Clair |
|---|---|---|
| Texte principal sur fond | 15,70 | 17,19 |
| Texte principal sur surface | 14,16 | 18,31 |
| Texte secondaire sur fond | 8,32 | 6,45 |
| Primaire sur fond | 9,33 | 4,69 |
| Texte sur bouton primaire | 9,33 | 5,00 |
| Accent sur fond | 6,73 | 7,78 (sur blanc) |
| Succès / attention / erreur sur fond | 9,53 / 10,97 / 6,62 | 5,48 / 5,02 / 6,47 (sur blanc) |
| Blanc sur bouton d'arrêt | 6,47 | 6,47 |

La bordure décorative (`--cx-bordure`) est volontairement peu contrastée : elle ne porte aucune information. Les bordures des champs de saisie utilisent `--cx-texte-2` (contraste ≥ 3).

---

## 4. Typographie

| Usage | Police | Graisse | Taille |
|---|---|---|---|
| Titre de page (h1) | **Exo 2** | 600 | 32 px |
| Titre de section (h2) | Exo 2 | 600 | 24 px |
| Sous-titre (h3) | Exo 2 | 500 | 20 px |
| Texte courant | **Inter** | 400 | 16 px (jamais moins de 14 px) |
| Libellés, boutons | Inter | 500 | 14–16 px |
| Valeurs chiffrées (confiance, latence) | Inter, chiffres tabulaires | 600 | selon contexte |
| Code, identifiants techniques | police à chasse fixe du système | 400 | 14 px |

- Les deux polices sont gratuites (Google Fonts) et chargées avec `next/font` : pas de requête externe au moment de l'affichage.
- Interligne : 1,5 pour le texte, 1,2 pour les titres.
- Exo 2 est proche du lettrage du logo ; elle ne sert **que** pour les titres.

---

## 5. Mise en page

- **Espacements :** multiples de 4 px (4, 8, 12, 16, 24, 32, 48).
- **Arrondis :** 6 px (champs, boutons) · 12 px (cartes, panneaux).
- **Zone cliquable minimale :** 44 × 44 px (accessibilité, usage avec des limitations motrices).
- **Ombres :** discrètes, surtout en thème clair ; pas d'effet lumineux (« glow ») : ambiance sobre (D-43).
- **Animations :** courtes (150–250 ms), **seulement pour signaler un changement** (ouverture d'un panneau, nouvel état) ; désactivées si l'utilisateur a demandé de réduire les animations (`prefers-reduced-motion`). Voir aussi la section 9.

---

## 6. Composants clés

| Composant | Règles |
|---|---|
| **Bouton d'arrêt d'urgence** | Toujours visible, même position sur tous les écrans, rouge `#B91C1C`, icône ■ + texte « Arrêt », jamais masqué par un menu ou une fenêtre |
| **Bouton primaire** | Fond `--cx-primaire`, texte `--cx-sur-primaire` ; un seul par zone |
| **Bouton secondaire** | Contour `--cx-primaire`, fond transparent |
| **Bandeau d'état global** | En haut de l'écran : couleur d'état + icône + texte (section 3.4) |
| **Jauge de confiance** | Barre + valeur en % en chiffres ; repère visible du seuil de confiance ; en dessous du seuil : texte « Confiance insuffisante » |
| **Courbes EEG** | Fond `--cx-surface`, signal brut en `--cx-texte-2`, signal filtré en `--cx-primaire`, grille en `--cx-bordure` ; chaque canal porte son nom |
| **Alertes** | Couleur d'état + icône + titre + action proposée ; les alertes d'arrêt restent affichées jusqu'à action |
| **Focus clavier** | Contour de 2 px en `--cx-primaire`, décalé de 2 px, sur tous les éléments interactifs |

---

## 7. Mise en œuvre technique

Principe : les couleurs sont des **variables CSS** (standard, indépendantes de Tailwind). Tailwind v4 les lit via `@theme`, ce qui permet d'écrire `bg-fond`, `text-primaire`, etc.

```css
/* frontend/app/globals.css (S4) */
@import "tailwindcss";

/* Thème sombre : par défaut */
:root {
  --cx-fond: #031430;
  --cx-surface: #0B1E3F;
  --cx-surface-2: #13294F;
  --cx-bordure: #2A4570;
  --cx-texte: #E8EEF7;
  --cx-texte-2: #9FB0CA;
  --cx-primaire: #19C8FB;
  --cx-sur-primaire: #031430;
  --cx-accent: #A78BFA;
  --cx-accent-fond: #5A12E9;
  --cx-succes: #34D399;
  --cx-attention: #FBBF24;
  --cx-erreur: #F87171;
  --cx-arret: #B91C1C;
}

/* Thème clair : activé par l'attribut data-theme="clair" sur <html> */
[data-theme="clair"] {
  --cx-fond: #F5F8FC;
  --cx-surface: #FFFFFF;
  --cx-surface-2: #FFFFFF;
  --cx-bordure: #D5DDEA;
  --cx-texte: #031430;
  --cx-texte-2: #4A5B78;
  --cx-primaire: #0077A8;
  --cx-sur-primaire: #FFFFFF;
  --cx-accent: #5A12E9;
  --cx-accent-fond: #5A12E9;
  --cx-succes: #047857;
  --cx-attention: #B45309;
  --cx-erreur: #B91C1C;
  --cx-arret: #B91C1C;
}

/* Tailwind lit les variables : bg-fond, text-texte, bg-primaire, text-succes… */
@theme inline {
  --color-fond: var(--cx-fond);
  --color-surface: var(--cx-surface);
  --color-surface-2: var(--cx-surface-2);
  --color-bordure: var(--cx-bordure);
  --color-texte: var(--cx-texte);
  --color-texte-2: var(--cx-texte-2);
  --color-primaire: var(--cx-primaire);
  --color-sur-primaire: var(--cx-sur-primaire);
  --color-accent: var(--cx-accent);
  --color-accent-fond: var(--cx-accent-fond);
  --color-succes: var(--cx-succes);
  --color-attention: var(--cx-attention);
  --color-erreur: var(--cx-erreur);
  --color-arret: var(--cx-arret);
  --font-titre: var(--font-exo2);
  --font-sans: var(--font-inter);
}
```

Exemple d'utilisation :

```tsx
<button className="bg-arret text-white font-medium rounded-md min-h-11 px-4">■ Arrêt</button>
```

Règle : **aucune couleur écrite en dur dans les composants** (`#19C8FB`, `text-cyan-400`…). On passe toujours par les noms de la charte, pour que le changement de thème fonctionne partout.

---

## 8. Documents et présentations

- Rapport et documents : fond blanc, logo `horizontal-nuit-transparent.png`, titres en Exo 2 bleu nuit, texte en Inter.
- Diapositives de soutenance : fond bleu nuit, logo `horizontal-blanc-transparent.png`, accents cyan et violet.
- Diagrammes (draw.io, fin de projet) : boîtes en surface claire, bordures bleu nuit, flux principal en cyan foncé `#0077A8`, IA en violet.

---

## 9. Ce que CortexOS IA ne fait pas (D-109)

CortexOS IA doit avoir l'air de ce qu'il est : un outil de supervision **sérieux, conçu et compris**. Ces interdits évitent l'apparence d'une application générique « faite en cinq minutes » et protègent l'honnêteté scientifique du projet.

### 9.1 Style

| Interdit | À la place |
|---|---|
| **Dégradés décoratifs** (fonds, textes, boutons, halos), quelle que soit la couleur | Couleurs **unies** de la charte. Seule exception : une échelle de couleur qui **représente une valeur** dans un graphique (ex. matrice de confusion), toujours avec les chiffres |
| **Boutons « pilule »** (entièrement arrondis) | Arrondis de 6 px (section 5) |
| **Émojis** utilisés comme icônes | Icônes dessinées d'une seule bibliothèque, au trait régulier (choix en S4) ; toujours avec un texte (principe 3) |
| **Animations au défilement** (éléments qui glissent, apparaissent, grossissent en scrollant), **curseur animé** ou personnalisé, effets de parallaxe | Contenu statique ; animations courtes seulement pour un changement d'état (section 5) |
| **Effets lumineux** (glow, néon, flou en arrière-plan, « verre dépoli ») | Ombres discrètes (section 5) |
| **Images générées par IA** (illustrations, photos, visages) | **Vraies captures d'écran** de CortexOS, vraies photos du matériel, schémas du projet (diagrammes) |
| Badge ou mention d'un outil de génération (« Made with … ») | Rien |
| Lancer une page **sans favicon** | Favicon = `icone-cyan-fond-nuit.png` (section 2.1), dès le squelette (S4) |

### 9.2 Contenu et chiffres

| Interdit | À la place |
|---|---|
| **Chiffres inventés** : précision, nombre d'utilisateurs, « +500 sessions », compteurs qui défilent | Seulement des **résultats réels**, avec leur **source** (casque, simulation, données publiques — F-32) et leur **date** ; pas de chiffre tant qu'il n'y a pas de mesure |
| **Faux avis, témoignages, logos de partenaires** | Rien, tant qu'il n'y a pas de vrai retour (et alors, avec l'accord écrit de la personne) |
| **Accroche vague** (« Libérez le pouvoir de votre esprit », « L'avenir commence ici ») | Une phrase **concrète** qui dit ce que fait CortexOS : « reconnaît une intention dans un signal EEG et commande un ordinateur, avec des garde-fous de sûreté » |
| **Promesses** : « lit vos pensées », « contrôle total », « 100 % fiable » | Le vocabulaire de la spécification : « reconnaît une intention définie » (F-13), « niveau de confiance », « peut se tromper » |
| **Texte de remplissage** : phrases génériques, listes de « fonctionnalités incroyables », lorem ipsum laissé en place | Chaque texte de l'interface est **précis, vérifié et relu par Eloge**, qui doit pouvoir le défendre en soutenance |
| Présenter une simulation comme un résultat réel | Indicateur de source **toujours visible** (FW-02) |

### 9.3 Vérification avant chaque livraison d'écran

Avant de considérer un écran comme terminé : pas de dégradé · pas d'émoji · aucun chiffre sans source · aucun texte non relu · favicon et titre de page présents · tout fonctionne au clavier · animations coupées avec `prefers-reduced-motion`.
