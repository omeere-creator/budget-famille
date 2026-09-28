# Budget Famille

Application web (PWA) de budgets familiaux mensuels : une jauge par budget (courses, sorties, dépenses fixes…) qui se remplit au fil des dépenses. Les dépenses s'ajoutent en photographiant un ticket, en important une capture d'écran de l'app bancaire, ou à la main.

## Fonctionnement

- **De salaire à salaire** : chaque budget repart à zéro le jour où le salaire Stellantis arrive sur le compte (normalement entre le 25 et le 31). Tant que le salaire n'est pas arrivé, la période en cours continue. Un salaire en retard, jusqu'au 5 du mois suivant, est bien reconnu. Les autres virements Stellantis, comme les notes de frais en milieu de mois, n'ouvrent pas de nouvelle période. Sans compte bancaire relié, la période démarre au jour habituel (25 par défaut). Tout se règle dans ⚙️ → *Période budgétaire*, qui permet aussi de revenir au mois civil. Les périodes passées restent consultables avec les flèches ‹ ›.
- **Jauges** : la barre verticale fine indique où tu *devrais* être à cette date du mois. La jauge passe en orange si tu dépenses plus vite que le rythme, en rouge si le budget est dépassé.
- **Scan** : la photo est envoyée à l'API Claude, qui renvoie commerçant, date, montant et budget suggéré. Un ticket donne une dépense ; une capture bancaire peut en donner plusieurs, que tu coches avant d'ajouter. Tout reste modifiable avant validation.
- **Données** : stockées dans le navigateur du téléphone (aucun serveur). Export / restauration JSON dans les réglages.

## Mise en ligne sur GitHub Pages

1. Crée un dépôt sur GitHub (ex. `budget-famille`). Il peut être public : aucune donnée ni clé n'est dans le code.
2. Dépose les fichiers de ce dossier à la racine (`index.html`, `manifest.json`, `sw.js`, `icon-192.png`, `icon-512.png`, `README.md`).
3. **Settings → Pages** → Source : *Deploy from a branch* → branche `main`, dossier `/ (root)` → Save.
4. Après une minute, l'app est en ligne sur `https://<ton-pseudo>.github.io/budget-famille/`.

## Installation sur le téléphone

- **iPhone (Safari)** : ouvre l'adresse → bouton Partager → *Sur l'écran d'accueil*.
- **Android (Chrome)** : menu ⋮ → *Installer l'application*.

Utilise toujours l'app depuis l'icône installée : c'est là que les données sont conservées durablement (sur iPhone, un site non installé peut voir ses données effacées après quelques semaines sans visite).

## Clé API pour le scan

