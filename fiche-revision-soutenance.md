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

## 7. Boutons, clics & automatismes — qui, quoi, **où dans le code**

> Style « là j'ai fait un bind pour… ». Pour chaque élément : **qui** l'explique + le **fichier · fonction**.

### Les vrais boutons

| Bouton | Qui | Où (UI + logique) |
|---|---|---|
| **Commencer la partie** | Moi | UI [choosePlayers.fxml:39](src/main/resources/fxml/choosePlayers.fxml#L39) → [ChoosePlayersView.java:85](src/main/java/fr/umontpellier/iut/dominionfx/views/ChoosePlayersView.java#L85) `setPlayersNamesList()` ; lancement par le listener [DominionIHM.java:224](src/main/java/fr/umontpellier/iut/dominionfx/DominionIHM.java#L224) |
| **Passer** | Moi | UI [currentPlayer.fxml:64](src/main/resources/fxml/currentPlayer.fxml#L64) → [CurrentPlayerView.java:251](src/main/java/fr/umontpellier/iut/dominionfx/views/CurrentPlayerView.java#L251) `onSkip()` → `game.skipWasChosen()` |
| **Jouer les trésors** | Moi | UI [currentPlayer.fxml:66](src/main/resources/fxml/currentPlayer.fxml#L66) → [CurrentPlayerView.java:257](src/main/java/fr/umontpellier/iut/dominionfx/views/CurrentPlayerView.java#L257) `onPlayTreasures()` |
| **Oui / Non** | Moi | UI [currentPlayer.fxml:68](src/main/resources/fxml/currentPlayer.fxml#L68) → [CurrentPlayerView.java:263](src/main/java/fr/umontpellier/iut/dominionfx/views/CurrentPlayerView.java#L263) `onYes()`/`onNo()` ; zone désactivée par bind [CurrentPlayerView.java:160](src/main/java/fr/umontpellier/iut/dominionfx/views/CurrentPlayerView.java#L160) |

### Les clics sur les cartes (zones cliquables, pas des boutons)

| Clic | Qui | Où |
|---|---|---|
| Carte **de la main** → jouer | Moi | [CurrentPlayerView.java:241](src/main/java/fr/umontpellier/iut/dominionfx/views/CurrentPlayerView.java#L241) `createCardPlaceholder()` |
| Pile de la **réserve** → acheter | Zakaria | [GameView.java:164](src/main/java/fr/umontpellier/iut/dominionfx/views/GameView.java#L164) `createSupplyPileView()` |
| Survol → **zoom** | Zakaria | [CardView.java:167](src/main/java/fr/umontpellier/iut/dominionfx/views/CardView.java#L167) `buildTransition()` |
| Survol **3 s → plein écran** + clic pour fermer | Moi | minuteur [CardView.java:103](src/main/java/fr/umontpellier/iut/dominionfx/views/CardView.java#L103) ; [CardView.java:113](src/main/java/fr/umontpellier/iut/dominionfx/views/CardView.java#L113) `showFullScreen()` / [CardView.java:151](src/main/java/fr/umontpellier/iut/dominionfx/views/CardView.java#L151) `hideFullScreen()` |

### Les petits automatismes (bind / listener, sans bouton)

| Automatisme | Qui | Où |
|---|---|---|
| Texte d'**instruction** (bind) | Moi | [GameView.java:118](src/main/java/fr/umontpellier/iut/dominionfx/views/GameView.java#L118) `createBindings()` |
| Zone **cartes temporaires** (bind visible/managed) | Moi | [GameView.java:120](src/main/java/fr/umontpellier/iut/dominionfx/views/GameView.java#L120) `createBindings()` |
| **Compteurs** actions/achats/argent/pioche/défausse (bind) | Moi | [CurrentPlayerView.java:154](src/main/java/fr/umontpellier/iut/dominionfx/views/CurrentPlayerView.java#L154) `bindToPlayer()` |
| **Points de victoire** (listener + recalcul) | Moi | [CurrentPlayerView.java:179](src/main/java/fr/umontpellier/iut/dominionfx/views/CurrentPlayerView.java#L179) `refreshVictoryPoints()` |
| **Journal** des actions (listener + scroll) | Moi | [GameView.java:80](src/main/java/fr/umontpellier/iut/dominionfx/views/GameView.java#L80) `setupActionLog()` / [:97](src/main/java/fr/umontpellier/iut/dominionfx/views/GameView.java#L97) `watchPlays()` |
| **Tapis masqués** quand vides (visible+managed) | Moi | [CurrentPlayerView.java:194](src/main/java/fr/umontpellier/iut/dominionfx/views/CurrentPlayerView.java#L194) `refreshIslandMat()` / [:212](src/main/java/fr/umontpellier/iut/dominionfx/views/CurrentPlayerView.java#L212) `setBlockVisible()` |
| **Badge nombre** + pile grisée (bind) | Zakaria | [GameView.java:156](src/main/java/fr/umontpellier/iut/dominionfx/views/GameView.java#L156) `createSupplyPileView()` |
| **Chronomètre** (Timeline 1 s) | Zakaria | [CurrentPlayerView.java:106](src/main/java/fr/umontpellier/iut/dominionfx/views/CurrentPlayerView.java#L106) `startTimer()` |
| Fenêtre **maximisée** + ScrollPane responsive | Moi | [DominionIHM.java:83](src/main/java/fr/umontpellier/iut/dominionfx/DominionIHM.java#L83) `startGame()` |

> **Astuce mémoire :** mes boutons/clics sont dans `CurrentPlayerView` (+ `ChoosePlayersView`), mes
> binds d'affichage dans `GameView.createBindings` et `CurrentPlayerView.bindToPlayer`. Tout ce qui
> touche la **réserve** (`GameView.createSupplyPileView`) et le **chrono** est à Zakaria.

---

## 8. Checklist avant d'entrer

- [ ] Le projet **compile et se lance** (tester un lancement complet juste avant).
- [ ] Le **dernier commit** est bien celui attendu (sinon `git checkout` dessus).
- [ ] ⚠️ Le fichier de contributions s'appelle bien **`ContributionsIndividuelles.md`** (l'e-mail le
      nomme précisément ; le dépôt a `contributionprojet.md` → **le renommer / le créer**).
- [ ] Je sais expliquer **bind vs listener** (§2) sans hésiter.
- [ ] Je sais quoi répondre sur **les autres joueurs** (§4).
- [ ] Je connais ma **frontière** avec Zakaria (§6).
