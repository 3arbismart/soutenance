# Fiche révision — Soutenance Dominion Seaside (Anas)

> Fiche courte à relire **juste avant** de passer. Le détail est dans
> [preparation-soutenance.md](preparation-soutenance.md).

---

## 0. Le format (ce que le jury attend)

- **25-30 min** : 3-4 min de démo, **le reste = questions sur MON code + modifs en direct**.
- **Pas de slides.** On discute uniquement du code.
- Code évalué = celui du **dépôt GitLab figé** (dernier commit attendu : jeudi soir).
  → *Vérifier que le dernier commit est le bon ; sinon `git checkout` vers ce commit.*
- **Chacun parle de SA partie.** La note est **individuelle**.
- ⚠️ **Réponse fausse = concept « non acquis ».** Une belle IHM ne suffit pas : il faut
  **comprendre chaque ligne** que j'ai écrite.

---

## 1. Les 5 critères notés → où c'est dans MON code

| # | Critère du jury | Ma réponse (fichier · fonction) |
|---|---|---|
| 1 | **Bindings + listeners** (réactions aux actions) | `CurrentPlayerView.bindToPlayer` (bind compteurs + listeners zones) · `GameView.watchPlays` (journal) · `createBindings` (instruction, zone temporaire) |
| 2 | **Richesse des composants / layouts** | `BorderPane` (main.fxml : haut/centre/droite/bas) · `HBox` 3 blocs (currentPlayer) · `FlowPane` (cartes) · `TilePane` (réserve) · `StackPane` (carte + overlay) · `ListView` (journal) |
| 3 | **Redimensionnement cohérent** | `DominionIHM.startGame` : `ScrollPane` + `setFitToWidth(true)`, `setMaximized`, tailles mini · `FlowPane` qui passe à la ligne |
| 4 | **Infos des autres joueurs** | ⚠️ **point faible — voir §4** (la vue dédiée n'est pas finie ; le **journal** montre quand même ce que l'autre joue) |
| 5 | **Vues supplémentaires** | `ChoosePlayersView` (fenêtre d'accueil, saisie des noms) — **entièrement à moi** |

---

## 2. À RÉVISER À FOND : bindings vs listeners (critère le plus lourd)

**La phrase de base :** « `bind` = la valeur se recopie toute seule ; `addListener` = j'exécute
**du code** quand ça change. »

| J'utilise… | Quand | Exemple chez moi |
|---|---|---|
| `bind()` | valeur simple à recopier | `actionsLabel.textProperty().bind(...)`, instruction, badges de réserve |
| `addListener()` | il faut **reconstruire** ou agir | main, cartes en jeu, tapis (reconstruction), journal (`watchPlays`) |

**Les 3 questions quasi sûres :**

1. **`bind` vs `addListener` ?** → réponse ci-dessus. *bind* pour les compteurs, *listener* pour
   les listes de cartes (je vide puis je remplis la zone).
2. **`setVisible` vs `setManaged` ?** → `visible=false` cache **mais garde la place** ;
   `managed=false` **retire de la mise en page** (récupère l'espace). Je mets **les deux** pour les
   tapis vides (`setBlockVisible`).
3. **Pourquoi détacher l'ancien joueur dans `bindToPlayer` ?** → sinon **fuite mémoire** + l'ancien
   joueur continuerait de déclencher des refresh (mises à jour fantômes). Donc je `removeListener` /
   `unbind` avant de relier le nouveau.

**Comment le journal marche sans toucher au moteur (très probable) :** j'**observe**
`player.getInPlay()` ; jouer une carte l'ajoute à cette liste → mon `ListChangeListener` détecte
l'ajout → j'écris « X joue Y ». Le modèle est la source de vérité, la vue **réagit**.

---

## 3. La démo (3-4 min) — déroulé à dérouler sans hésiter

1. **Lancement** → la **fenêtre d'accueil** (`ChoosePlayersView`) : je saisis 2 noms, « Commencer ».
2. La partie s'ouvre **maximisée** ; je montre la structure : autres joueurs en haut, plateau +
   réserve au centre, **journal à droite**, joueur courant en bas.
3. Je **joue un trésor / une carte** → on voit le **compteur d'argent** bouger (binding) et la ligne
   apparaître dans le **journal** (listener).
4. Je **survole une carte 3 s** → **aperçu plein écran**, je clique pour fermer.
5. Si une carte tapis sort (Island/Native Village), montrer que le **bloc apparaît** puis
   **disparaît** quand il est vide.
6. Montrer le **redimensionnement** : réduire la fenêtre → les cartes passent à la ligne, barre de
   défilement, rien n'est coupé.

---

## 4. ⚠️ Point faible à anticiper — les autres joueurs (critère 4)

**État réel :** `OtherPlayersView` est **vide** et la zone `otherPlayersZone` (en haut) **n'est pas
remplie**. Donc le panneau dédié aux autres joueurs **n'est pas terminé**.

**Ce que je dis si on me pose la question (honnête + je maîtrise le concept) :**

> « Le panneau dédié aux autres joueurs n'est pas finalisé. En revanche, le **journal des actions**
> montre en direct ce que **chaque** joueur joue. Pour le compléter, je sais comment faire : la vue
> écouterait `currentPlayerProperty`, et pour chaque joueur **non courant** j'afficherais ses infos
> **publiques** (nom, nombre de cartes en main, taille de pioche/défausse) via des bindings. »

**Pseudo-code à savoir réciter :**

```
fonction afficherAutresJoueurs :
    vider la zone du haut
    POUR chaque joueur de la partie qui N'EST PAS le joueur courant :
        créer un petit bloc : nom + "cartes en main : N"
        LIER N à la taille de sa main (Bindings.size)
        ajouter le bloc dans la zone du haut
    refaire ça quand le joueur courant change
```

> 💡 **Si le dépôt n'est pas encore figé**, c'est LE point à combler (≈ 30 lignes) — dis-le-moi,
> je te l'implémente proprement (et ça transforme un critère « manquant » en critère « validé »).

---

## 5. Modifs « en direct » : les réflexes

Le jury va demander de **modifier le code en direct**. Les cas préparés sont en
[preparation-soutenance.md §4](preparation-soutenance.md). Les plus probables :

- **Changer le délai du plein écran** (3 s → 1 s) → 1 ligne : `FULLSCREEN_HOVER_DELAY`.
- **Limiter le journal** aux 50 dernières lignes → dans `watchPlays`, `if size>50 remove(0)`.
- **Bouton « Effacer le journal »** → `onClearLog()` qui fait `logEntries.clear()`.
- **Ajouter un 3ᵉ joueur** → `getNumberOfPlayers()` renvoie 3 + un `player3Field`.
- **Afficher le nombre de cartes en main** → `bind` sur `Bindings.size(player.getHand())`.

**Réflexe modif en direct :** valeur simple → `bind` ; réaction → `addListener` ; toujours penser à
**détacher** si je relie à un autre joueur.

---

## 6. Frontière Anas / Zakaria (à dire clairement — note individuelle)

| Fichier | **Moi (Anas)** | Zakaria |
|---|---|---|
| `GameView` | `BorderPane`, `createBindings`, **journal** (`setupActionLog`/`watchPlays`) | `buildSupply` (réserve) |
| `CurrentPlayerView` | panneau, `bindToPlayer`, `refresh*`, `fillCardZone`, FlowPane, tapis masqués | **chronomètre** (`startTimer`) |
| `CardView` | `loadImage`, plein écran (`showFullScreen`/`hideFullScreen`), minuteur 3 s | **zoom** au survol |
| `DominionIHM` | `ScrollPane` + `setMaximized`, activation fenêtre d'accueil | correction `getRandomKingdomCards` |
| `ChoosePlayersView` | **tout** (fenêtre d'accueil) | — |
| `ScoresView` | — | **tout** (fin de partie) |

**Phrase de secours :** « Cette partie-là, c'est Zakaria qui l'a faite, mais je peux vous
l'expliquer. »

---

## 7. Checklist avant d'entrer

- [ ] Le projet **compile et se lance** (tester un lancement complet juste avant).
- [ ] Le **dernier commit** est bien celui attendu (sinon `git checkout` dessus).
- [ ] ⚠️ Le fichier de contributions s'appelle bien **`ContributionsIndividuelles.md`** (l'e-mail le
      nomme précisément ; le dépôt a `contributionprojet.md` → **le renommer / le créer**).
- [ ] Je sais expliquer **bind vs listener** (§2) sans hésiter.
- [ ] Je sais quoi répondre sur **les autres joueurs** (§4).
- [ ] Je connais ma **frontière** avec Zakaria (§6).
```
