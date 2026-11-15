# Connecteurs MCP : faire parler ton agent avec l'extérieur

## C'est quoi MCP ? (version débutant)

**MCP = Model Context Protocol.** C'est un standard ouvert qui permet de brancher une IA (Claude, ChatGPT, un agent dans ton éditeur…) sur des outils et des données externes : fichiers, bases de données, API, réseaux sociaux…

La doc officielle le compare à un **port USB-C pour les applis d'IA** : une seule prise standard, plein d'appareils compatibles. Source : [modelcontextprotocol.io](https://modelcontextprotocol.io/docs/getting-started/intro).

Il y a trois acteurs :

| Acteur | C'est quoi | Exemple |
|--------|-----------|---------|
| **Client** (ou hôte) | L'appli où tu parles à l'IA | Claude Code, Claude Desktop, Cursor, VS Code |
| **Serveur MCP** | Un petit programme qui expose des « outils » à l'IA | Le serveur MCP d'X, un serveur Instagram |
| **Service externe** | La vraie plateforme derrière | L'API d'X, l'API Instagram de Meta |

Concrètement : tu déclares le serveur dans un fichier de config de ton client → le client le lance (ou s'y connecte) → l'IA voit de nouveaux outils (« chercher des posts », « lire mes mentions »…) et peut les appeler, en te demandant la permission selon les réglages de ton outil.

Deux façons de faire tourner un serveur :

- **Local (stdio)** : le client lance un programme sur ta machine (souvent via `npx`).
- **Distant (HTTP)** : le client se connecte à une URL.

## Le fichier d'exemple

[`mcp.example.json`](mcp.example.json) est au format `.mcp.json` de **Claude Code** (clé `mcpServers`). Claude Code remplace `${NOM_DE_VARIABLE}` par la valeur de la variable d'environnement correspondante, dans `command`, `args`, `env`, `url` et `headers` ([doc](https://code.claude.com/docs/en/mcp)). C'est ce qui permet de **partager la config sans partager tes clés**.

D'autres clients utilisent un format très proche (Cursor, Claude Desktop : clé `mcpServers` aussi ; VS Code : clé `servers`). Regarde la doc de ton client pour l'emplacement exact du fichier.

### Mise en place (Claude Code)

```bash
# 1. Copie l'exemple à la racine de TON projet
cp mcp/mcp.example.json .mcp.json

# 2. Crée ton fichier de secrets local (ignoré par git) et remplis-le
cp mcp/.env.example .env

# 3. Charge les variables dans ton terminal puis lance Claude Code
set -a; source .env; set +a      # bash / zsh
claude

# 4. Vérifie l'état des serveurs
claude mcp list
```

À la première ouverture, Claude Code te demande d'**approuver** les serveurs déclarés dans `.mcp.json` (c'est voulu : un repo cloné ne peut pas s'auto-approuver).

Supprime le bloc `instagram` de ton `.mcp.json` tant que tu ne l'as pas configuré, sinon le serveur échouera au démarrage.

> Sous Windows, la syntaxe `source .env` ne marche pas dans PowerShell : définis les variables autrement (paramètres système, ou `$env:X_CLIENT_ID="..."` dans la session). *À vérifier selon ton terminal.*

## Exemple 1 — X / Twitter (serveur officiel)

X publie lui-même un serveur MCP : `https://api.x.com/mcp`, qu'on atteint via le pont open source **`xurl`** (`npx -y @xdevplatform/xurl mcp https://api.x.com/mcp`). Source : [docs.x.com/tools/mcp](https://docs.x.com/tools/mcp).

**Pré-requis :**

1. Un compte développeur et une **app X** avec **OAuth 2.0** activé, créée sur [developer.x.com](https://developer.x.com).
2. Déclarer l'URL de redirection `http://localhost:8080/callback` dans l'app (nécessaire pour la première connexion dans le navigateur).
3. Récupérer `CLIENT_ID` et `CLIENT_SECRET` → dans ton `.env` (`X_CLIENT_ID`, `X_CLIENT_SECRET`).
4. Node.js installé (pour `npx`).

**Au premier lancement**, le pont ouvre ton navigateur pour te connecter à X, puis garde et rafraîchit le token tout seul (cache dans un dossier `~/.xurl` de ton compte utilisateur). La doc d'X conseille un délai de démarrage généreux côté client pour laisser le temps à ce premier login.

**Ce que la doc d'X annonce comme possible** (via ton compte) : récupérer des posts, rechercher des posts / utilisateurs / actus, lire tes mentions et ta timeline, gérer tes favoris (bookmarks), tendances, et créer / publier des **Articles**.

**À vérifier avant de compter dessus :**

- Si ce serveur permet de publier un **post classique** (un tweet) : ce n'est pas listé clairement dans la doc officielle au moment de l'écriture.
- Le coût d'accès à l'API X : l'accès dépend de l'offre (« package ») de ton app dans le portail développeur. **Consulte les tarifs sur developer.x.com, aucun prix n'est donné ici.**

## Exemple 2 — Instagram (modèle générique)

Il n'y a pas, à notre connaissance, de serveur MCP Instagram officiel publié par Meta (*à vérifier*). Il existe des serveurs communautaires qui s'appuient sur l'**API officielle Instagram** de Meta.

Ce qu'on sait côté Meta ([doc Instagram Platform](https://developers.facebook.com/docs/instagram-platform/overview)) :

- L'API donne accès aux comptes Instagram **professionnels** (Business ou Creator). Un compte perso classique ne suffit pas : passe ton compte en professionnel dans l'appli Instagram.
- Il faut créer une **app Meta** sur [developers.facebook.com](https://developers.facebook.com) et obtenir un **token d'accès** avec les bonnes permissions.
- Il y a des limites de débit et de publication : consulte la doc officielle pour les valeurs actuelles.

**Comment choisir un serveur communautaire :**

1. Il utilise **uniquement l'API officielle de Meta** (pas de scraping, pas de login par mot de passe ni cookie de session : risque de blocage de ton compte).
2. Le code est public, maintenu récemment, et le README liste clairement les outils et permissions.
3. Les outils de **publication** sont désactivables, ou au minimum séparés des outils de lecture.

Ensuite, remplace dans `.mcp.json` : `[commande-du-serveur-mcp-instagram]`, les `args`, et les **noms de variables** dans `env` par ceux qu'indique le README du serveur choisi (ceux de l'exemple sont indicatifs, *à vérifier*).

## Règles de sécurité (à lire, vraiment)

1. **Aucun secret dans un fichier commité.** `.mcp.json` ne contient que des `${VARIABLES}`. Les vraies valeurs vont dans `.env` (ignoré par git).
2. **Validation humaine avant toute publication.** Un agent connecté à un réseau social peut poster en ton nom. Garde les demandes de permission activées pour les outils d'écriture et relis chaque contenu.
3. **Le contenu externe peut piéger l'agent.** Un post ou un commentaire lu par l'agent peut contenir des instructions malveillantes (*prompt injection*). Ne donne pas à un même agent la lecture de contenus publics **et** la publication sans contrôle.
4. **N'installe que des serveurs de confiance.** Un serveur MCP est un programme qui tourne avec tes droits.
5. **Moindre privilège.** Ne demande que les permissions (scopes) dont tu as besoin. Lecture seule si tu veux juste analyser.
6. **Fuite de clé ?** Révoque-la immédiatement dans le portail de la plateforme, puis nettoie.
