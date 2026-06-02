# Journal des modifications CSS — jolietteencommun.org

Chaque entrée : problème → solution → code exact → **comment ajuster**.

---

## 2026-06-02 — Bouton "Ajouter au calendrier" sur chaque carte

**Problème :** Les visiteurs ne pouvaient pas ajouter un événement directement à leur calendrier (Google, Apple, Outlook, etc.) depuis la page.

**Solution :** Utilisation du web component `add-to-calendar-button` (chargé via CDN). Chaque carte reçoit un bouton dans un nouveau pied de carte `.jc-card-footer` qui aligne le lien "Voir plus" à gauche et le bouton calendrier à droite.

```js
// Script chargé une fois dans le premier bloc <!-- wp:html --> des 3 fonctions :
'<script src="https://cdn.jsdelivr.net/npm/add-to-calendar-button" async defer></script>'

// Nouvelle fonction — construit le web component
function genererBoutonCalendrier(ev) {
  let startDate = formaterDateISO(ev.dateDebut);
  if (!startDate) return '';
  // ... attributs name, startDate, endDate, startTime, endTime, timeZone, location, description
  html += " options=\"'Google','Apple','Outlook.com','Microsoft365','Yahoo','iCal'\"";
  html += ' label="Ajouter au calendrier" buttonStyle="text" size="3" lightMode="bodyScheme"';
}

// Formate une date en yyyy-MM-dd
function formaterDateISO(valeur) {
  let d = valeur instanceof Date ? valeur : new Date(valeur);
  return Utilities.formatDate(d, Session.getScriptTimeZone(), 'yyyy-MM-dd');
}

// Formate une heure en HH:mm (gère Date et chaînes "14h30")
function formaterHeureISO(valeur) {
  if (valeur instanceof Date) return Utilities.formatDate(valeur, Session.getScriptTimeZone(), 'HH:mm');
  let h = valeur.toString().trim().replace('h', ':');
  // → normalise en "14:30"
}

// Dans genererBlocHTML — remplace le <label> seul par un pied de carte
html += '      <div class="jc-card-footer">\n';
html += '        <label for="jc-toggle-' + idUnique + '" class="jc-toggle-label"></label>\n';
html += '        ' + genererBoutonCalendrier({...}) + '\n';
html += '      </div>\n';
```

```css
/* Pied de carte : Voir plus à gauche, bouton calendrier à droite */
.jc-card-footer {
  display: flex !important;
  justify-content: space-between !important;   /* ← espace entre les deux éléments */
  align-items: center !important;
  gap: 8px !important;                         /* ← espace minimum entre eux */
  margin-top: auto !important;                 /* ← colle au bas de la carte */
  padding-top: 8px !important;                 /* ← espace au-dessus du pied */
}

/* Sélecteur mis à jour : le label est maintenant imbriqué dans .jc-card-footer */
.jc-toggle-input:checked ~ .jc-card-footer .jc-toggle-label::before { content: "Voir moins \2191"; }
```

**Pour ajuster :**
- `label="Ajouter au calendrier"` dans `genererBoutonCalendrier` → texte du bouton.
- `size="3"` → taille du bouton (1 = petit, 5 = grand).
- `buttonStyle="text"` → style du bouton. Autres options : `"round"`, `"flat"`, `"neumorphism"`.
- `options=` → liste des calendriers proposés. Retirer une option pour la supprimer du menu.
- `padding-top: 8px` sur `.jc-card-footer` → espace entre le contenu de la carte et le pied.
- `gap: 8px` → espace entre "Voir plus" et le bouton calendrier si ils se touchent.

---

## 2026-06-02 — Page accueil : forum + sondage en haut, événements en dessous

**Problème :** La grille d'événements s'affichait avant le forum et le sondage. On voulait l'inverse.

**Solution :** Séparation du HTML en deux variables distinctes dans `synchroniserTimelineCitoyenne`, puis assemblage dans le bon ordre.

```js
let htmlForum = '...'; // bloc forum + sondage
let htmlEvenements = '...'; // boutons + grille + script
let htmlContenu = htmlForum + htmlEvenements; // ← forum d'abord
```

**Pour ajuster :**
- Inverser l'ordre → `let htmlContenu = htmlEvenements + htmlForum`.

---

## 2026-06-02 — Liste expositions : 2 colonnes sur ordinateur

**Problème :** Avec plusieurs expositions, la liste prenait trop de hauteur sur grand écran.

**Solution :** CSS `column-count: 2` sur `.jc-expo-liste` à partir de 768 px. `break-inside: avoid` sur chaque item pour qu'aucune vignette ne soit coupée entre deux colonnes.

```css
@media (min-width: 768px) {
  .jc-expo-liste {
    column-count: 2 !important;     /* ← nombre de colonnes */
    column-gap: 40px !important;    /* ← espace entre les colonnes */
  }
}
.jc-expo-item {
  break-inside: avoid !important;
  -webkit-column-break-inside: avoid !important;
}
```

