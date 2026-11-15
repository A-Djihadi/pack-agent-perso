# Budget token : contrôler tes agents avec un harness

## Le problème

Un agent IA, ça consomme des **tokens** (des morceaux de texte) à chaque échange : ce que tu envoies (ton message + tout le contexte : fichiers lus, historique, résultats d'outils) **et** ce qu'il répond. Plus la conversation est longue, plus chaque nouveau message coûte cher, parce que tout l'historique est renvoyé.

Avec plusieurs agents (chef de projet + 4 experts) et des connecteurs MCP, la facture peut grimper vite sans que tu t'en rendes compte. D'où l'idée : **contrôler les agents avec un harness**.

## C'est quoi un harness ?

Le **harness** (le « harnais »), c'est le programme qui fait tourner le modèle : il lui donne des outils, gère le contexte, demande tes permissions et peut **mettre des limites**. Claude Code, Cursor, Hermes Agent, etc. sont des harness.

Le modèle ne sait pas combien tu as dépensé. **C'est le harness qui peut compter et couper.** Ton job : configurer ces garde-fous avant de lâcher les agents.

## Les 3 niveaux de contrôle

| Niveau | Question | Outil |
|--------|----------|-------|
| 1. Plafond fournisseur | « Combien je peux perdre au pire ce mois-ci ? » | Limite de dépense dans ton compte (console / paramètres de ton offre) |
| 2. Limite par exécution | « Combien un agent peut dépenser sur UNE tâche ? » | Options du harness (nombre de tours, budget max) |
| 3. Hygiène quotidienne | « Comment éviter de gaspiller ? » | Contexte court, bon modèle, moins d'outils chargés |

---

## Niveau 1 — Le plafond chez le fournisseur

C'est le filet de sécurité ultime : même si un agent part en boucle, il s'arrête au plafond.

Exemple Anthropic (d'après la [doc coûts de Claude Code](https://code.claude.com/docs/en/costs)) :

- **Compte API (Claude Console)** : tu peux fixer des **limites de dépense par workspace** dans la Console.
- **Abonnement individuel** : si les crédits d'usage supplémentaires sont activés, tu peux fixer une **limite de dépense mensuelle** dans les paramètres d'usage de ton compte claude.ai.

Chez les autres fournisseurs, cherche « spend limit », « budget » ou « usage limit » dans la doc. Les options et les prix changent : **vérifie sur la page officielle, on ne donne aucun chiffre ici.**

## Niveau 2 — Limiter chaque exécution (exemples Claude Code)

Les options ci-dessous sont documentées dans la [référence CLI de Claude Code](https://code.claude.com/docs/en/cli-reference). Elles marchent en **mode non interactif** (`-p`, quand l'agent tourne sans toi, par exemple dans un script).

```bash
# Limiter le nombre de tours de l'agent (pas de limite par défaut)
claude -p --max-turns 5 "Fais la revue hebdo à partir de docs/"

# Arrêter l'exécution quand la dépense ESTIMÉE atteint un montant (en dollars)
claude -p --max-budget-usd [montant] "Implémente le ticket 12"

# Choisir un modèle moins cher pour une tâche simple
claude -p --model haiku "Résume docs/decisions.md en 5 points"

# Ne charger QUE les serveurs MCP nécessaires (moins d'outils = moins de contexte)
claude -p --strict-mcp-config --mcp-config ./mcp-lecture-seule.json "Liste mes mentions X"
```

À savoir sur `--max-budget-usd` (selon la doc) :

- c'est une **estimation** calculée côté client, qui peut différer de ta facture ;
- la dépense **peut dépasser** le plafond : garde de la marge ;
- la dépense des sous-agents compte dans le plafond.

### Exemple : script de revue hebdo bridé

`revue-hebdo.sh` (à adapter) :

```bash
#!/usr/bin/env bash
set -euo pipefail

# Garde-fous : à ajuster selon TON budget dans PROJET.md
MAX_TURNS=10
MAX_BUDGET_USD="[montant]"

claude -p \
  --agent chef-de-projet \
  --model sonnet \
  --max-turns "$MAX_TURNS" \
  --max-budget-usd "$MAX_BUDGET_USD" \
  "Revue hebdo : lis PROJET.md et docs/, puis écris docs/revues/$(date +%F).md (fait / bloqué / décisions / 3 priorités)."
```

`--agent chef-de-projet` suppose que tu as créé `.claude/agents/chef-de-projet.md` (voir [`../roles/chef-de-projet.md`](../roles/chef-de-projet.md)). Les alias de modèle disponibles (`sonnet`, `haiku`, etc.) dépendent de ta version : *à vérifier* avec `claude --help`.

### Voir ce que tu as consommé

- Dans Claude Code, la commande **`/usage`** affiche l'usage de tokens de la session et un **coût estimé** (calculé au prix public, ce n'est pas une facture). Les abonnés y voient aussi leurs barres d'usage du forfait.
- Pour la facture réelle : la page de coûts de ta console / de ton compte fournisseur.

### Et dans les autres harness ?

Le principe est le même partout : chercher dans la doc une **limite de tours / d'itérations**, une **limite de budget**, et une **commande d'usage**. Exemple : Hermes Agent documente un réglage `agent.max_turns` dans sa config et des commandes `/usage` et `/insights` (*à vérifier selon ta version* sur [la doc Hermes](https://hermes-agent.nousresearch.com/docs)).

## Niveau 3 — Hygiène : dépenser moins sans travailler moins

Conseils tirés de la [doc coûts de Claude Code](https://code.claude.com/docs/en/costs), valables dans l'esprit pour tous les outils :

1. **Repars de zéro entre deux tâches sans rapport** (`/clear` dans Claude Code). Un vieux contexte se repaie à chaque message.
2. **Résume quand c'est long** (`/compact` + ce qu'il faut garder, ex. `/compact garde les décisions et la liste des tickets`). Attention : résumer un très gros contexte est lui-même une grosse requête.
3. **Le bon modèle pour la bonne tâche** : modèle léger pour résumer / reformuler, modèle costaud seulement pour l'architecture ou les bugs tordus (`/model`, ou `model:` dans la fiche du sous-agent).
4. **Baisse l'effort de réflexion** sur les tâches simples (`/effort` dans Claude Code), la réflexion étendue étant facturée comme des tokens de sortie.
5. **Isole les tâches bavardes** (logs, tests, recherche de doc) dans un sous-agent : seul le résumé revient dans ta conversation principale. Le sous-agent consomme quand même : donne-lui un modèle léger.
6. **Charge moins de serveurs MCP** : chaque serveur ajoute des descriptions d'outils au contexte. Désactive ceux dont tu ne te sers pas.
7. **Des fichiers de contexte courts** (`PROJET.md`, `CLAUDE.md`…) : ils sont relus souvent.
8. **Attention aux agents qui tournent tout seuls** (tâches planifiées, boucles, agents en arrière-plan) : ils consomment même quand tu ne regardes pas.

## La routine budget (5 minutes par semaine)

1. Regarde ta conso de la semaine (console fournisseur ou commande d'usage).
2. Donne le chiffre au chef de projet pendant la revue hebdo.
3. S'il dépasse 50 % du budget mensuel à mi-mois : il propose des économies (modèle, contexte, moins d'automatismes).
4. Ajuste `MAX_TURNS` / `MAX_BUDGET_USD` dans tes scripts.

## Checklist avant de lancer un agent en autonomie

- [ ] Plafond de dépense configuré chez le fournisseur
- [ ] `--max-turns` (ou équivalent) défini
- [ ] `--max-budget-usd` (ou équivalent) défini, avec de la marge
- [ ] Seuls les serveurs MCP nécessaires sont chargés
- [ ] Aucun outil de publication / suppression sans validation humaine
- [ ] Tu sais où regarder la conso
