# Prompt versionné `question_gen_f1_v1.md` (FORMULE 1)

> Source de vérité humaine pour la génération de questions **F1** via Claude (Epic F1, 2026-09-28).
> La F1 a son **propre prompt actif** en DB (`prompt_templates` sport-scopé : 1 ligne active par sport).
> **Publication** : copie le bloc « TEMPLATE À PUBLIER » ci-dessous dans `/admin/prompts` avec le **sélecteur de sport = f1**, version `v1.0`. DEV d'abord ; PROD seulement sur GO Louca.
> ✅ **Leçons rugby intégrées d'office** : le template a été relu **section par section** contre `question_gen_rugby_v1.md` v1.2 (table `wrong_answers`, intégrité de l'énoncé, Tags, exemple JSON) — l'oubli de la v1.0 rugby (15/15 rejetées à 3 distracteurs) ne peut pas se reproduire ici. L'exemple JSON est **accentué** (incident v1.1 rugby : le modèle imite l'exemple avant d'obéir aux règles).
> ⚠️ **Prérequis** : `'f1'` dans `PROMPT_SPORT_SLUGS` (`admin/lib/schemas/create-prompt.ts`) et dans `V1_ALLOWED_SPORT_SLUGS` (`admin/app/api/admin/batches/route.ts`) — faits dans ce lot.
> ⚠️ **Prérequis EF** : `'f1'` dans le picklist Valibot **ET** l'enum JSON Schema de `mobile/supabase/functions/_shared/schema.ts` — fait dans ce lot — **et REDÉPLOYER `generate_questions_batch` / `poll_batch_results`** (incident basket 2026-08-28 : 123 orphelines, bundle resté vieux).
> ⚠️ **Comme le rugby** : la F1 passe `sports.status = 'live'` (migration 0301). Une question F1 **validée** tombe dans le pool du **Daily mixte**, `f1_enabled` OFF ou non. Le robinet, c'est la validation admin.

## Variables interpolées au runtime

Identiques aux autres sports : `{{sport_slug}}` (= `f1`), `{{difficulty}}`, `{{count}}`, `{{theme_directive}}`, `{{exclusion_block}}`.

## Thèmes disponibles (migration 0301, 8 feuilles)

`championnat-du-monde-f1` · `ecuries-f1` · `grands-prix-f1` · `francais-en-f1` · `f1-moderne` · `legendes-f1` · `records-f1` · `regles-f1`

## Contrat de sortie (caps DURS — `_shared/schema.ts`, sinon REJECTED à l'import)

- `text` : **10-140 chars** (HARD CAP mobile ; sweet spot 80-100).
- `correct_answer` : 1-80 chars, **entité unique sans parenthèses ni multi-entités**.
- `wrong_answers` : pool 7-10 distractors `{ "text" ≤80, "danger": 1|2|3 }`.
- `explanation` : 50-300 chars, ton chambreur.
- `source_url` : URI 10-500.
- `themes` : ≥1 slug (le thème imposé), jamais vide.

---

## TEMPLATE À PUBLIER (copier tout ce qui suit dans /admin/prompts, sport = f1)

# Tâche — F1 v1.0

Génère **{{count}} questions** au format **JSON strict** spécifié plus bas, pour le sport `{{sport_slug}}` à la difficulté `{{difficulty}}`.

{{theme_directive}}
# Contexte projet

Clatch est un jeu mobile FR de quiz sportif, ton **chambreur (humour second degré)**. Cette série porte sur la **FORMULE 1** (Championnat du Monde depuis 1950, pilotes et constructeurs, écuries, Grands Prix et circuits, Français en F1, règlement). Format : `correct_answer + wrong_answers pool` (anti-mémorisation + danger calibré + tagging thématique).

# Règles éditoriales

1. **Ton chambreur dans `explanation`** : second degré, jamais méchant gratuit ni dégradant. Registre F1 : la pole, le tour de chauffe, l'undercut, le train de pneus, la voiture de sécurité, le drapeau à damier, le tête-à-queue, l'aspiration, le DRS, les stands, le muret, le podium au champagne.

2. **Doctrine « référence nécessaire » IP** : noms de pilotes, d'écuries, de circuits et de Grands Prix = informatif factuel autorisé (« au Grand Prix de Monaco 1984 », « avec McLaren en 1988 »). **MAIS** : n'implique JAMAIS un partenariat officiel avec la FIA, la F1 ou une écurie, n'utilise PAS logos ni sponsors comme label commercial ; en cas de doute (sponsors titres, livrées), reste factuel ou fallback générique.

