# Pack agent perso — « OG »

> Le petit pack promis dans l'épisode 1 **« Projets perso quand on est flemmard »** (mot-clé **OG**).
> Objectif : te donner de quoi monter une mini-équipe d'agents IA qui t'aide à mener ton projet perso jusqu'au bout, sans y laisser ton week-end ni ton compte en banque.

Tu as une idée de projet perso, mais entre coder, décider, documenter et en parler sur les réseaux, tu lâches au bout de deux semaines ? Ce pack te propose 4 briques :

| # | Brique | Dossier | En une phrase |
|---|--------|---------|---------------|
| 1 | **Automatiser** | [`roles/chef-de-projet.md`](roles/chef-de-projet.md) | Un agent « chef de projet » qui revoit les fonctionnalités, tranche les décisions et surveille le budget token. |
| 2 | **Composer l'équipe** | [`roles/`](roles/) | 4 experts : dev, architecte, fonctionnel, marketing. Chacun a sa fiche (prompt + livrables). |
| 3 | **Connecteurs MCP** | [`mcp/`](mcp/) | Un exemple de config pour que ton agent parle au monde extérieur (X/Twitter, Instagram). |
| 4 | **Budget token** | [`budget-token/`](budget-token/) | Comment contrôler tes agents avec un *harness* pour ne pas cramer tes tokens. |

---

## Avant de commencer

**Ce qu'il te faut :**

- Un outil d'agent IA (on appelle ça un **harness** : le programme qui fait tourner le modèle, lui donne des outils et le surveille). Exemples : Claude Code, Cursor, VS Code + Copilot, Hermes Agent… Les exemples de commandes du pack utilisent **Claude Code** parce que sa doc est publique et précise, mais les fiches de rôle marchent dans n'importe quel outil (c'est juste du texte).
- Un compte chez le fournisseur du modèle (abonnement ou clé API). **Les prix changent souvent : regarde la page tarifs officielle de ton fournisseur, on ne met aucun chiffre ici.**
- Git installé, et idéalement un compte GitHub.
- Pour la partie MCP : Node.js (pour la commande `npx`).

**Ce que ce pack n'est pas :** un logiciel. Ce sont des fiches, des prompts et des exemples de config à copier-coller et à adapter. Rien ne s'installe tout seul.

---

## Pas à pas

### Étape 0 — Récupère le pack

```bash
git clone https://github.com/[ton-compte]/pack-agent-perso.git
cd pack-agent-perso
```

(Ou clique sur **Use this template** sur GitHub si l'option est activée sur le repo — *à vérifier*.)

### Étape 1 — Écris ta fiche projet (10 min, pas plus)

Crée un fichier `PROJET.md` à la racine de **ton** projet avec :

```markdown
# [Nom de ton projet]
- Le problème : [qui a quel problème ?]
- La solution en une phrase : [...]
- Ce que je veux avoir dans 4 semaines : [...]
- Ce que je ne ferai PAS (hors périmètre) : [...]
- Mon budget token max par semaine : [montant ou nombre de tokens]
- Mes contraintes : [temps dispo, techno imposée, etc.]
```

Tous les agents vont lire ce fichier. Plus il est clair, moins ils partent en freestyle (et moins tu paies).

### Étape 2 — Installe le chef de projet

Lis [`roles/chef-de-projet.md`](roles/chef-de-projet.md). C'est **le** rôle à installer en premier : il reçoit tes idées, découpe le travail, dit « non » aux features inutiles et garde un œil sur la conso.

Deux façons de l'utiliser :

1. **Simple (n'importe quel outil)** : copie le bloc « Prompt » de la fiche en début de conversation.
2. **Claude Code** : copie le bloc « Version sous-agent » dans `.claude/agents/chef-de-projet.md` dans ton projet. Claude Code le détecte et peut lui déléguer du travail.

### Étape 3 — Ajoute les experts dont tu as besoin

Pas besoin des 4 dès le jour 1. Ordre conseillé :

1. [`expert-fonctionnel`](roles/expert-fonctionnel.md) — pour transformer ton idée en user stories claires.
2. [`expert-architecte`](roles/expert-architecte.md) — pour choisir une structure simple avant de coder.
3. [`expert-dev`](roles/expert-dev.md) — pour coder ticket par ticket, avec des tests.
4. [`expert-marketing`](roles/expert-marketing.md) — quand tu as quelque chose à montrer.

Le chef de projet distribue le travail entre eux. Toi, tu valides.

### Étape 4 — Branche un connecteur MCP (optionnel)

Lis [`mcp/README.md`](mcp/README.md) : c'est quoi MCP, puis un exemple pour X/Twitter (serveur officiel d'X) et Instagram (modèle générique à compléter).
**Règle d'or : aucune clé dans un fichier commité.** Tout passe par des variables d'environnement (voir [`mcp/.env.example`](mcp/.env.example)).

### Étape 5 — Pose ton budget token AVANT de lancer les agents

Lis [`budget-token/README.md`](budget-token/README.md). Minimum vital :

- un plafond de dépense côté fournisseur (si ton offre le permet) ;
- une limite de tours / de budget pour les agents qui tournent sans toi ;
- un coup d'œil à ta conso chaque semaine.

### Étape 6 — Le rituel hebdo (15 min)

Chaque semaine, demande au chef de projet :

> « Fais la revue de la semaine : ce qui est fini, ce qui bloque, les décisions à prendre, la conso de tokens vs le budget, et les 3 priorités de la semaine prochaine. »

Tu tranches, il exécute. C'est tout.

---

## Arborescence

```
pack-agent-perso/
├── README.md                 ← tu es ici
├── LICENSE                   ← licence à valider
├── roles/
│   ├── chef-de-projet.md
│   ├── expert-dev.md
│   ├── expert-architecte.md
│   ├── expert-fonctionnel.md
│   └── expert-marketing.md
├── mcp/
│   ├── README.md             ← MCP expliqué + mode d'emploi
│   ├── mcp.example.json      ← exemple de config (placeholders)
│   └── .env.example          ← variables à remplir (jamais commitées)
└── budget-token/
    └── README.md             ← contrôler les agents avec un harness
```

## Sécurité, en 4 lignes

- Ne commite **jamais** de clé API, token ou mot de passe. Le `.gitignore` du pack ignore déjà `.env`.
- Un agent avec un connecteur réseau social peut **publier en ton nom** : garde une validation humaine avant chaque publication.
- N'installe que des serveurs MCP dont tu connais la source (lis le code ou au moins le README et les permissions demandées).
- Si une clé fuit : révoque-la d'abord, nettoie ensuite.

## Points « à vérifier »

Ce pack ne contient volontairement aucun prix ni chiffre de quota. Tout ce qui est marqué *à vérifier* dépend de ton outil, de ta version ou de ton offre : contrôle sur la doc officielle avant de t'y fier.

## Licence

⚠️ **Licence à valider.** Le fichier [`LICENSE`](LICENSE) est un placeholder : la licence définitive n'est pas encore choisie. En attendant, considère que la réutilisation n'est pas encore officiellement autorisée.
