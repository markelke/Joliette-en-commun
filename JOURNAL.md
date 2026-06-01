# Journal des modifications CSS — jolietteencommun.org

Chaque entrée : problème → solution → code exact → **comment ajuster**.

---

## 2026-06-01 — Forum : titres longs coupés

**Problème :** Les titres de forums comme "Le Babillard (Annonces & Entraide)" étaient tronqués. La colonne info (`.wpforo-forum-info`) est dans un flex row et sans `min-width: 0`, elle ne rétrécit pas correctement, forçant le titre à déborder ou se couper.

**Solution :** Donner à la colonne info `min-width: 0` pour qu'elle puisse rétrécir, et autoriser le retour à la ligne sur le titre et son lien.

```css
.wpforo-forum-info {
  min-width: 0 !important;
  flex: 1 !important;
}
.wpforo-forum-title {
  white-space: normal !important;
  overflow: visible !important;
  word-break: break-word !important;
}
.wpforo-forum-title a {
  white-space: normal !important;
  overflow: visible !important;
  text-overflow: unset !important;
}
```

**Pour ajuster :**
- Remettre `white-space: nowrap` sur `.wpforo-forum-title` pour forcer le titre sur une seule ligne (avec troncature).
- `word-break: break-word` → `word-break: normal` si on veut que les mots longs ne se coupent pas au milieu d'un mot.

---

## 2026-06-01 — Forum wpForo coupé sur mobile

**Problème :** Sur téléphone, le forum était rogné des deux côtés de l'écran. Les marges intérieures du thème GeneratePress sur `.inside-article` et `.entry-content` réduisaient l'espace disponible.

**Solution :** Annuler les paddings/marges latéraux de ces deux conteneurs en dessous de 768 px.

```css
/* Dans @media (max-width: 768px) */
.inside-article,
.entry-content {
  padding-left: 0 !important;
  padding-right: 0 !important;
  margin-left: 0 !important;
  margin-right: 0 !important;
}
```

**Pour ajuster :**
- Remplacer `0` par ex. `12px` pour laisser un petit espace latéral sur mobile.
- Changer `768px` dans `@media (max-width: 768px)` pour cibler une largeur d'écran différente (ex. `480px` pour petits téléphones seulement).

---

## 2026-06-01 — Forum : espace vide &nbsp; et padding "Sujets récents"

**Problème 1 :** wpForo injecte `<li>&nbsp;</li>` dans la description de certains forums, créant un espace blanc inutile.

**Solution :** Masquer tous les `<li>` dans `.wpforo-forum-description` et supprimer les marges de la liste.

```css
.wpforo-forum-description ul {
  margin: 0 !important;
  padding: 0 !important;
  list-style: none !important;
}
.wpforo-forum-description li {
  display: none !important;
}
```

**Pour ajuster :**
- Remettre `display: list-item` sur `.wpforo-forum-description li` si on veut afficher de vraies descriptions de forums plus tard.

**Problème 2 :** L'étiquette "Sujets récents" (`.wpfcl-5`) était trop collée à son contexte.

**Solution :** Ajouter du padding autour.

```css
.wpfcl-5 {
  padding: 4px 8px !important;
}
```

**Pour ajuster :**
- `4px 8px` = haut/bas gauche/droite. Augmenter le premier chiffre pour plus d'espace vertical, le deuxième pour plus d'espace horizontal.

---

## 2026-06-01 — Titres du forum trop collés

**Problème :** Les titres de forums wpForo manquaient d'espace au-dessus et en dessous.

**Solution :** Ajouter du padding vertical sur `.wpforo-forum-title`.

```css
.wpforo-forum-title {
  padding: 5px 5px 4px 5px !important;  /* haut droite bas gauche */
  margin: 0 0 4px 0 !important;
}
```

**Pour ajuster :**
- Les 4 valeurs de `padding` : `haut droite bas gauche`. Ex. `8px 5px 6px 5px` pour plus d'espace vertical.
- `margin: 0 0 4px 0` : le `4px` est l'espace sous le titre avant la description. Augmenter pour aérer davantage.

---

## 2026-05-xx — Grille d'événements : débordement corrigé

**Problème :** Les cartes d'événements débordaient hors de leur colonne sur certaines tailles d'écran.