3. **Angle féminin (rare mais réel)** : la F1 compte très peu de femmes en course — ne force PAS un quota, mais mobilise les faits vrais et marquants : Maria Teresa de Filippis (première femme au départ d'un Grand Prix, 1958), Lella Lombardi (seule femme à avoir marqué des points, un demi-point au GP d'Espagne 1975), Susie Wolff (essais libres en 2014-2015), Claire Williams (directrice d'écurie), la F1 Academy (depuis 2023). **Vérifie chaque fait avant de l'utiliser.**

4. **Pilotes ET constructeurs** : il existe **deux** titres mondiaux chaque saison (pilotes depuis 1950, constructeurs depuis 1958). **Précise TOUJOURS lequel dans l'énoncé** : « champion du monde des pilotes » ≠ « titre constructeurs ». Une écurie qui gagne le titre constructeurs n'a pas forcément le champion pilotes la même année.

5. **ÉCURIES : L'ANNÉE DANS L'ÉNONCÉ (RÈGLE DURE)** : les écuries changent de nom et de propriétaire. Toleman → Benetton → Renault → Lotus → Renault → Alpine ; Jordan → Midland → Spyker → Force India → Racing Point → Aston Martin ; Minardi → Toro Rosso → AlphaTauri → RB ; Sauber → BMW Sauber → Sauber → Alfa Romeo → Sauber. Toute question sur une écurie **porte l'année ou la saison**, sinon la réponse peut devenir fausse ou double.

6. **Difficulté calibrée** :
   - **bronze** : faits ultra-connus, fan occasionnel. Varie les angles (nationalité d'Ayrton Senna, couleur historique des Ferrari, principauté qui accueille un GP en ville, surnom d'Alain Prost « le Professeur », signification du drapeau à damier, écurie de Lewis Hamilton lors d'un titre donné).
   - **argent** : fan averti. Angles subtils : champion d'une saison précise, circuit d'un GP historique, vainqueur d'un GP de France précis, coéquipier d'un champion une année donnée, nombre de titres d'un pilote au terme de sa carrière.
   - **or** : érudit/expert. Marques exactes et éditions anciennes (les cinq titres de Fangio, le titre posthume de Jochen Rindt en 1970, la première victoire d'une écurie, la saison 1950), pilotes hors stars, détails **règlement** (barèmes de points selon les époques, le « Grand Chelem » pole + victoire + meilleur tour + tête de bout en bout, drapeaux bleu/jaune/rouge, courses sprint depuis 2021).

7. **ANTI-RÉPÉTITION / VARIÉTÉ OBLIGATOIRE** : ne ressors PAS que Senna, Schumacher, Hamilton et Verstappen. Le bloc « Questions DÉJÀ en base » fourni plus bas liste ce qui existe : génère du contenu **strictement nouveau** (un même record/pilote/saison sous un autre angle compte comme un doublon). **Fais TOURNER les axes sur l'ensemble du batch** :
   - **Époques** : les pionniers (1950-1967), l'ère Cosworth et des garagistes, les turbos des années 80, Prost-Senna, l'ère Schumacher-Ferrari, Alonso et Vettel, l'ère hybride depuis 2014. Vise ≤ 40 % d'ère hybride sur un batch générique.
   - **Géographie** : ne surpondère pas la France et le Royaume-Uni — mobilise l'Italie, l'Allemagne, le Brésil, l'Argentine, la Finlande, l'Autriche, l'Australie, les Pays-Bas, le Mexique, le Japon, les GP d'Amérique, d'Asie et du Moyen-Orient.
   - **Angles rares à privilégier** : les Français (Prost, Arnoux, Laffite, Pironi, Tambay, Jabouille et la première victoire turbo, Panis à Monaco 1996, Alesi à Montréal 1995, Gasly à Monza 2020, Ocon en Hongrie 2021), Ligier, Matra, les circuits disparus (Reims, Rouen, Clermont-Ferrand), l'écurie Brawn GP.

8. **ANTI-PÉRISSABLE (CRITIQUE)** : bannis tout superlatif battable ou ancré au présent (« le champion EN TITRE », « le recordman ACTUEL », « le pilote de telle écurie »). **Ancre TOUJOURS dans le temps** : « au terme de la saison 2020 », « au Grand Prix d'Italie 2020 », « lors de son titre de 1988 ». Une question doit rester vraie dans 10 ans. **Aucun résultat d'un Grand Prix ou d'une saison non encore terminés.** Les records de victoires, poles et titres tombent : ancre-les à une date. Les barèmes de points ont changé plusieurs fois : **une question de points porte l'année**.

9. **Sources** — **Wikipédia ≤25 %**, **diversifie activement** et choisis LA PLUS PERTINENTE ; privilégie le primaire/spécialisé.
   - **Instances officielles** : **Formula1.com** ⭐ (résultats, archives par saison) · **FIA (fia.com)** ⭐ (règlement sportif et technique, classements officiels).
   - **Statistiques / archives** : **StatsF1 (statsf1.com)** ⭐ · **Forix / Autosport Forix** · **ChicaneF1** · **Motorsport Stats**.
   - **Presse spécialisée** : **Auto Hebdo** · **L'Équipe (rubrique F1)** · **Autosport** · **Motorsport.com** · **Nextgen-Auto** · **The Race**.
   - Une source par question, la plus proche du fait. Un fait de 1958 ne se source pas sur un site d'actualité de 2026.

10. **INTÉGRITÉ DE L'ÉNONCÉ (RÈGLE DURE — priorité absolue ; tout manquement rend la question INUTILISABLE)** :
   - **Vérité littérale de l'énoncé** : la `correct_answer` doit satisfaire CHAQUE prédicat de l'énoncé — nationalité, écurie, année, nombre, circuit, Grand Prix, titre pilotes ou constructeurs. S'il dit « quel Français », la réponse EST française ; s'il dit « remporte le titre », c'est bien le champion et pas le vainqueur d'un GP. **Interdiction absolue de fabriquer une fausse prémisse** pour tendre un piège.
   - **Le piège vit UNIQUEMENT dans les `wrong_answers`** (les leurres danger=3), JAMAIS dans l'énoncé ni dans la `correct_answer`.
   - **L'`explanation` CONFIRME la `correct_answer`** : elle ne nomme JAMAIS une autre entité comme vraie réponse, ne se corrige pas, ne doute pas. Mots BANNIS dans l'explication : « piège », « à reformuler », « mea culpa », « en fait c'est… », « oups », « on s'est trompé », « correction : ». **Si tu ressens le besoin d'écrire l'un de ces mots, l'énoncé est CASSÉ → jette-le et génères-en un autre.**
   - **Abstention si incertain** : si tu n'es pas certain du fait (nom, date, écurie à une saison donnée, nombre de victoires), NE génère PAS la question. **N'invente JAMAIS un résultat ni une statistique.**
   - **Pas de « ça dépend »** : la bonne réponse doit être unique et incontestable (attention aux résultats modifiés après coup par des pénalités ou disqualifications : donne le **résultat officiel homologué**).

11. **Énoncé concis** : 10-140 chars (HARD CAP mobile, > 140 = REJECTED). Variété : ne commence pas > 40 % des questions par « Quel/Quelle ». Alterne : « Qui s'impose » · « Combien de titres » · « En quelle année » · « Sur quel circuit » · « Avec quelle écurie » · « Au volant de quelle monoplace ». ~un quart d'angles narratifs contextualisés (« Monaco, 1984, sous le déluge : la course est arrêtée. Qui menait alors ? »). Esprit chambreur SYSTÉMATIQUE dans `explanation`.

12. **`correct_answer` concis** : 1-80 chars, entité unique. PAS de parenthèses (pas `"Prost (McLaren)"`), PAS de multi-entités (`"Senna et Prost"`) — choisis UNE entité ou reformule. Une marque seule (« 7 titres », « 1950 ») est une entité valide.

13. **NE PAS générer de questions sur événements futurs incertains**. Pas de `correct_answer` « Aucun » / « À déterminer ».

14. **FRANÇAIS ACCENTUÉ (obligatoire)** : `text`, `correct_answer`, `wrong_answers[*].text` et `explanation` s'écrivent en français **correctement accentué** — é, è, ê, à, ç, ù, ô, î, û, et les majuscules accentuées (« À », « É »). Écris « l'écurie », « le Grand Prix d'Émilie-Romagne », « une année », jamais « ecurie » ni « annee ».

# Format `wrong_answers` (pool 7-10 distractors, danger 1-3)

Chaque `wrong_answers[i]` = `{ "text": "...", "danger": 1|2|3 }`. Le client tire 3 wrong au hasard à chaque rendu (anti-mémorisation). **Pool 7-10 distractors OBLIGATOIRE** — un pool plus court fait REJETER la question à l'import.

- **danger = 1 (décor)** : évident, peu probable de tromper.
- **danger = 2 (plausible)** : crédible mais inexact, peut tromper un fan moyen.
- **danger = 3 (piège fort)** : très proche de la bonne réponse, piège un fan averti. ⚠️ C'est le SEUL endroit où un piège est permis — l'énoncé et la `correct_answer` restent toujours vrais.

**Distribution attendue** (suis strictement) :

| Difficulté | Total wrong | danger=1 | danger=2 | danger=3 |
|---|---|---|---|---|
| bronze | 7 | 5 | 2 | 0 |
| argent | 8 | 2 | 4 | 2 |
| or | 10 | 2 | 4 | 4 |

**Anti-collision** : aucun `wrong_answers[*].text` ne doit égaler `correct_answer` (insensible casse/accents). Pas de doublons internes au pool.

# Tags (thème imposé prioritaire)

**Le tag qui fait foi est le slug du thème imposé** (cf. directive « THÈME UNIQUE IMPOSÉ ») — utilise-le tel quel sur CHAQUE question. Slugs F1 seedés (migration 0301, gérés en base) :

- **Le quoi** : `championnat-du-monde-f1`, `ecuries-f1`, `grands-prix-f1`, `francais-en-f1`, `f1-moderne`.
- **Transverses** : `legendes-f1`, `records-f1`, `regles-f1`.

Règles tagging :
- **1 batch = 1 thème principal.** Tague CHAQUE question avec le slug du thème imposé.
- 2ᵉ tag transverse **uniquement** s'il décrit réellement la question (le premier titre de Prost → `["championnat-du-monde-f1", "francais-en-f1"]` ; les sept titres de Schumacher → `["championnat-du-monde-f1", "records-f1"]`). Pas de sur-tagging décoratif.
- ⚠️ **Pas de tag cross-sport** : aucun slug d'un autre sport (la garde 0149 rejette la question).

# Format de sortie strict

Retourne **uniquement** un JSON array d'objets (pas de markdown wrapper, pas de préface, pas de commentaire). Aucun champ supplémentaire.

```json
[
  {
    "sport_slug": "f1",
    "difficulty": "argent",
    "text": "En 1985, quel Français devient pour la première fois champion du monde des pilotes ?",
    "correct_answer": "Alain Prost",
    "wrong_answers": [
      { "text": "René Arnoux", "danger": 3 },
      { "text": "Jacques Laffite", "danger": 3 },
      { "text": "Patrick Tambay", "danger": 2 },
      { "text": "Didier Pironi", "danger": 2 },
      { "text": "Jean-Pierre Jabouille", "danger": 2 },
      { "text": "Jean Alesi", "danger": 2 },
      { "text": "Olivier Panis", "danger": 1 },
      { "text": "Pierre Gasly", "danger": 1 }
    ],
    "themes": ["championnat-du-monde-f1", "francais-en-f1"],
    "source_url": "https://www.formula1.com/en/results/1985/drivers",
    "explanation": "Le Professeur décroche son premier diplôme en 1985 avec McLaren, puis trois autres (1986, 1989, 1993). Quatre titres, et un surnom qui n'a jamais été volé."
  }
]
```

> ℹ️ Exemple = argent (8 wrong : 2×d1 + 4×d2 + 2×d3). Pour bronze : 7 wrong (5×d1+2×d2). Pour or : 10 wrong (2×d1+4×d2+4×d3). **Suis la table de distribution.**

{{exclusion_block}}
# Exclusions strictes

- ❌ Aucune question sur paris sportifs, cotes, scandales liés aux paris.
- ❌ Aucune question sur dopage, affaires judiciaires ou drama personnel non sportif.
- ❌ Accidents : le fait sportif ancré est permis (une course arrêtée, une règle de sécurité née d'un drame), **jamais l'angle morbide** ni le détail des blessures.
- ❌ Aucun champ JSON en dehors de la spec ci-dessus.
- ❌ Aucun `themes` vide : chaque question porte au moins le slug du thème imposé.
- ❌ Aucun usage de marque/logo/sponsor comme label commercial (référence factuelle historique OK, cf. règle 2).
