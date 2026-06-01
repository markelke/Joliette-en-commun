# Journal des modifications CSS — jolietteencommun.org

Chaque entrée résume ce qui a changé et pourquoi, suivi du code exact ajouté ou modifié.

---

## 2026-06-01 — Forum wpForo coupé sur mobile

**Problème :** Sur téléphone, le forum était rogné des deux côtés de l'écran. Les marges intérieures du thème GeneratePress sur `.inside-article` et `.entry-content` réduisaient l'espace disponible pour le forum.

**Solution :** Annuler ces paddings/marges latéraux en dessous de 768 px. Les éléments parents du forum descendent jusqu'à zéro sur les côtés, le forum peut occuper toute la largeur.

```css
/* Ajouté dans @media (max-width: 768px) — section 8 du fichier wp css */
.inside-article,
.entry-content {
  padding-left: 0 !important;
  padding-right: 0 !important;
  margin-left: 0 !important;
  margin-right: 0 !important;
}
```

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

**Problème 2 :** L'étiquette "Sujets récents" (`.wpfcl-5`) était trop collée à son contexte.

**Solution :** Ajouter du padding autour.

```css
.wpfcl-5 {
  padding: 4px 8px !important;
}
```

---

## 2026-06-01 — Titres du forum trop collés

**Problème :** Les liens des titres de forums wpForo ("Le Babillard", "Bâtir des projets ensemble") manquaient d'espace au-dessus et en dessous.

**Solution :** Ajouter du padding vertical sur `.wpforo-forum-title`.

```css
.wpforo-forum-title {
  padding: 6px 0 4px 0 !important;
  margin: 0 0 4px 0 !important;
}
```

---

## 2026-05-xx — Grille d'événements : débordement corrigé

**Problème :** Les cartes d'événements débordaient hors de leur colonne sur certaines tailles d'écran.

**Solution :** Forcer `minmax(0, 1fr)` dans la grille et `min-width: 0` sur les cartes pour que chaque cellule ne dépasse jamais sa colonne.

```css
.jc-events-grid {
  grid-template-columns: repeat(auto-fit, minmax(0, 1fr));
}
.jc-event-card {
  min-width: 0;
}
```

---

## 2026-05-xx — Calendrier complet : masquage mobile désactivé

**Problème :** La règle mobile qui cache les événements après le 4e s'appliquait aussi au calendrier complet, rendant la plupart des événements invisibles sur téléphone.

**Solution :** Surcharger la règle de masquage uniquement dans le conteneur `.jc-calendrier-complet`.

```css
@media (max-width: 1023px) {
  .jc-calendrier-complet .jc-events-grid-layout .jc-timeline .jc-timeline-item:nth-child(n+5) {
    display: flex !important;
  }
}
```

---

## 2026-05-xx — Toggle "Voir plus" : simplification

**Problème :** Le bouton S'inscrire apparaissait dans la version condensée des cartes, et la description débordait.

**Solution :** Limiter la description à 2 lignes avec `line-clamp`, afficher seulement la méta courte (heure + lieu), retirer le bouton S'inscrire de la vue réduite.

```css
.jc-timeline-item:not(.jc-expanded) .jc-desc {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
.jc-timeline-item:not(.jc-expanded) .jc-btn-inscrire {
  display: none !important;
}
```
