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

Utilisation :

- Appels à l'API Claude en HTTP direct depuis des Edge Functions Supabase. Opus pour la
  rédaction des questions, Sonnet pour les tâches courtes. Batch API pour les traitements
  différés. Structured Outputs sur `question-v2.schema.json` : la réponse du modèle est
  validée contre le schéma avant insertion en base.
- Un prompt existe en trois exemplaires tenus synchrones : le fichier Markdown, une constante
  de repli dans l'Edge Function, et la ligne active de la table `prompt_templates` (une par
  sport, publiée par une RPC admin). Toute modification incrémente la version et ajoute une
  ligne au changelog en bas du fichier.
- Placeholders remplis par l'Edge Function avant l'appel : `{{sport_slug}}`,
  `{{difficulty}}`, `{{count}}`, `{{theme_directive}}` (thème imposé) et
  `{{exclusion_block}}` (énoncés déjà en base sur le même thème, jusqu'à 120).
- Les règles éditoriales sont dans le prompt : ton, répartition hommes / femmes, propriété
  intellectuelle (marques, symboles olympiques), 7 à 10 mauvaises réponses dont 1 à 3
  plausibles, exclusion des paris sportifs et de la vie privée.

## `agents/`

Huit sous-agents Claude Code (frontmatter `name`, `description`, `tools`, `model`). Un skill
d'orchestration les lance tous les matins à 08h00 (launchd, mode headless).

| Agent | Rôle |
|---|---|
| `moisson-<sport>` (×7) | relève l'actualité de la veille d'un sport et rend des faits sourcés, classés `evergreen` / `one_shot` / `incertain` / `rejet` / `case_vide`, en JSON. Ne rédige pas de question. |
| `moisson-lecteur` | vérifie un lot de questions rédigées : cohérence énoncé / réponse / explication, bonne réponse confirmée par deux sources indépendantes, mauvaises réponses effectivement fausses, durée de validité, français. Verdict `approve` / `reject` / `unsure` avec confiance et sources. |

Fonctionnement :

- Entrées nommées dans le message de lancement (`date_veille`, `fenetre_app`, `budget`,
  `deja_traites`), sortie JSON au format fixé dans le fichier.
- Plafond de recherches web par agent : 14 pour la veille, 20 par lot de lecture. Minimum une
  recherche : la mémoire du modèle n'est pas acceptée comme source.
- Deux lecteurs par lot, lancés séparément ; aucun ne reçoit le classement de durabilité ni le
  verdict de l'autre. Un désaccord donne `unsure`.
- Un `unsure` n'est jamais appliqué automatiquement. Une erreur corrigeable donne `unsure` +
  correction, pas `reject`. L'activation automatique demande deux `approve`, une confiance
  ≥ 0,80 et deux sources ; le reste part en revue manuelle.

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
