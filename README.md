# 📅 JEC Apps Script — Synchronisation des événements

Script Google Apps Script qui lit les événements depuis Google Sheets
et met à jour automatiquement les pages événements de **jolietteencommun.org**
via l'API REST WordPress.

Travaille en tandem avec le repo **JOLIETTE-EN-COMMUN** (CSS) — le script
génère le HTML, le CSS le met en forme.

---

## Comment ça fonctionne

### Page de prévisualisation (8 événements)
1. Lit les événements dans deux onglets Google Sheets : `Mformulaire`
   (saisie manuelle) et `formulaire` (réponses Google Forms).
2. Filtre : garde uniquement les événements dont le statut est **« publié »**
   et dont la date de fin (ou de début) est aujourd'hui ou dans le futur.
3. Trie par date de début, ordre croissant.
4. Génère du HTML pour les **8 prochains événements** (grille `.jc-events-grid-layout`).
5. Ajoute les boutons d'action, le bouton mobile « Voir plus », le forum et le sondage.
6. Envoie à la page `PAGE_ID` via l'API REST WordPress.

### Page calendrier complet (tous les événements)
1. Même collecte et filtre que ci-dessus, mais **sans limite** de nombre.
2. Groupe les événements par mois.
3. Génère une liste compacte avec miniature, date, titre, méta et bouton.
4. Envoie à la page `PAGE_ID_COMPLET` via l'API REST WordPress.

---

## Structure des colonnes Google Sheets

Les deux onglets (`Mformulaire` et `formulaire`) respectent cette structure :

| Colonne | Index | Contenu |
|---------|-------|---------|
| A | 0 | *(non utilisée)* |
| B | 1 | Titre de l'événement |
| C | 2 | Description |
| D | 3 | Date de début |
| E | 4 | Heure de début |
| F | 5 | Date de fin |
| G | 6 | Heure de fin |
| H | 7 | Adresse |
| I | 8 | Ville |
| J | 9 | Coût |
| K | 10 | Lien bouton (URL vers inscription ou info) |
| L | 11 | Production / Présenté par |
| M | 12 | URL de l'image (doit commencer par `http`) |
| N | 13 | *(non utilisée)* |
| O | 14 | Statut — doit être exactement `publié` pour apparaître |
| P | 15 | État d'envoi (mis à jour automatiquement : `✓ Envoyé`) |

---

## Configuration — Script Properties

Les identifiants ne sont **jamais** dans le code.
Ils sont stockés dans **⚙️ Paramètres du projet → Propriétés de script**.

Voir `SCRIPT_PROPERTIES.example` pour les valeurs à saisir.

| Clé | Description |
|-----|-------------|
| `WP_URL` | URL publique du site WordPress (sans barre oblique finale) |
| `WP_USER` | Nom d'utilisateur WordPress administrateur |
| `WP_PASSWORD` | Mot de passe d'application WordPress (WP Admin → Utilisateurs → profil) |
| `PAGE_ID` | ID de la page prévisualisation (8 événements en grille) |
| `PAGE_ID_COMPLET` | ID de la page calendrier complet (tous les événements par mois) |

**Comment y accéder :**
1. Ouvre l'éditeur Apps Script
2. Clique sur ⚙️ **Paramètres du projet** (colonne de gauche)
3. Section **Propriétés de script → Ajouter une propriété**
4. Ajoute chaque clé du tableau avec sa vraie valeur

---

## Utilisation

**Page prévisualisation (8 événements) :**
Sheets → menu du haut → **📅 Événements → 🚀 Envoyer vers le site web**

**Page calendrier complet :**
Sheets → menu du haut → **📅 Événements → 📋 Mettre à jour le calendrier complet**

Le script marque les nouvelles lignes `✓ Envoyé` dans la colonne P.

---

## Fonctions

| Fonction | Rôle |
|----------|------|
| `synchroniserTimelineCitoyenne()` | Page prévisualisation — 8 événements en grille |
| `synchroniserCalendrierComplet()` | Page calendrier complet — tous les événements par mois |
| `genererBlocHTML(...)` | Génère le HTML d'une tuile (grille prévisualisation) |
| `genererLigneCalendrier(...)` | Génère le HTML d'une ligne (liste calendrier complet) |
| `onOpen()` | Ajoute le menu 📅 Événements dans Google Sheets |

---

## Structure HTML générée

### Page prévisualisation
```html
<div class="jc-events-header">
  <a class="gb-text gb-text-169aeb6e" ...>Aller au calendrier →</a>
  <a class="gb-text gb-text-d96ad459" ...>Ajouter une activité →</a>
</div>
<div class="jc-events-grid-layout">
  <ol class="jc-timeline">
    <li class="jc-timeline-item">...</li>
    <!-- jusqu'à 8 items -->
  </ol>
  <button id="jc-load-more-btn">Voir plus d'événements ▾</button>
</div>
<!-- Forum + Sondage -->
<div class="wp-block-columns">
  [wpforo]  |  [forminator_poll id="3623"]
</div>
```

### Page calendrier complet
```html
<div class="jc-calendrier-complet">
  <a class="jc-retour-btn">← Retour aux événements à venir</a>
  <div class="jc-cal-mois-groupe">
    <h2 class="jc-cal-mois-titre">Juin 2026</h2>
    <div class="jc-cal-evenement">
      <img class="jc-cal-img" ...>
      <div class="jc-cal-contenu">
        <div class="jc-cal-date">15 juin 2026 • 14h00</div>
        <h3 class="jc-cal-titre">Titre de l'événement</h3>
        <div class="jc-cal-meta">📍 Lieu | 💰 Coût</div>
      </div>
      <a class="jc-cal-btn">S'inscrire</a>
    </div>
  </div>
</div>
```

---

## Sécurité

- Les identifiants WordPress sont dans **Script Properties**, jamais dans le code.
- Le repo est **privé** : même sans identifiants, il révèle la structure
  du site et du tableur.
- Si le code a été partagé avec les identifiants visibles : révoquer
  immédiatement le mot de passe dans WP Admin → Utilisateurs →
  Mots de passe d'application.
- **Lien du formulaire :** utiliser le vrai lien de partage
  (bouton **Envoyer → icône 🔗** dans Google Forms), pas `?usp=preview`.

---

## À faire / évolutions prévues

- [x] ~~Corriger le bug des images (`imageUrL` non transmis)~~ ✓
- [x] ~~Ajouter un bouton « Voir plus » sur mobile~~ ✓
- [x] ~~Construire la page calendrier complet groupée par mois~~ ✓
- [ ] Créer la page WordPress du calendrier complet et configurer `PAGE_ID_COMPLET`
- [ ] Vérifier et remplacer le lien Google Forms par le vrai lien de partage
- [ ] Mettre à jour `PAGE_ID` vers la vraie page événements (remplace la page de test 3793)
- [ ] Mettre à jour le lien « Aller au calendrier » dans le script vers la nouvelle URL
