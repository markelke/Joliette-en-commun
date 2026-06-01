# Joliette-en-commun
community website
# CSS personnalisé — Joliette en commun

Feuille de style du site **jolietteencommun.org**
(WordPress + GeneratePress · wpForo · The Events Calendar · Forminator).

Le code complet est dans **`style.css`**.

---

## Où va ce code

Copier **tout** le contenu de `style.css` et le coller dans :
**Apparence → Personnaliser → CSS additionnel**, puis cliquer sur **Publier**.

---

## Workflow

1. Modifier `style.css` ici (dans VS Code).
2. Enregistrer, puis « commit » sur GitHub → historique + sauvegarde.
3. Tester sur le site local (LocalWP).
4. Coller la version finale dans le Customizer du site en ligne.

---

## Modifier une partie (sans tout refaire)

| Pour changer… | Aller à la section… | Valeur à modifier |
|---|---|---|
| Couleur de l'en-tête | 1. En-tête | `#2d5a3d` |
| Taille du logo | 1. En-tête | `width` / `height: 72px` |
| Couleur de fond du site | 2. Fond crème | `#faf6f0` |
| Espace sous l'en-tête | 2. Fond crème | `padding-top: 30px` |
| Couleur de survol des boutons | 4. Boutons | `#1e3d2a` |
| Rangée des boutons d'action (calendrier / formulaire) | 4. Boutons | `.jc-events-header` |
| Bouton « Voir plus » mobile | 4. Boutons | `#jc-load-more-btn` |
| Apparence du sondage | 5. Sondage Forminator | `.forminator-poll` |
| Apparence des événements (grille) | 7. Événements | `.jc-timeline…` |
| Hauteur des photos d'événements | 7. Événements | `height: 250px` |
| Nombre d'événements visibles sur mobile | 7. Événements | `:nth-child(n+5)` |
| Seuil d'affichage mobile | 8. Règles mobiles | `768px` |

**Astuce couleur :** pour changer une couleur **partout** d'un seul coup,
utilise Rechercher/Remplacer dans VS Code (Cmd+H) — ex. remplacer toutes
les occurrences de `#2d5a3d` par ta nouvelle couleur.

---

## À savoir (les pièges)

- **Fond crème `#faf6f0` partout.** Ne jamais remettre de vert foncé
  (`#1e3d2a`) sur `body` / `.site-content` : c'est ce qui assombrissait
  tout le site.
- **Page IDs :** accueil = `1807`, communauté = `1855`.
- **Tester sans toucher au site en ligne :** créer une page de test (en
  *Brouillon* ou *Privé*), noter son ID, et préfixer les règles d'essai
  avec `.page-id-XXXX`. Seule cette page sera affectée.
- **En local :** LiteSpeed Cache et Redis ne tournent pas — c'est normal,
  on les ignore.
- **La section 7 (Événements) dépend du script Google Sheets.** Le CSS
  habille le HTML que le script génère — les deux repos travaillent ensemble.
  Voir le repo `jec-apps-script` pour la logique d'alimentation.

---

## Sections de `style.css`

1. En-tête (vert forêt, logo, titre, slogan)
2. Fond crème global
3. Mise en page (colonnes forum + sondage)
4. Boutons (calendrier, retour, header événements, voir plus mobile)
5. Sondage Forminator
6. Éléments masqués
7. Événements (grille 4 colonnes — alimentée par le script Google Sheets)
8. Règles mobiles