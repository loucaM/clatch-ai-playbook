# CLATCH · prompts et agents

Extraits du dépôt privé de **CLATCH**, quiz sportif en production sur iOS et Android
(le Daily se joue sans compte : [clatch-app.com/daily](https://clatch-app.com/daily)).

Le contenu de l'app est produit chaque matin par un pipeline LLM : des agents balaient
l'actu de la veille, un modèle rédige les questions, d'autres agents les contre-vérifient,
et seules celles qui passent le seuil de confiance sont activées. Ce dépôt publie les deux
briques qui se lisent sans le code : les **prompts** et les **agents**.

```
prompts/   les prompts de génération de questions, versionnés, et leur JSON Schema
agents/    les 8 sous-agents Claude Code du pipeline quotidien
```

## `prompts/` : génération de questions

| Fichier | Rôle |
|---|---|
| `question_gen_v3.md` | prompt foot courant : mono-thème imposé + bloc anti-répétition |
| `question_gen_v2.md` | version précédente, conservée pour le changelog (v1.0 → v2.9) |
| `question_gen_<sport>_v1.md` | un prompt par sport : athlétisme, basket, F1, rugby, tennis, vélo |
| `question-v2.schema.json` | le JSON Schema imposé au modèle en Structured Outputs |

Comment ils sont utilisés :

- **Appel** : Claude (Opus pour les questions, Sonnet pour les tâches courtes) via la
  **Batch API** pour tout ce qui n'est pas temps réel, en **Structured Outputs** sur le
  schéma ci-dessus. La sortie est typée et validée avant d'entrer en base.
- **Versionnés en trois endroits synchronisés** : le fichier Markdown (source de vérité
  humaine), une constante de repli dans l'Edge Function, et la ligne active de la table
  `prompt_templates` (une seule active par sport, publiée par une RPC admin). Un ajustement
  = bump de version + ligne de changelog en bas du fichier.
- **Placeholders remplis au runtime** par l'Edge Function : `{{sport_slug}}`,
  `{{difficulty}}`, `{{count}}`, `{{theme_directive}}` (thème unique imposé) et
  `{{exclusion_block}}` (jusqu'à 120 énoncés déjà en base, pour ne pas se répéter).
- **Doctrine éditoriale dans le prompt** : ton chambreur, politique H/F, propriété
  intellectuelle (marques, anneaux olympiques), pool de mauvaises réponses 7-10 avec
  « dangers », pas de paris sportifs, pas de vie privée.

## `agents/` : le pipeline quotidien

Huit sous-agents Claude Code (frontmatter `name` / `description` / `tools` / `model`),
lancés par un skill d'orchestration tous les matins à 08h00 en mode headless.

| Agent | Rôle |
|---|---|
| `moisson-<sport>` (×7) | balayer l'actu de la veille d'un sport, en parallèle, et rendre des **faits sourcés et pré-classés** (`evergreen` / `one_shot` / `incertain` / `rejet` / `case_vide`) en JSON strict, jamais une question rédigée |
| `moisson-lecteur` | contre-vérifier un lot de questions rédigées : cohérence interne, bonne réponse confirmée par **deux sources indépendantes**, mauvaises réponses vraiment fausses, périssabilité, français. Verdict tri-état `approve` / `reject` / `unsure` avec confiance, checks et sources |

Ce que ces fichiers montrent :

- **Un agent = un contrat**. Entrées nommées (`date_veille`, `fenetre_app`, `budget`,
  `deja_traites`), sortie JSON stricte, règles de datation non négociables.
- **Budget dur de recherches** (14 par agent de veille, 20 par lot de lecture), compté à voix
  haute. Dépasser n'est pas du zèle, c'est un bug ; et un plancher : jamais zéro recherche,
  « ta mémoire n'est pas une source ».
- **Indépendance des lecteurs** : deux instances par lot, aucune ne voit le tag de durabilité
  ni le verdict de l'autre. Un désaccord vaut `unsure`.
- **Principe du doute** : seule la certitude agit. `unsure` n'est jamais appliqué, une
  erreur réparable donne `unsure` + correction, jamais `reject`. L'activation automatique
  exige deux `approve`, une confiance ≥ 0,80 et au moins deux sources ; tout le reste
  attend la revue humaine.

Le pipeline complet, pour situer les agents :

```
7 agents moisson-<sport> en parallèle
  → tri (grille éditoriale)
  → génération ancrée sur les faits sourcés (Claude, Structured Outputs)
  → dédoublonnage par embeddings (pgvector)
  → insertion en file de revue
  → 2 lecteurs indépendants par lot
  → activation automatique au-dessus du seuil, le reste à la revue humaine
  → composition de la grille du Daily
  → contrôle final par le chemin du joueur
```

## Ce que ça nous a appris

- Un exemple pèse plus qu'une consigne.
- N'injecter que le contexte utile ; le reste coûte et dilue.
- Les budgets sont des plafonds durs, sinon les agents les dépassent pour prouver du vide.
- Un verdict en trois états, avec le doute qui remonte à l'humain, vaut mieux qu'un oui/non.
- Le prompt est du code : versionné, changelog, une seule version active, repli en cas de
  panne.

## Périmètre

Extraits sélectionnés et relus avant publication : aucune clé, aucun identifiant de projet,
aucune donnée joueur. Voir `NOTICE.md`.
