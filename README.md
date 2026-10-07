# CLATCH · prompts et agents

Extraits du dépôt privé de CLATCH, quiz sportif disponible sur iOS et Android. Le Daily se
joue sans compte : [clatch-app.com/daily](https://clatch-app.com/daily).

Les questions de l'app sont produites chaque matin par une chaîne automatisée : des agents
relèvent l'actualité sportive de la veille, un modèle rédige les questions, deux lecteurs
indépendants les vérifient, celles qui passent le seuil sont activées. Ce dépôt contient les
prompts et les définitions d'agents de cette chaîne.

```
prompts/   prompts de génération de questions, versionnés, et leur JSON Schema
agents/    les 8 sous-agents Claude Code du pipeline quotidien
```

## `prompts/`

| Fichier | Rôle |
|---|---|
| `question_gen_v3.md` | prompt foot en vigueur : thème imposé + liste d'exclusion |
| `question_gen_v2.md` | version précédente, conservée avec son changelog (v1.0 → v2.9) |
| `question_gen_<sport>_v1.md` | un prompt par sport : athlétisme, basket, F1, rugby, tennis, vélo |
| `question-v2.schema.json` | JSON Schema imposé au modèle |

Utilisation (étiquettes = domaines de la certification Claude Certified Developer,
Foundations) :

- **Applications & Integration** : appels à l'API Claude en HTTP direct depuis des Edge
  Functions Supabase ; OpenAI pour les embeddings et la modération.
- **Structured Output** : JSON Schema `question-v2.schema.json` imposé au modèle, réponse
  validée contre le schéma avant insertion en base.
- **Model Selection & Optimization** : Opus pour la rédaction des questions, Sonnet pour les
  tâches courtes ; Message Batches API pour les traitements différés (environ −70 % sur la
  rédaction éditoriale par rapport à Opus en synchrone).
- **Prompt & Context Engineering** : un prompt existe en trois exemplaires tenus synchrones,
  le fichier Markdown, une constante de repli dans l'Edge Function, et la ligne active de la
  table `prompt_templates` (une par sport, publiée par une RPC admin). Toute modification
  incrémente la version et ajoute une ligne au changelog en bas du fichier. Placeholders
  remplis avant l'appel : `{{sport_slug}}`, `{{difficulty}}`, `{{count}}`,
  `{{theme_directive}}` (thème imposé) et `{{exclusion_block}}` (énoncés déjà en base sur le
  même thème et la même difficulté, jusqu'à 250).
- Les règles éditoriales sont dans le prompt : ton, répartition hommes / femmes, propriété
  intellectuelle (marques, symboles olympiques), 7 à 10 mauvaises réponses dont 1 à 3
  plausibles, exclusion des paris sportifs et de la vie privée.

Retours d'expérience :

- **Prompt & Context Engineering** : few-shot (une question modèle dans le prompt) plutôt
  que règles vagues ; contexte borné à la tâche (250 énoncés ciblés, 500 caractères
  d'instruction libre).
- **Model Selection & Optimization** : extended thinking coupé sur les tâches courtes pour
  tenir sous 30 secondes ; tout ce qui peut attendre passe en batch.
- **Structured Output** : schéma + validation avant insertion, donc pas de parsing ni de champ
  manquant.

## `agents/`

Huit sous-agents Claude Code (frontmatter `name`, `description`, `tools`, `model`). Un skill
d'orchestration les lance tous les matins à 08h00 (launchd, mode headless).

| Agent | Rôle |
|---|---|
| `moisson-<sport>` (×7) | relève l'actualité de la veille d'un sport et rend des faits sourcés, classés `evergreen` / `one_shot` / `incertain` / `rejet` / `case_vide`, en JSON. Ne rédige pas de question. |
| `moisson-lecteur` | vérifie un lot de questions rédigées : cohérence énoncé / réponse / explication, bonne réponse confirmée par deux sources indépendantes, mauvaises réponses effectivement fausses, durée de validité, français. Verdict `approve` / `reject` / `unsure` avec confiance et sources. |

Fonctionnement :

- **Agents & Workflows** : pipeline orchestrator-workers, les 7 agents de veille en
  parallèle, puis tri, génération, dédoublonnage, deux relecteurs indépendants, activation
  automatique au-dessus du seuil, composition de la grille ; un contrôle par étape.
- **Agents & Workflows** : relecteur = evaluator indépendant (LLM-as-judge), verdict
  tri-état `approve` / `reject` / `unsure`, désaccord entre les deux relecteurs = `unsure`.
  Un `unsure` n'est jamais appliqué automatiquement ; une erreur corrigeable donne `unsure` +
  correction, pas `reject`. L'activation automatique demande deux `approve`, une confiance
  ≥ 0,80 et deux sources ; le reste part en revue manuelle.
- **Claude Code** : sous-agents déclarés en frontmatter (outils et modèle par agent), skills
  métier pour l'orchestration, run headless `claude -p` quotidien lancé par launchd, liste
  blanche d'outils, coût du run journalisé.
- **Tools & MCPs** : MCP Supabase (SQL, migrations, Edge Functions), Sentry, context7,
  codegraph ; l'Edge Function `dedup_check` est appelée comme un outil, son secret est lu
  dans le Vault.

Retours d'expérience :

- **Agents & Workflows** : budgets durs de recherches par agent (14 en veille, 20 par lot de
  relecture), comptés par l'agent, plancher à une recherche.
- **Context Management & Reliability** : un agent = un contrat (entrées nommées
  `date_veille`, `fenetre_app`, `budget`, `deja_traites` ; sortie JSON stricte) ; les
  sous-agents rendent des faits, la session d'orchestration reste légère.
- **Agents & Workflows** : relecteurs aveugles (ni le classement de durabilité, ni l'avis de
  l'autre) ; un cas concret dans la fiche agent plutôt qu'une règle générale.
- **Reliability** : contrôle de fin de run par le chemin du joueur, pas par les logs du
  pipeline.

Chaîne complète :

```
7 agents moisson-<sport> en parallèle
  → tri selon la grille éditoriale
  → rédaction des questions à partir des faits sourcés (Claude, Structured Outputs)
  → détection des doublons par embeddings (pgvector)
  → insertion en file de revue
  → 2 lecteurs indépendants par lot
  → activation automatique au-dessus du seuil, revue manuelle pour le reste
  → composition de la grille du Daily
  → contrôle final en conditions joueur
```

## Périmètre

Extraits relus avant publication : aucune clé, aucun identifiant de projet, aucune donnée
joueur. Voir `NOTICE.md`.
