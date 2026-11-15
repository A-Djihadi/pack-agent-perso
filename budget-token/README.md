# Budget token : contrôler tes agents avec un harness

## Le problème

Un agent IA consomme des tokens (des morceaux de texte) à chaque échange. Ça compte dans les deux sens : ce que tu envoies et ce qu'il répond. Ce que tu envoies, c'est ton message plus tout le contexte, donc les fichiers lus, l'historique et les résultats d'outils. Plus la conversation est longue, plus chaque nouveau message coûte cher, parce que tout l'historique est renvoyé.

Avec plusieurs agents (le chef de projet et 4 experts) et des connecteurs MCP, la facture peut grimper vite sans que tu t'en rendes compte. D'où l'idée de contrôler les agents avec un harness.

## C'est quoi un harness ?

Le harness (le « harnais »), c'est le programme qui fait tourner le modèle. Il lui donne des outils, gère le contexte, demande tes permissions et peut mettre des limites. Claude Code, Cursor ou Hermes Agent sont des harness.

Le modèle ne sait pas combien tu as dépensé. C'est le harness qui peut compter et couper. À toi de configurer ces garde-fous avant de lâcher les agents.

## Les 3 niveaux de contrôle

| Niveau | Question | Outil |
|--------|----------|-------|
| 1. Plafond fournisseur | « Combien je peux perdre au pire ce mois-ci ? » | Limite de dépense dans ton compte (console ou paramètres de ton offre) |
| 2. Limite par exécution | « Combien un agent peut dépenser sur UNE tâche ? » | Options du harness (nombre de tours, budget max) |
| 3. Hygiène quotidienne | « Comment éviter de gaspiller ? » | Contexte court, bon modèle, moins d'outils chargés |

---

## Niveau 1 : le plafond chez le fournisseur

C'est ton dernier filet de sécurité. Même si un agent part en boucle, il s'arrête au plafond.

Exemple chez Anthropic, d'après la [doc coûts de Claude Code](https://code.claude.com/docs/en/costs) :

- Avec un compte API (Claude Console), tu peux fixer des limites de dépense par workspace dans la Console.
- Avec un abonnement individuel, si les crédits d'usage supplémentaires sont activés, tu peux fixer une limite de dépense mensuelle dans les paramètres d'usage de ton compte claude.ai.

Chez les autres fournisseurs, cherche « spend limit », « budget » ou « usage limit » dans la doc. Les options et les prix changent. Vérifie sur la page officielle, on ne donne aucun chiffre ici.

## Niveau 2 : limiter chaque exécution (exemples Claude Code)

Les options ci-dessous sont documentées dans la [référence CLI de Claude Code](https://code.claude.com/docs/en/cli-reference). Elles marchent en mode non interactif (`-p`), quand l'agent tourne sans toi, par exemple dans un script.

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

Ce que dit la doc sur `--max-budget-usd` :

- c'est une estimation calculée côté client, elle peut différer de ta facture ;
- la dépense peut dépasser le plafond, donc garde de la marge ;
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

`--agent chef-de-projet` suppose que tu as créé `.claude/agents/chef-de-projet.md` (voir [`../roles/chef-de-projet.md`](../roles/chef-de-projet.md)). Les alias de modèle disponibles, comme `sonnet` ou `haiku`, dépendent de ta version. C'est *à vérifier* avec `claude --help`.

### Voir ce que tu as consommé

- Dans Claude Code, la commande `/usage` affiche l'usage de tokens de la session et un coût estimé. Ce coût est calculé au prix public, ce n'est pas une facture. Les abonnés y voient aussi leurs barres d'usage du forfait.
- Pour la facture réelle, va sur la page de coûts de ta console ou de ton compte fournisseur.

### Et dans les autres harness ?

Le principe ne change pas. Cherche dans la doc une limite de tours ou d'itérations, une limite de budget et une commande d'usage. Par exemple, Hermes Agent documente un réglage `agent.max_turns` dans sa config et des commandes `/usage` et `/insights` (*à vérifier selon ta version* sur [la doc Hermes](https://hermes-agent.nousresearch.com/docs)).

## Niveau 3 : hygiène, ou comment dépenser moins sans travailler moins

Ces conseils viennent de la [doc coûts de Claude Code](https://code.claude.com/docs/en/costs). L'idée vaut pour tous les outils.

1. Repars de zéro entre deux tâches sans rapport (`/clear` dans Claude Code). Un vieux contexte se repaie à chaque message.
2. Résume quand ça devient long. Utilise `/compact` en précisant ce qu'il faut garder, par exemple `/compact garde les décisions et la liste des tickets`. Attention, résumer un très gros contexte est lui-même une grosse requête.
3. Prends le bon modèle pour chaque tâche. Un modèle léger suffit pour résumer ou reformuler. Garde le modèle costaud pour l'architecture ou les bugs tordus (`/model`, ou `model:` dans la fiche du sous-agent).
4. Baisse l'effort de réflexion sur les tâches simples (`/effort` dans Claude Code). La réflexion étendue est facturée comme des tokens de sortie.
5. Isole les tâches bavardes (logs, tests, recherche de doc) dans un sous-agent. Seul le résumé revient dans ta conversation principale. Le sous-agent consomme quand même, alors donne-lui un modèle léger.
6. Charge moins de serveurs MCP. Chaque serveur ajoute des descriptions d'outils au contexte. Désactive ceux dont tu ne te sers pas.
7. Garde tes fichiers de contexte courts (`PROJET.md`, `CLAUDE.md`...). Ils sont relus souvent.
8. Méfie-toi des agents qui tournent tout seuls (tâches planifiées, boucles, agents en arrière-plan). Ils consomment même quand tu ne regardes pas.

## La routine budget (5 minutes par semaine)

1. Regarde ta conso de la semaine (console fournisseur ou commande d'usage).
2. Donne le chiffre au chef de projet pendant la revue hebdo.
3. Si tu dépasses 50 % du budget mensuel à mi-mois, il propose des économies (modèle, contexte, moins d'automatismes).
4. Ajuste `MAX_TURNS` et `MAX_BUDGET_USD` dans tes scripts.

## Checklist avant de lancer un agent en autonomie

- [ ] Plafond de dépense configuré chez le fournisseur
- [ ] `--max-turns` (ou équivalent) défini
- [ ] `--max-budget-usd` (ou équivalent) défini, avec de la marge
- [ ] Seuls les serveurs MCP nécessaires sont chargés
- [ ] Aucun outil de publication ou de suppression sans validation humaine
- [ ] Tu sais où regarder la conso
