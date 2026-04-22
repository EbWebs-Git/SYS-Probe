# 💬 Idées de débats — Multi'Verts

Plateforme web de soumission et de vote pour des sujets de débat, connectée en temps réel à Google Sheets via Google Apps Script.

![Badge](https://img.shields.io/badge/stack-HTML%20%2F%20CSS%20%2F%20JS-2d7d46?style=flat-square)
![Badge](https://img.shields.io/badge/backend-Google%20Apps%20Script-f5c518?style=flat-square)
![Badge](https://img.shields.io/badge/database-Google%20Sheets-34a853?style=flat-square)

---

## 📌 Présentation

Ce projet est une application web **100% frontend** (un seul fichier `index.html`) qui utilise Google Sheets comme base de données et Google Apps Script comme API REST légère. Il a été développé pour le groupe **Multi'Verts** afin de permettre aux membres de proposer et voter pour des sujets de débat de manière simple et ludique.

---

## ✨ Fonctionnalités

- **Soumettre un débat** — titre, description, catégorie, auteur, groupe
- **Voter** — 👍 / 👎 par sujet, modifiable à tout moment, annulable
- **Classement automatique** — les sujets sont triés par score (likes − dislikes)
- **Badge "Prochain débat"** — mis en avant automatiquement sur le sujet #1
- **12 catégories** — Société, Écologie, Jeunesse, Culture, Sport, Numérique, Aventure, Service, Créativité, Nature, Solidarité, Autre
- **Archives mensuelles** — archivage automatique des débats en fin de mois
- **Page À propos dynamique** — contenu modifiable directement depuis Google Sheets (cellule Z2 de Feuille 1)
- **Rafraîchissement automatique** — toutes les 60 secondes
- **Responsive** — mobile et desktop

---

## 🏗️ Architecture

```
index.html                  ← Application complète (HTML + CSS + JS)
    │
    ├── fetch GET  ──────► Google Apps Script (doGet)
    │                           │
    │                           ├── ?action=apropos → lit Feuille 1!Z2
    │                           └── (défaut)        → lit Feuille 1 (toutes les lignes)
    │
    └── fetch POST ──────► Google Apps Script (doPost)
                                │
                                ├── action: "append" → ajoute une ligne
                                └── action: "update" → met à jour une cellule (votes)
```

### Stack technique

| Couche | Technologie |
|--------|------------|
| Frontend | HTML5 / CSS3 / JavaScript ES2020 (vanilla) |
| Fonts | Google Fonts — Fredoka One + Nunito |
| Backend | Google Apps Script (déployé en Web App) |
| Base de données | Google Sheets |
| Stockage local | localStorage (votes + archives) |

---

## 📂 Structure du Google Sheet

### Feuille 1 — Base de données des débats

| Colonne | Contenu |
|---------|---------|
| A | `id` — identifiant unique (ex: `idea_1234567890`) |
| B | `titre` — sujet du débat |
| C | `description` — description détaillée |
| D | `categorie` — slug de catégorie (ex: `ecologie`) |
| E | `auteur` — prénom ou surnom |
| F | `groupe` — groupe / unité |
| G | `date` — date de soumission |
| H | `likes` — nombre de votes positifs |
| I | `dislikes` — nombre de votes négatifs |
| Z | `apropos` *(ligne 2)* — contenu de la page À propos |

> La ligne 1 contient les en-têtes. Les données commencent à la ligne 2.

---

## 🚀 Installation & Déploiement

### 1. Cloner / télécharger

```bash
git clone https://github.com/ton-user/multiverts-debats.git
```

Ou simplement télécharger `index.html`.

### 2. Créer le Google Sheet

1. Créer un nouveau Google Sheets
2. Nommer la première feuille **Feuille 1**
3. En ligne 1, ajouter les en-têtes : `id`, `titre`, `description`, `categorie`, `auteur`, `groupe`, `date`, `likes`, `dislikes`
4. En cellule **Z2**, écrire le texte de la page À propos

### 3. Créer le Google Apps Script

1. Dans le Google Sheet → **Extensions → Apps Script**
2. Remplacer tout le contenu par :

```javascript
const SHEET_ID = 'VOTRE_SHEET_ID_ICI';

function doGet(e) {
  if (e.parameter.action === 'apropos') {
    const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName('Feuille 1');
    const texte = sheet.getRange('Z2').getValue();
    return ContentService.createTextOutput(JSON.stringify({ apropos: texte }))
      .setMimeType(ContentService.MimeType.JSON);
  }
  const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName('Feuille 1');
  const rows = sheet.getDataRange().getValues();
  const headers = rows[0];
  const data = rows.slice(1).map(r => {
    const obj = {};
    headers.forEach((h, i) => obj[h] = r[i]);
    return obj;
  });
  return ContentService.createTextOutput(JSON.stringify(data))
    .setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  const sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName('Feuille 1');
  const body = JSON.parse(e.postData.contents);
  if (body.action === 'append') sheet.appendRow(body.row);
  if (body.action === 'update') sheet.getRange(body.row, body.col).setValue(body.val);
  return ContentService.createTextOutput('ok').setMimeType(ContentService.MimeType.TEXT);
}
```

3. Remplacer `VOTRE_SHEET_ID_ICI` par l'ID de votre feuille (visible dans l'URL)

### 4. Déployer l'Apps Script

1. **Déployer → Nouveau déploiement**
2. Type : **Application Web**
3. Exécuter en tant que : **Moi**
4. Accès : **Tout le monde**
5. Cliquer **Déployer** et copier l'URL générée

### 5. Configurer le frontend

Dans `index.html`, remplacer la valeur de `API_URL` :

```javascript
const API_URL = 'https://script.google.com/macros/s/VOTRE_URL_ICI/exec';
```

### 6. Héberger

Le fichier `index.html` peut être hébergé n'importe où :
- GitHub Pages
- Netlify / Vercel (drag & drop)
- Tout hébergement statique

---

## ⚙️ Configuration

| Constante | Emplacement | Description |
|-----------|------------|-------------|
| `API_URL` | `index.html` ligne ~352 | URL du Web App Google Apps Script |
| `VOTES_KEY` | `index.html` | Clé localStorage pour les votes |
| `ARCH_KEY` | `index.html` | Clé localStorage pour les archives |
| Texte À propos | Google Sheets `Feuille 1!Z2` | Contenu de la page À propos |

---

## 🔄 Mettre à jour le déploiement Apps Script

> ⚠️ Après chaque modification du code Apps Script, il faut **redéployer** en créant une **nouvelle version** — sinon l'ancienne version reste active.

1. Apps Script → **Déployer → Gérer les déploiements**
2. Cliquer l'icône ✏️
3. Version → **Nouvelle version**
4. **Déployer**

---

## 🛠️ Développement local

Aucune dépendance, aucun build. Ouvrir directement `index.html` dans un navigateur.

> Note : les appels fetch vers Google Apps Script peuvent être bloqués en `file://`. Utiliser un serveur local :

```bash
python3 -m http.server 8080
# puis ouvrir http://localhost:8080
```

---

## 📄 Licence

Projet développé par **Ethan MORGADO** pour le Multi'Verts.
Contact : [contact@ebwebs.org](mailto:contact@ebwebs.org)
