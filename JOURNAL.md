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

**Problème :** La description des cartes d'événements débordait, et deux versions de la méta (courte et complète) devaient s'alterner selon que la carte est ouverte ou fermée.

**Solution :** Une checkbox cachée (`.jc-toggle-input`) sert d'interrupteur CSS pur. Quand elle est cochée, le sélecteur `~ ` active les états "étendu". La description est limitée à 2 lignes par défaut via `line-clamp`, la méta complète est cachée par défaut, et le label affiche "Voir plus →" ou "Voir moins ↑" automatiquement.

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
  -webkit-line-clamp: 2 !important;       /* ← ajuster pour plus/moins de lignes */
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
  font-size: 0.85rem !important;  /* ← taille du lien Voir plus */
  margin-top: 4px !important;
}
.jc-toggle-label::before { content: "Voir plus \2192"; }
.jc-toggle-input:checked ~ .jc-toggle-label::before { content: "Voir moins \2191"; }
```
