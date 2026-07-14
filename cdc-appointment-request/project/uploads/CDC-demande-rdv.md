# Cahier des charges — Site "Demande RDV"
### Mini-site interactif de demande de rendez-vous

**Version :** 1.0
**Stack technique :** Next.js (App Router)
**Type de projet :** Site vitrine à page unique, à effet "surprise", partageable via lien privé

---

## 1. Contexte et objectif

### 1.1 Pitch
Un mini-site à parcours unique, dans l'esprit d'une carte interactive : l'utilisateur qui reçoit le lien répond à une suite de questions ludiques qui aboutissent à une demande de rendez-vous. Le ton est léger, romantique et un peu taquin (le bouton "non" fuit le curseur).

### 1.2 Objectif fonctionnel
- Faire vivre un parcours en 5 étapes menant à la confirmation d'un rendez-vous (date, heure, repas).
- Provoquer un effet de surprise et d'humour plutôt que de servir un formulaire classique.
- Le site est pensé pour être envoyé à **une seule personne** via un lien unique — pas de recherche, pas de SEO à optimiser, pas de compte utilisateur.

### 1.3 Hors périmètre (v1)
- Pas de back-office / CMS.
- Pas de base de données obligatoire (voir §7 pour l'option de persistance).
- Pas de multi-langue (site en français uniquement).
- Pas d'authentification.

---

## 2. Stack technique

| Élément | Choix | Justification |
|---|---|---|
| Framework | **Next.js 14+ (App Router)** | Rendu rapide, routing simple, déploiement facile (Vercel) |
| Langage | TypeScript | Sécurité de typage sur l'état du parcours |
| Style | CSS Modules **ou** Tailwind CSS | Au choix ; ce CDC utilise des noms de classes génériques compatibles avec les deux |
| Police | Google Fonts (`next/font/google`) — `Cormorant Garamond` + `Poppins` | Cohérence typographique élégante/romantique |
| Animations | CSS natif + `framer-motion` (optionnel, pour les transitions d'étapes) | Transitions douces entre les écrans |
| État du parcours | `useState` / `useReducer` en local (Client Component) | Pas besoin de state management global pour un flux linéaire |
| Déploiement | Vercel (recommandé) | Compatible nativement avec Next.js, déploiement en un clic |
| Persistance (optionnelle) | Voir §7 | Pour enregistrer la réponse (date/heure/repas choisis) quelque part |

### 2.1 Arborescence proposée

```
demande-rdv/
├── app/
│   ├── layout.tsx              # Layout racine, polices, meta
│   ├── page.tsx                # Page unique, orchestre les étapes
│   ├── globals.css
│   └── api/
│       └── reponse/
│           └── route.ts        # (optionnel) endpoint pour enregistrer la réponse
├── components/
│   ├── Etape1Question.tsx
│   ├── Etape2Confirmation.tsx
│   ├── Etape3DateHeure.tsx
│   ├── Etape4Repas.tsx
│   ├── Etape5Final.tsx
│   ├── BoutonNonFuyard.tsx
│   ├── DecorFlottant.tsx        # cœurs/fleurs animés en fond
│   └── CarteConteneur.tsx       # wrapper visuel commun (carte blanche centrée)
├── lib/
│   └── types.ts                # Types partagés (RepasChoisi, ReponseParcours, etc.)
├── public/
│   └── (assets optionnels : photo, favicon)
├── next.config.js
├── tsconfig.json
└── package.json
```

---

## 3. Parcours utilisateur (user flow)

Le site est **une seule page** (`/`) qui affiche successivement 5 écrans ("étapes"), sans rechargement de page. La navigation se fait par état local (`etapeActuelle: number`), pas par routes Next.js différentes — cela reste dans l'esprit "carte" plutôt que "site à pages".

```
[Étape 1: Question]
   "Tu veux bien sortir avec moi ?"
   Bouton OUI -----------------------> [Étape 2]
   Bouton NON (fuit le curseur/doigt, ne peut jamais être cliqué)

[Étape 2: Confirmation]
   "Attends, tu as vraiment dit oui ??"
   Bouton "ok ok →" ------------------> [Étape 3]

[Étape 3: Date & heure]
   Champ date (input date)
   Champ heure (select)
   Bouton "suivant →" ----------------> [Étape 4]
   Bouton "← retour" ------------------> [Étape 2]

[Étape 4: Choix du repas]
   Grille de 6 choix (Pizza / Sushi / Burgers / Pâtes / Tacos / Ramen)
   Sélection visuelle (un seul choix actif à la fois)
   Bouton "c'est parti →" -------------> [Étape 5]
   Bouton "← retour" ------------------> [Étape 3]

[Étape 5: Message final]
   Récapitulatif (date, heure, repas choisis)
   Message romantique + P.S. humoristique
   (optionnel) Bouton pour envoyer la réponse au propriétaire du site
```

### 3.1 Règles de navigation
- Aucune étape ne peut être sautée : on avance uniquement via les boutons d'action.
- Le retour arrière est possible depuis l'étape 3 et 4 uniquement (pas de retour depuis l'étape 5, qui est un état final).
- Le bouton "non" de l'étape 1 **n'a jamais d'état cliqué avec succès** — c'est un running gag assumé, pas un bug.

---

## 4. Spécification détaillée par écran

### Étape 1 — Question
| Élément | Détail |
|---|---|
| Emoji/illustration | Emoji ou icône ronde en tête de carte |
| Titre | `Tu veux bien sortir avec moi ? 🌸` (H1, police display) |
| Sous-texte | Phrase courte en italique, ton léger |
| Bouton OUI | Style plein, couleur d'accent, mène directement à l'étape 2 |
| Bouton NON | **Comportement spécial** : au survol (desktop) ou au tap (mobile), le bouton se déplace à une position aléatoire dans le viewport. Il ne doit jamais pouvoir être cliqué normalement. |

**Comportement technique du bouton NON :**
- Desktop : écouteur `mouseenter` → recalcule une position aléatoire (`top`/`left` en `position: fixed`) dans les limites de la fenêtre, avec une marge de sécurité (éviter les bords).
- Mobile : écouteur `touchstart` → même comportement (le survol n'existant pas au tactile).
- Le clic lui-même déclenche aussi un repositionnement, par sécurité si l'utilisateur arrive à cliquer pile au moment du déplacement.
- Accessibilité : prévoir un mode alternatif si `prefers-reduced-motion` est activé (voir §8) — le bouton peut rester fixe mais désactivé, avec un `aria-label` explicite ("Ce bouton n'est pas disponible").

### Étape 2 — Confirmation
| Élément | Détail |
|---|---|
| Titre | `Attends, tu as vraiment dit oui ?? 😭` |
| Sous-texte | Phrase humoristique |
| Bouton | `ok ok →` → mène à l'étape 3 |

### Étape 3 — Date & heure
| Élément | Détail |
|---|---|
| Champ 1 | `<input type="date">` — label "Choisis un jour" |
| Champ 2 | `<select>` — label "À quelle heure ?" avec options prédéfinies (ex : déjeuner, café, dîner, soirée) |
| Validation | Le bouton "suivant" reste utilisable même si les champs sont vides (valeurs par défaut affichées à l'étape 5 : "une date à définir" / "une heure à définir") — cohérent avec le ton non contraignant du site |
| Bouton retour | Renvoie à l'étape 2 |

### Étape 4 — Choix du repas
| Élément | Détail |
|---|---|
| Titre | `On a envie de quoi ? 🍽️✨` |
| Grille | 6 cartes cliquables (icône + libellé), disposées en 3 colonnes desktop / 2 colonnes mobile |
| Sélection | Un seul choix actif à la fois, retour visuel (bordure + fond + léger `translateY`) |
| Bouton suivant | `c'est parti →` → étape 5. Reste actif même sans sélection (valeur par défaut : "une surprise") |
| Bouton retour | Renvoie à l'étape 3 |

### Étape 5 — Message final
| Élément | Détail |
|---|---|
| Récapitulatif | Affiche jour / heure / repas choisis, injectés dynamiquement depuis l'état du parcours |
| Message principal | Texte romantique fixe, personnalisable |
| P.S. | Phrase de clôture humoristique |
| Pas de retour | Écran terminal du parcours |

---

## 5. Modèle de données (état du parcours)

```ts
// lib/types.ts

export type Repas = 'Pizza' | 'Sushi' | 'Burgers' | 'Pâtes' | 'Tacos' | 'Ramen';

export interface ReponseParcours {
  etapeActuelle: 1 | 2 | 3 | 4 | 5;
  date: string | null;       // format ISO (yyyy-mm-dd) issu de l'input date
  heure: string | null;      // libellé sélectionné (ex: "6:00 PM — dîner")
  repas: Repas | null;
}
```

L'état est géré en local via `useState<ReponseParcours>` dans `page.tsx`, ou via un `useReducer` si le nombre d'actions grandit (recommandé si on ajoute des étapes plus tard).

---

## 6. Design system

### 6.1 Palette de couleurs
| Nom | Hex | Usage |
|---|---|---|
| Rose fond clair | `#ffe1ea` | Fond dégradé (haut) |
| Rose fond soutenu | `#ffd0de` | Fond dégradé (bas) |
| Rose fort (accent) | `#e8447c` | Boutons principaux, titres soulignés |
| Rose doux | `#f6a8c2` | Éléments secondaires |
| Violet | `#c98bd0` | Bouton "non" |
| Texte principal | `#3a2530` | Corps de texte |
| Carte (surface) | `#fffafc` | Fond de la carte centrale |

### 6.2 Typographie
- **Titres (`h1`, `h2`)** : `Cormorant Garamond`, semi-bold, serif — pour le ton élégant/romantique.
- **Corps de texte, boutons, labels** : `Poppins`, regular/medium — lisibilité, ton actuel.
- Échelle : titres `1.7rem–2rem` (mobile-first, augmenter légèrement sur desktop si besoin).

### 6.3 Composants visuels communs
- **Carte centrale** : fond blanc cassé, `border-radius` généreux (~26px), ombre douce colorée (`box-shadow` teinté rose), largeur max ~440px, centrée verticalement et horizontalement.
- **Décor flottant** : emojis (🌸 💗 🌷 ✨) positionnés aléatoirement en fond, animation de flottement lente (`translateY` + `rotate` en boucle), `pointer-events: none` pour ne jamais gêner les clics.
- **Boutons** : forme pilule (`border-radius: 30px`), pas de bordure, transition légère au clic (`scale(0.96)`).

### 6.4 Responsive
- Mobile-first. Point de rupture principal autour de `480px`.
- Sur mobile, la grille de repas passe de 3 à 2 colonnes si nécessaire selon les tests de lisibilité.
- Le bouton "non" doit rester dans les limites du viewport sur toutes les tailles d'écran (calcul de position aléatoire borné par `window.innerWidth`/`innerHeight`).

---

## 7. Option de persistance (facultatif, v1.1)

Si on veut que la personne qui répond envoie réellement sa réponse (date/heure/repas) au créateur du site plutôt que de rester uniquement côté client :

**Option simple — API Route Next.js + email**
- `app/api/reponse/route.ts` : reçoit un `POST` avec `{ date, heure, repas }`.
- Envoie un email (via [Resend](https://resend.com) ou équivalent) ou pousse un message (webhook Discord/Telegram) au créateur du site.
- Pas de base de données nécessaire pour un usage ponctuel.

**Option avec stockage — base de données légère**
- Utiliser une base comme Supabase, PlanetScale ou simplement un fichier JSON si hébergement simple.
- Utile seulement si le site est réutilisé pour plusieurs personnes/plusieurs occasions.

*Décision à prendre selon le besoin réel : pour un usage "one-shot" (une seule personne, un seul envoi), l'option email/webhook suffit largement et évite toute infra superflue.*

---

## 8. Accessibilité et qualité

- **Contraste** : vérifier le contraste texte/fond (rose foncé sur fond clair) selon WCAG AA.
- **Focus clavier** : tous les boutons et champs doivent avoir un état de focus visible (`outline` personnalisé si le style par défaut est supprimé).
- **`prefers-reduced-motion`** : désactiver les animations de flottement et le comportement fuyant du bouton "non" si l'utilisateur a activé la réduction de mouvement dans son OS — proposer une alternative statique et utilisable.
- **Lecteurs d'écran** : `aria-label` explicites sur les boutons emoji-only, et sur le bouton "non" pour signaler son comportement particulier.
- **Formulaires** : `label` associés correctement aux champs date/heure (via `htmlFor`/`id`).

---

## 9. SEO et métadonnées

Le site étant destiné à un envoi de lien privé (pas de découverte publique), les métadonnées restent minimales :

```ts
// app/layout.tsx
export const metadata = {
  title: "Demande de rendez-vous",
  description: "Une petite surprise pour toi.",
  robots: "noindex, nofollow", // pour ne pas être indexé publiquement
};
```

---

## 10. Déploiement

1. Repo Git (GitHub/GitLab).
2. Connexion du repo à **Vercel**.
3. Déploiement automatique sur chaque push (branche `main` → production).
4. Domaine : nom de domaine personnalisé optionnel, ou sous-domaine `.vercel.app` suffisant pour un usage privé.
5. Variables d'environnement (si option email/webhook du §7 activée) : clé API du service d'envoi, stockée dans les paramètres Vercel (jamais en dur dans le code).

---

## 11. Critères d'acceptation (checklist de recette)

- [ ] Les 5 étapes s'enchaînent sans rechargement de page.
- [ ] Le bouton "non" ne peut jamais être cliqué avec succès, sur desktop et mobile.
- [ ] Les champs date/heure/repas ne sont pas obligatoires pour avancer, mais leurs valeurs sont bien reprises dans le récapitulatif final si elles sont renseignées.
- [ ] Le récapitulatif final affiche les bonnes valeurs choisies (ou les valeurs par défaut sinon).
- [ ] Le site est utilisable et lisible sur mobile (test réel sur un écran ≤ 390px de large).
- [ ] Les animations sont désactivées ou adaptées si `prefers-reduced-motion` est actif.
- [ ] Le site n'est pas indexé par les moteurs de recherche (`robots: noindex`).
- [ ] Déploiement fonctionnel sur Vercel avec URL partageable.

---

## 12. Évolutions possibles (hors v1)

- Ajouter une vraie photo du couple/de la personne en haut de la carte.
- Ajouter un fond musical (lecture automatique au clic, jamais autoplay bloquant).
- Générer une capture d'écran ou un PDF récapitulatif du "oui" et des choix, envoyé automatiquement par email.
- Rendre le contenu (texte, prénom, date de référence) configurable via variables d'environnement pour réutiliser le site avec un autre message sans toucher au code.