1. Crée un compte sur [console.anthropic.com](https://console.anthropic.com) et ajoute un peu de crédit (c'est distinct d'un abonnement Claude).
2. Crée une clé API, et fixe une **limite de dépense mensuelle** basse dans la console.
3. Colle la clé dans l'app : ⚙️ → *Lecture des tickets*.

Coût indicatif avec Claude Haiku 4.5 : une fraction de centime par ticket. La clé n'est jamais incluse dans les exports ni dans le code.

## Compte bancaire : tous les débits et crédits

Un robot installé dans ton dépôt privé `budget-inbox` lit ton compte BNP Paribas Fortis toutes les 3 heures, via l'open banking européen (PSD2, lecture seule, Enable Banking). Installation : voir le README du dossier `budget-inbox`.

Dans l'app :
- **Débits** : un mouvement déjà connu n'est pas recréé. C'est le cas d'un paiement Apple Pay, d'une dépense encodée ou scannée, ou d'une dépense récurrente. L'app le rattache simplement à la ligne bancaire. Une facture récurrente au montant variable (énergie…) prend le montant réel.
- **Crédits** : proposés comme 💶 *Revenu* (salaire, allocations…) ou comme ↩︎ *remboursement* déduit d'un budget.
- **Classement** : un bénéficiaire déjà rencontré est classé automatiquement. Les nouveaux passent par « À vérifier », avec une proposition de Claude et un nom nettoyé si la clé API est configurée.
- **Carte « Solde du compte »** : solde, entrées, dépenses et solde du mois. Touche-la pour voir le détail des entrées.
- L'import commence au dernier salaire reçu avant la connexion. Chaque salaire est classé automatiquement en revenu et ouvre la période suivante.

## Dépenses récurrentes

Pour les domiciliations et ordres permanents (prêt, énergie, assurances, abonnements…) : ⚙️ → *Dépenses récurrentes*, ou coche « Répéter chaque mois » en encodant une dépense.

- Chaque dépense récurrente s'inscrit **automatiquement le jour du prélèvement**, une seule fois par mois. Si tu en supprimes une, elle ne revient pas ce mois-là.
- Avant l'échéance, la jauge montre la partie **encore à prélever** en couleur claire, avec le « reste réel » après ces prélèvements. Un dépassement à venir est signalé avant qu'il n'arrive.
- Modifier le montant ne touche que les mois suivants. Mettre en pause ou supprimer conserve l'historique.

## Paiements Apple Pay automatiques (iPhone)

À chaque paiement Apple Pay, un raccourci iOS dépose silencieusement le montant et le commerçant dans un dépôt GitHub privé. À l'ouverture, l'app les récupère, les classe et ferme la demande. Aucun scan, rien à faire à la caisse.

**Classement** : un commerçant déjà rencontré va directement dans son budget. Un nouveau commerçant arrive dans « À vérifier », avec une proposition de Claude si la clé API est configurée. Un seul toucher le confirme, et il sera classé automatiquement ensuite.

### 1. Le dépôt de réception (une fois)
1. Sur GitHub, crée un dépôt **privé** nommé `budget-inbox`, en cochant « Add a README ».
2. Photo de profil → **Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token**.
   - *Repository access* : **Only select repositories** → `budget-inbox`
   - *Permissions → Repository permissions* : **Issues → Read and write**
   - *Expiration* : la plus longue possible (note la date pour le renouveler).
3. Copie le jeton (`github_pat_…`).
4. Dans l'app : ⚙️ → *Paiements Apple Pay automatiques* → dépôt `ton-pseudo/budget-inbox` + jeton → Enregistrer.

### 2. L'automatisation iOS (une fois)
1. App **Raccourcis** → onglet **Automatisation** → **+** → **Transaction**.
2. Choisis ta ou tes cartes BNP Paribas Fortis, laisse toutes les catégories, puis coche **Exécuter immédiatement** → Suivant → **Nouveau raccourci vide**.
3. Ajoute l'action **Obtenir le contenu de l'URL** :
   - URL : `https://api.github.com/repos/ton-pseudo/budget-inbox/issues`
   - Touche la flèche pour déplier : **Méthode** `POST`
   - **En-têtes** :
     - `Authorization` = `Bearer github_pat_…` (ton jeton)
     - `Accept` = `application/vnd.github+json`
   - **Corps de la requête** : `JSON`, deux champs de type Texte :
     - `title` = `depense`
     - `body` = deux lignes :
       ```
       montant=[Montant]
       commercant=[Commerçant]
       ```
       où `[Montant]` et `[Commerçant]` sont des variables : touche *Entrée du raccourci* (la transaction) et choisis la propriété Montant (*Amount*), puis Commerçant (*Merchant*).
4. Fais un petit paiement test, puis ouvre l'app : il apparaît dans « À vérifier ».

L'automatisation se déclenche sur les paiements faits avec le Wallet de l'iPhone. Les virements, domiciliations et paiements par carte physique n'y passent pas : ajoute-les à la main ou par capture bancaire.

## Sauvegardes

⚙️ → *Exporter une sauvegarde* crée un fichier `.json`. Garde-le dans iCloud / Google Drive ; *Restaurer* le recharge (utile en changeant de téléphone).
