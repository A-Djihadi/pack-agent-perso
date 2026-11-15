# Pack agent perso (mot-clé OG)

> C'est le petit pack promis dans l'épisode 1, « Projets perso quand on est flemmard ».
> Le but : monter une mini-équipe d'agents IA qui t'aide à finir ton projet perso, sans y laisser ton week-end ni ton compte en banque.

Tu as une idée de projet perso. Mais entre coder, décider, documenter et en parler sur les réseaux, tu lâches au bout de deux semaines ? Le pack tient en 4 briques :

| # | Brique | Dossier | En une phrase |
|---|--------|---------|---------------|
| 1 | Automatiser | [`roles/chef-de-projet.md`](roles/chef-de-projet.md) | Un agent « chef de projet » revoit les fonctionnalités, tranche les décisions et surveille le budget token. |
| 2 | Composer l'équipe | [`roles/`](roles/) | 4 experts : dev, architecte, fonctionnel, marketing. Chacun a sa fiche avec un prompt et ses livrables. |
| 3 | Connecteurs MCP | [`mcp/`](mcp/) | Un exemple de config pour que ton agent parle au monde extérieur (X/Twitter, Instagram). |
| 4 | Budget token | [`budget-token/`](budget-token/) | Comment contrôler tes agents avec un *harness* pour ne pas cramer tes tokens. |

---

## Avant de commencer

Il te faut :

- Un outil d'agent IA. On appelle ça un harness : le programme qui fait tourner le modèle, lui donne des outils et le surveille. Par exemple Claude Code, Cursor, VS Code avec Copilot ou Hermes Agent. Les commandes du pack utilisent Claude Code parce que sa doc est publique et précise. Les fiches de rôle, elles, marchent dans n'importe quel outil : c'est juste du texte.
- Un compte chez le fournisseur du modèle (abonnement ou clé API). Les prix changent souvent. Regarde la page tarifs officielle de ton fournisseur, on ne met aucun chiffre ici.
- Git installé, et idéalement un compte GitHub.
- Node.js pour la partie MCP (la commande `npx` en a besoin).

Ce pack n'est pas un logiciel. Ce sont des fiches, des prompts et des exemples de config à copier-coller puis adapter. Rien ne s'installe tout seul.

---

## Pas à pas

### Étape 0 : récupère le pack

```bash
git clone https://github.com/A-Djihadi/pack-agent-perso.git
cd pack-agent-perso
```

Tu peux aussi cliquer sur « Use this template » sur GitHub, si l'option est activée sur le repo (*à vérifier*).

### Étape 1 : écris ta fiche projet (10 min, pas plus)

Crée un fichier `PROJET.md` à la racine de ton projet avec :

```markdown
# [Nom de ton projet]
- Le problème : [qui a quel problème ?]
- La solution en une phrase : [...]
- Ce que je veux avoir dans 4 semaines : [...]
- Ce que je ne ferai PAS (hors périmètre) : [...]
- Mon budget token max par semaine : [montant ou nombre de tokens]
- Mes contraintes : [temps dispo, techno imposée...]
```

Tous les agents vont lire ce fichier. Plus il est clair, moins ils partent en freestyle, et moins tu paies.

### Étape 2 : installe le chef de projet

Lis [`roles/chef-de-projet.md`](roles/chef-de-projet.md). C'est le rôle à installer en premier. Il reçoit tes idées, découpe le travail, dit non aux features inutiles et garde un œil sur la conso.

Tu as deux façons de l'utiliser :

1. Avec n'importe quel outil, copie le bloc « Prompt » de la fiche en début de conversation.
2. Avec Claude Code, copie le bloc « Version sous-agent » dans `.claude/agents/chef-de-projet.md` dans ton projet. Claude Code le détecte et peut lui déléguer du travail.

### Étape 3 : ajoute les experts dont tu as besoin

Pas besoin des 4 dès le premier jour. Voici l'ordre conseillé :

1. [`expert-fonctionnel`](roles/expert-fonctionnel.md) transforme ton idée en user stories claires.
2. [`expert-architecte`](roles/expert-architecte.md) choisit une structure simple avant que tu codes.
3. [`expert-dev`](roles/expert-dev.md) code ticket par ticket, avec des tests.
4. [`expert-marketing`](roles/expert-marketing.md) entre en jeu quand tu as quelque chose à montrer.

Le chef de projet distribue le travail entre eux. Toi, tu valides.

### Étape 4 : branche un connecteur MCP (optionnel)

Lis [`mcp/README.md`](mcp/README.md). Tu y trouveras ce qu'est MCP, puis un exemple pour X/Twitter (serveur officiel d'X) et un pour Instagram (modèle générique à compléter).
Une règle ne bouge pas : aucune clé dans un fichier commité. Tout passe par des variables d'environnement (voir [`mcp/.env.example`](mcp/.env.example)).

### Étape 5 : pose ton budget token AVANT de lancer les agents

Lis [`budget-token/README.md`](budget-token/README.md). Le minimum :

- un plafond de dépense côté fournisseur, si ton offre le permet ;
- une limite de tours ou de budget pour les agents qui tournent sans toi ;
- un coup d'œil à ta conso chaque semaine.

### Étape 6 : le rituel hebdo (15 min)

Chaque semaine, demande au chef de projet :

> « Fais la revue de la semaine : ce qui est fini, ce qui bloque, les décisions à prendre, la conso de tokens vs le budget, et les 3 priorités de la semaine prochaine. »

Tu tranches, il exécute. C'est tout.

---

## Arborescence

```
pack-agent-perso/
├── README.md                 ← tu es ici
├── LICENSE                   ← Licence : CC-BY-4.0 (attribution Djihadi Ahamdy)
├── roles/
│   ├── chef-de-projet.md
│   ├── expert-dev.md
│   ├── expert-architecte.md
│   ├── expert-fonctionnel.md
│   └── expert-marketing.md
├── mcp/
│   ├── README.md             ← MCP expliqué + mode d'emploi
│   ├── mcp.example.json      ← exemple de config (valeurs à remplacer)
│   └── .env.example          ← variables à remplir (jamais commitées)
└── budget-token/
    └── README.md             ← contrôler les agents avec un harness
```

## Sécurité, en 4 lignes

- Ne commite jamais de clé API, de token ou de mot de passe. Le `.gitignore` du pack ignore déjà `.env`.
- Un agent branché sur un réseau social peut publier en ton nom. Garde une validation humaine avant chaque publication.
- N'installe que des serveurs MCP dont tu connais la source. Lis le code, ou au moins le README et les permissions demandées.
- Si une clé fuit, révoque-la d'abord et nettoie ensuite.

## Points « à vérifier »

Ce pack ne contient volontairement aucun prix ni chiffre de quota. Tout ce qui est marqué *à vérifier* dépend de ton outil, de ta version ou de ton offre. Contrôle sur la doc officielle avant de t'y fier.

## Licence

Licence : CC-BY-4.0 (attribution Djihadi Ahamdy). Le texte complet est dans le fichier [`LICENSE`](LICENSE).