**Solution :** Forcer `minmax(0, 1fr)` dans la grille et `min-width: 0` sur les cartes.

```css
.jc-events-grid {
  grid-template-columns: repeat(auto-fit, minmax(0, 1fr));
}
.jc-event-card {
  min-width: 0;
}
```

**Pour ajuster :**
- Remplacer `minmax(0, 1fr)` par ex. `minmax(280px, 1fr)` pour forcer une largeur minimale par carte avant qu'elles passent à la ligne.

---

## 2026-05-xx — Calendrier complet : masquage mobile désactivé

**Problème :** La règle mobile qui cache les événements après le 4e s'appliquait aussi au calendrier complet.

**Solution :** Surcharger la règle de masquage uniquement dans le conteneur `.jc-calendrier-complet`.

```css
@media (max-width: 1023px) {
  .jc-calendrier-complet .jc-events-grid-layout .jc-timeline .jc-timeline-item:nth-child(n+5) {
    display: flex !important;
  }
}
```

**Pour ajuster :**
- `1023px` : seuil en dessous duquel la règle s'applique. Changer pour cibler une autre taille d'écran.
- `n+5` : signifie "à partir du 5e élément". Changer en `n+4` pour montrer 3 éléments, `n+6` pour en montrer 5.

---

## 2026-05-xx — Toggle "Voir plus" : simplification

**Problème :** La description des cartes d'événements débordait, et deux versions de la méta (courte et complète) devaient s'alterner selon que la carte est ouverte ou fermée.

**Solution :** Une checkbox cachée (`.jc-toggle-input`) sert d'interrupteur CSS pur. Quand elle est cochée, le sélecteur `~` active les états "étendu".

```css
/* Checkbox invisible — sert d'interrupteur */
.jc-toggle-input {
  position: absolute !important;
  opacity: 0 !important;
  pointer-events: none !important;
}

/* Description : 2 lignes par défaut, complète quand étendu */
.jc-timeline-desc {
  display: -webkit-box !important;
  -webkit-line-clamp: 2 !important;
  -webkit-box-orient: vertical !important;
  overflow: hidden !important;
  margin: 4px 0 8px 0 !important;
  font-size: 0.9rem !important;
  color: #333 !important;
}
.jc-toggle-input:checked ~ .jc-timeline-desc {
  -webkit-line-clamp: unset !important;
  overflow: visible !important;
}

/* Méta courte : visible par défaut, cachée quand étendu */
.jc-meta-court {
  font-size: 0.85rem !important;
  color: #2d5a3d !important;
  margin-bottom: 8px !important;
  white-space: nowrap !important;
  overflow: hidden !important;
  text-overflow: ellipsis !important;
}
.jc-toggle-input:checked ~ .jc-meta-court { display: none !important; }

/* Méta complète : cachée par défaut, visible quand étendu */
.jc-meta-complet { display: none !important; }
.jc-toggle-input:checked ~ .jc-meta-complet {
  display: block !important;
  font-size: 0.85rem !important;
  color: #2d5a3d !important;
  line-height: 1.6 !important;
  margin-bottom: 8px !important;
}

/* Label Voir plus / Voir moins — texte généré en CSS */
.jc-toggle-label {
  display: inline-block !important;
  cursor: pointer !important;
  color: #2d5a3d !important;
  font-weight: 600 !important;
  font-size: 0.85rem !important;
  margin-top: 4px !important;
}
.jc-toggle-label::before { content: "Voir plus \2192"; }
.jc-toggle-input:checked ~ .jc-toggle-label::before { content: "Voir moins \2191"; }
```

**Pour ajuster :**
- `-webkit-line-clamp: 2` → changer `2` pour afficher plus ou moins de lignes dans la vue réduite.
- `font-size: 0.9rem` sur `.jc-timeline-desc` → taille du texte de description.
- `font-size: 0.85rem` sur `.jc-meta-court` et `.jc-meta-complet` → taille du texte de méta (heure, lieu, coût).
- `color: #2d5a3d` sur `.jc-toggle-label` → couleur du lien "Voir plus". Remplacer par n'importe quelle couleur hex.
- `margin-top: 4px` sur `.jc-toggle-label` → espace au-dessus du lien "Voir plus".
- Texte "Voir plus →" et "Voir moins ↑" : modifier directement le `content:` des deux dernières lignes.