**Pour ajuster :**
- `column-count: 2` → changer en `3` pour 3 colonnes sur très grand écran.
- `column-gap: 40px` → espace entre les colonnes. Réduire si les items sont trop serrés.
- `min-width: 768px` → seuil d'activation. Augmenter pour activer seulement sur plus grand écran.

---

## 2026-06-01 — Calendrier complet : expositions en bloc compact

**Problème :** Les expositions s'entassaient dans le groupement par mois avec tous les autres événements, prenant trop de place et noyant les activités régulières.

**Solution (JS — synchroniserCalendrierComplet) :** Après le tri, les événements sont séparés en deux groupes via `estExposition()`. Les expositions apparaissent d'abord dans un bloc compact, puis les autres événements sont groupés par mois normalement.

```js
let expositions = listeEvenements.filter(function(ev) { return estExposition(ev.titre); });
let autres = listeEvenements.filter(function(ev) { return !estExposition(ev.titre); });

if (expositions.length > 0) {
  html += genererListeExpositionsCompacte(expositions, 'https://jolietteencommun.org/expositions/');
}
if (autres.length > 0) {
  // groupement par mois sur `autres` seulement
}
```

**Nouvelle fonction `genererListeExpositionsCompacte(expositions, url)` :**

```js
function genererListeExpositionsCompacte(expositions, url) {
  let html = '<div class="jc-expo-compacte">\n';
  html += '<h2 class="jc-expo-titre">🎨 Expositions en cours</h2>\n';
  html += '<ul class="jc-expo-liste">\n';
  expositions.forEach(function(ev) {
    let lien = (ev.lienBouton && ev.lienBouton.toString().startsWith('http'))
      ? '<a href="' + ev.lienBouton + '" target="_blank" rel="noopener">' + ev.titre + '</a>'
      : ev.titre;
    let dateFin = ev.dateFin ? formaterDateFr(ev.dateFin) : '';
    html += '  <li class="jc-expo-item">🎨 ' + lien;
    if (dateFin) html += ' <span class="jc-expo-date">jusqu\'au ' + dateFin + '</span>';
    html += '</li>\n';
  });
  html += '</ul>\n';
  html += '<a href="' + url + '" class="jc-expo-btn">Voir toutes les expositions →</a>\n';
  html += '</div>\n';
  return html;
}
```

**Solution (CSS — section 7C) :**

```css
.jc-expo-compacte {
  background-color: #d6e6d6 !important;     /* ← fond sauge clair */
  border: 1px solid #c8dfc8 !important;
  border-left: 4px solid #2d5a3d !important; /* ← barre verte à gauche */
  border-radius: 12px !important;
  padding: 20px 24px !important;             /* ← espace intérieur */
  margin-bottom: 36px !important;            /* ← espace sous le bloc */
}
.jc-expo-titre {
  font-size: 1.2rem !important;
  font-weight: 800 !important;
  color: #2d5a3d !important;
  margin: 0 0 16px 0 !important;
}
.jc-expo-liste {
  list-style: none !important;
  padding: 0 !important;
  margin: 0 0 16px 0 !important;
}
.jc-expo-item {
  padding: 10px 0 !important;
  border-bottom: 1px solid #c8dfc8 !important;
  font-size: 0.95rem !important;
  color: #1e3d2a !important;
}
.jc-expo-item:last-child {
  border-bottom: none !important;
}
.jc-expo-date {
  color: #c8963e !important;         /* ← bronze pour la date de fin */
  font-size: 0.85rem !important;
  font-weight: 500 !important;
  margin-left: 6px !important;
}
```

**Mise à jour — vignette image par exposition :**

Chaque `<li>` contient maintenant une vignette (image ou repli emoji) + un bloc info :

```js
let vignette = (ev.imageUrL && ev.imageUrL.toString().startsWith('http'))
  ? '<img src="' + ev.imageUrL + '" class="jc-expo-vignette" alt="' + ev.titre + '">'
  : '<span class="jc-expo-vignette-vide">🎨</span>';
// → <li class="jc-expo-item">[vignette]<div class="jc-expo-info">[titre][date]</div></li>
```

```css
.jc-expo-item {
  display: flex !important;
  align-items: center !important;
  gap: 12px !important;              /* ← espace entre vignette et texte */
}
.jc-expo-vignette {
  width: 60px !important;            /* ← taille de la vignette */
  height: 60px !important;
  object-fit: cover !important;
  border-radius: 8px !important;
  flex-shrink: 0 !important;
}
.jc-expo-vignette-vide {
  width: 60px !important;
  height: 60px !important;
  background-color: #c8dfc8 !important; /* ← fond sauge si pas d'image */
  border-radius: 8px !important;
  display: flex !important;
  align-items: center !important;
  justify-content: center !important;
  font-size: 1.4rem !important;         /* ← taille de l'emoji repli */
}
.jc-expo-info {
  display: flex !important;
  flex-direction: column !important;
  gap: 4px !important;               /* ← espace entre titre et date */
}
```

