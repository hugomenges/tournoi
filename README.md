# 🏀 Tournoi Basket Live

Application web (un seul fichier `index.html`) pour organiser un tournoi de basket **en temps réel** : plusieurs équipes, plusieurs terrains joués simultanément, scores saisis en direct par les élèves sur téléphone ou tablette.

Hébergement : **GitHub Pages** · Données temps réel : **Firebase Realtime Database**

---

## ✨ Fonctionnalités

| Vue | Pour qui | Ce qu'on y fait |
|---|---|---|
| ⚙️ **Organisation** (protégée par PIN) | Enseignant | Équipes, terrains, génération du planning, corrections de scores, réglages, QR codes, impression |
| 📝 **Saisie des scores** | Élèves (1 appareil par terrain) | Choix du terrain, démarrer le match, boutons **+1 / +2 / +3**, annuler le dernier panier, chrono, terminer |
| 📺 **Écran** | Vidéoprojecteur / TV | Tous les terrains en direct, chronos, classement, matchs à suivre, derniers résultats |

- **Planning automatique** « tous contre tous » (aller simple ou aller-retour), réparti par **créneaux** sur tous les terrains disponibles, sans qu'une équipe joue deux fois sur le même créneau.
- **Le match suivant s'affiche tout seul** sur le terrain dès qu'un match est terminé.
- **Saisie fiable à plusieurs** : chaque panier est enregistré par transaction Firebase, donc deux appareils sur le même terrain ne s'écrasent pas.
- **Chrono synchronisé** (heure du serveur) avec pause et reprise, visible sur l'écran et sur les téléphones.
- **Classement en direct** : points (par défaut V = 3, N = 2, D = 1, barème UNSS, modifiable), puis différence de points, puis points marqués.
- **Impression du planning** en noir et blanc, avec des cases vides pour noter les scores à la main en secours.
- **Plusieurs tournois** dans la même base, chacun identifié par un code (ex. `BASKET4A`).

---

## 🚀 Installation (15 min)

### 1. Créer le projet Firebase
1. Va sur <https://console.firebase.google.com> → **Ajouter un projet** (Google Analytics n'est pas utile).
2. Menu **Créer → Realtime Database** → **Créer une base de données** → choisis la région **europe-west1 (Belgique)** → démarre en **mode verrouillé**.
3. Onglet **Règles** : colle les règles ci-dessous, puis clique sur **Publier**.
4. **Paramètres du projet** (⚙️) → **Vos applications** → icône **Web `</>`** → donne un nom à l'appli → copie l'objet `firebaseConfig`.

### 2. Configurer `index.html`
Ouvre `index.html` et remplace le bloc `firebaseConfig` en haut du script par le tien. Vérifie que la ligne `databaseURL` est bien présente : c'est l'URL affichée en haut de la page Realtime Database.

### 3. Publier sur GitHub Pages
1. Crée un dépôt (ex. `tournoi-basket`) et dépose `index.html` et `README.md` à la racine.
2. **Settings → Pages → Source : Deploy from a branch → `main` / `/ (root)`** → Save.
3. Au bout d'une minute environ, l'appli est en ligne à l'adresse `https://<ton-pseudo>.github.io/tournoi-basket/`.

> 💡 La clé `apiKey` de Firebase n'est pas un secret : elle peut se trouver dans un dépôt public. La sécurité repose sur les **règles** de la base de données.

---

## 🔐 Règles Realtime Database

Règles conseillées : lecture ouverte, écriture limitée à la branche `tournois` avec un minimum de validation.

```json
{
  "rules": {
    ".read": false,
    ".write": false,
    "tournois": {
      "$code": {
        ".read": true,
        ".write": true,
        ".validate": "$code.matches(/^[A-Z0-9_-]{2,30}$/)",
        "matchs": {
          "$m": {
            "sa": { ".validate": "newData.isNumber() && newData.val() >= 0 && newData.val() < 500" },
            "sb": { ".validate": "newData.isNumber() && newData.val() >= 0 && newData.val() < 500" }
          }
        }
      }
    }
  }
}
```

⚠️ **Limite à connaître** : le PIN organisateur protège l'**interface**, pas la base. Quelqu'un qui connaît le code du tournoi et qui s'y connaît techniquement pourrait écrire directement dans Firebase. Pour un tournoi en classe ou à l'AS, c'est suffisant. Pour aller plus loin, il faut ajouter Firebase Authentication (connexion anonyme ou par compte Google) et des règles basées sur `auth.uid`.

---

## 🧭 Utilisation le jour J

1. **Avant la séance** (organisateur)
   - Accueil → **Créer un tournoi** : nom, code, PIN.
   - **Équipes** : colle la liste des équipes, une par ligne.
   - **Terrains** : ajoute autant de terrains que nécessaire (Terrain 1, Terrain 2…).
   - **Réglages** : durée des matchs, barème.
   - **Matchs** → **⚡ Générer**, puis **🖨️ Imprimer** si tu veux une version papier de secours.
2. **Partage** : affiche les QR codes. Les élèves scannent le QR « Saisie » et choisissent leur terrain. L'appareil retient le terrain choisi.
3. **Vidéoprojecteur** : ouvre le lien « Écran », puis passe en plein écran avec ⛶.
4. **Pendant un match** : **▶ Démarrer** lance le chrono. Les élèves saisissent les paniers avec +1, +2 ou +3. ↩ annule le dernier panier en cas d'erreur. **🏁 Terminer** valide le score, et le match suivant apparaît automatiquement.
5. **Corrections** : depuis Organisation → Matchs, ✏️ corrige un score (format `12-8`), ↺ remet un match à zéro, 🗑 supprime un match.

**Idée pédagogique** : sur chaque terrain, une équipe au repos tient la table de marque (un élève à la saisie, un au chrono). Cela correspond aux rôles sociaux d'arbitre et d'OTM.

---

## 🗂️ Structure des données

```
tournois/
  {CODE}/
    meta:      { nom, pin (SHA-256), duree, pv, pn, pd, cree }
    equipes:   { {id}: { nom, couleur, ordre } }
    terrains:  { {id}: { nom, ordre } }
    matchs:    { {id}: { creneau, terrain, a, b, sa, sb,
                         statut: "attente" | "encours" | "fini",
                         chrono: { run, fin | reste }, log: [ {s, p} ], debut, fin } }
```

## 🔗 Liens directs

- Saisie : `…/index.html?t=CODE&v=eleve`
- Écran : `…/index.html?t=CODE&v=ecran`
- Organisation : `…/index.html?t=CODE&v=admin`

## 🛠️ Technique
- HTML, CSS et JavaScript (modules ES), sans étape de compilation.
- Firebase JS SDK 10 (modulaire) chargé depuis le CDN `gstatic`.
- QR codes : `qrcode-generator`, chargé depuis jsDelivr.
- Tester en local : `python3 -m http.server`, puis ouvrir <http://localhost:8000>. Un serveur local est nécessaire, car un fichier ouvert directement ne fonctionne pas : le calcul du PIN exige une connexion HTTPS ou localhost.