**Pour ajuster :**
- `background-color: #eaf4ea` sur `.jc-expo-compacte` → couleur de fond du bloc.
- `border-left: 4px solid #2d5a3d` → épaisseur et couleur de la barre verticale gauche.
- `margin-bottom: 36px` → espace entre le bloc expositions et le premier groupe de mois.
- `width/height: 60px` sur `.jc-expo-vignette` et `.jc-expo-vignette-vide` → taille des vignettes. Changer les deux en même temps.
- `gap: 12px` sur `.jc-expo-item` → espace entre la vignette et le texte.
- `gap: 4px` sur `.jc-expo-info` → espace entre le titre et la date "jusqu'au X".
- `color: #c8963e` sur `.jc-expo-date` → couleur de la date. Bronze par défaut.
- Le lien "Voir toutes les expositions →" utilise `.jc-expo-btn`.
- L'URL du lien : `'https://jolietteencommun.org/expositions/'` dans `synchroniserCalendrierComplet()`.

---

## 2026-06-01 — Sondage Forminator : deux messages superposés

**Problème :** Le message original anglais "You have already voted for this poll." et le remplacement CSS français apparaissaient en même temps. Le `font-size: 0` seul ne suffisait pas — le CSS de Forminator surchargeait la règle avec une spécificité plus élevée.

**Solution :** Augmenter la spécificité du sélecteur en ajoutant `.forminator-response-message` dans la chaîne (3 classes au lieu de 2), et ajouter `line-height: 0` pour vraiment écraser la hauteur de ligne du texte original.

```css
.forminator-poll .forminator-response-message .forminator-label--forminator-error {
  font-size: 0 !important;
  line-height: 0 !important;       /* ← clé : réduit aussi la hauteur de ligne à 0 */
  color: transparent !important;
  display: block !important;
}
.forminator-poll .forminator-response-message .forminator-label--forminator-error::before {
  content: "Vous avez déjà enregistré votre vote. Merci !" !important;
  font-size: 0.95rem !important;
  line-height: 1.5 !important;     /* ← rétablit une hauteur de ligne normale */
  color: #2d5a3d !important;
  font-weight: 500 !important;
  display: block !important;
}
```

**Pour ajuster :**
- Changer le texte entre les guillemets du `content:` pour modifier le message affiché.
- `font-size: 0.95rem` sur `::before` → taille du message de remplacement.
- `line-height: 1.5` sur `::before` → interligne du message. Augmenter pour plus d'espace.
- `color: #2d5a3d` → couleur du texte (vert forêt). Remplacer par n'importe quelle couleur hex.

---

## 2026-06-01 — Forum : supprimer les icônes avant les titres de sujets

**Problème :** Chaque sujet récent affichait une icône Font Awesome avant son titre (punaise pour épinglé, feuille pour répondu/non-répondu). Ces icônes ajoutaient du bruit visuel inutile.

**Solution :** Masquer tous les `<i>` à l'intérieur de `.wpforo-last-topic-title`.

```css
.wpforo-last-topic-title i {
  display: none !important;
}
```

**Pour ajuster :**
- Pour remettre une icône spécifique visible, remplacer `display: none` par `display: inline` et cibler sa classe précise. Ex : `.wpforo-last-topic-title i.fa-thumbtack { display: inline !important; }` pour garder seulement la punaise "épinglé".

---

## 2026-06-01 — Forum : titre du dernier sujet coupé en cours de ligne

**Problème :** Le titre du dernier sujet (`.wpforo-last-topic-title`) se coupait aux deux tiers de la ligne sans passer à la suivante. Le div est dans un flex row sans `min-width: 0`, donc il ne rétrécit pas et le texte déborde ou se tronque.

**Solution :** Forcer le retour à la ligne sur le conteneur, le lien et le pseudo-élément `::after` qui affiche le vrai titre.

```css
.wpforo-last-topic-title {
  white-space: normal !important;
  overflow: visible !important;
  min-width: 0 !important;
  flex: 1 !important;
  word-break: break-word !important;
}
.wpforo-last-topic-title a {
  font-size: 0 !important;
  white-space: normal !important;
  line-height: 1.4 !important;
  display: block !important;
}
.wpforo-last-topic-title a::after {
  content: attr(title);
  font-size: 0.95rem !important;
  font-weight: inherit !important;
  white-space: normal !important;
  display: inline !important;
}
```

**Pour ajuster :**
- `line-height: 1.4` sur le lien → espacement entre les lignes quand le titre passe à la ligne. Augmenter pour plus d'air (ex. `1.6`).
- `font-size: 0.95rem` sur `::after` → taille du texte du titre. Augmenter ou diminuer selon le rendu.
- `word-break: break-word` → permet de couper un mot très long au milieu. Changer en `word-break: normal` si on préfère que les mots ne soient jamais coupés.

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
