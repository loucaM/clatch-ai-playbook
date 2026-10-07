# Prompt versionné `question_gen_basket_v1.md` (BASKET)

> Source de vérité humaine pour la génération de questions **basket** via Claude (Epic Basket, Story B1.3, 2026-08-13).
> Le basket a son **propre prompt actif** en DB (`prompt_templates` sport-scopé : 1 ligne active par sport).
> **Publication** : copie le bloc « TEMPLATE À PUBLIER » ci-dessous dans `/admin/prompts` avec le **sélecteur de sport = basket**, version `v1.0`. DEV d'abord ; PROD seulement sur GO Louca.
> ⚠️ **Prérequis** : `'basket'` doit d'abord être présent dans `PROMPT_SPORT_SLUGS` (`admin/lib/schemas/create-prompt.ts`) — fait en Story B1.2 — sinon le `z.enum` du form de publication refuse le sport.
> Le foot garde son `question_gen_v3.md`, le tennis son `question_gen_tennis_v1.md`, le vélo son `question_gen_velo_v1.md`, l'athlé son `question_gen_athletisme_v1.md`.

## Variables interpolées au runtime

L'EF `generate_questions_batch` remplace ces placeholders avant l'appel Anthropic (identique foot/tennis/vélo/athlé) :

| Placeholder | Rempli par | Contenu |
|---|---|---|
| `{{sport_slug}}` | EF (body) | `basket` |
| `{{difficulty}}` | EF (body) | `bronze` \| `argent` \| `or` |
| `{{count}}` | EF (body) | 1-50 |
| `{{theme_directive}}` | EF | Bloc « THÈME UNIQUE IMPOSÉ » (label + description) — vide si pas de thème |
| `{{exclusion_block}}` | EF (service-role) | Bloc « Questions DÉJÀ en base » (theme+difficulty) — vide si N=0 |

## Contrat de sortie (caps DURS — `_shared/schema.ts`, sinon REJECTED à l'import)

- `text` : **10-140 chars** (HARD CAP mobile ; sweet spot 80-100).
- `correct_answer` : 1-80 chars, **entité unique sans parenthèses ni multi-entités** (une marque « 100 points » ou « 42 » est une entité valide).
- `wrong_answers` : pool 7-10 distractors `{ "text" ≤80, "danger": 1|2|3 }`.
- `explanation` : 50-300 chars, ton chambreur.
- `source_url` : URI 10-500.
- `themes` : ≥1 slug (le thème imposé), jamais vide.

---

## TEMPLATE À PUBLIER (copier tout ce qui suit dans /admin/prompts, sport = basket)

# Tâche — basket v1.1

Génère **{{count}} questions** au format **JSON strict** spécifié plus bas, pour le sport `{{sport_slug}}` à la difficulté `{{difficulty}}`.

{{theme_directive}}
# Contexte projet

Clatch est un jeu mobile FR de quiz sportif, ton **chambreur (humour second degré)**. Cette série porte sur le **BASKET** (NBA, basket français et européen, équipes de France, WNBA, sélections, histoire du jeu). Format : `correct_answer + wrong_answers pool` (anti-mémorisation + danger calibré + tagging thématique).

# Règles éditoriales

1. **Ton chambreur dans `explanation`** : second degré, jamais méchant gratuit ni dégradant. Registre basket : buzzer beater, money time, poster, airball, la raquette, le parquet, prolongation, Game 7, alley-oop, brique.
2. **Doctrine "référence nécessaire" IP** : noms de joueurs/joueuses, franchises, clubs et compétitions = informatif factuel autorisé (« aux Finales NBA 1998 », « à l'EuroBasket 2013 », « champions olympiques 2024 »). **MAIS** : n'implique JAMAIS un partenariat officiel, n'utilise PAS logos ni marques comme label commercial ; en cas de doute (équipementiers, naming de salles), reste factuel ou fallback générique.
3. **Politique éditoriale H/F (basket)** : le basket féminin compte (WNBA, EuroLeague féminine, les Braqueuses médaillées olympiques 2012, l'équipe de France féminine). Vise une **part significative de questions féminines (~30 %)**, zéro question dégradante ou stéréotypée. À mobiliser : Céline Dumerc, Sandrine Gruda, Marine Johannès, Diana Taurasi, Sue Bird, Lisa Leslie, A'ja Wilson, Caitlin Clark, Breanna Stewart, Lauren Jackson.
4. **Difficulté calibrée** :
   - **bronze** : faits ultra-connus, fan occasionnel. Varie les angles (franchise de Michael Jordan, nombre de points d'un panier à 3 pts, ville des Lakers, poste de Victor Wembanyama, pays hôte d'un EuroBasket récent). Évite le trop trivial répété.
   - **argent** : fan averti. Angles subtils : MVP d'une finale précise, palmarès d'un club français, campagne des Bleus (Sydney 2000, Tokyo 2021, Paris 2024), records de franchise, draft d'une star, carrière européenne d'un joueur NBA.
   - **or** : érudit/expert. Marques exactes (100 points de Wilt en 1962, 81 de Kobe en 2006, quadruple-double, séries d'invincibilité), éditions anciennes (premiers EuroBasket, ères Bill Russell/Celtics), joueurs hors stars, détails **règlement** (24 secondes, 3 secondes dans la raquette, goaltending, bonus de fautes, dimensions : arceau à 3,05 m, ligne à 3 pts FIBA 6,75 m vs NBA 7,24 m).
5. **ANTI-RÉPÉTITION / VARIÉTÉ OBLIGATOIRE** : ne ressors PAS que Jordan, LeBron et Wembanyama. Le bloc « Questions DÉJÀ en base » fourni plus bas liste ce qui existe : génère du contenu **strictement nouveau** (ni ces faits, ni ces réponses, ni de simple reformulation — un même record/joueur/édition sous un autre angle compte comme un doublon). **Fais TOURNER les axes sur l'ensemble du batch** :
   - **Compétitions** : alterne NBA, Betclic Élite / basket français, EuroLeague, EuroBasket, Mondiaux, JO, WNBA. Vise ≤ 40 % de NBA sur un batch générique.
   - **Géographie** : ne surpondère pas les USA — mobilise la France (ASVEL, Monaco, Limoges CSP 1993, Pau-Orthez), l'Europe (Real, Panathinaïkos, Étoile Rouge, la Yougoslavie historique), les mondiaux.
   - **Époques** : mélange ères (Russell/Chamberlain, Bird/Magic, Jordan 90s, Shaq/Kobe/Duncan 2000s, 2010+), pas seulement l'actu.
   - **Genre** : ~30 % féminin (cf. règle 3).
   - **Angles rares à privilégier** : pivots européens pionniers, Limoges champion d'Europe 1993, les Braqueuses 2012, Tony Parker MVP des Finales 2007, la draft des Français, sixièmes hommes, records de la Betclic Élite.
6. **ANTI-PÉRISSABLE (CRITIQUE)** : bannis tout superlatif battable ou ancré au présent (« le meilleur marqueur ACTUEL », « détenteur du record en titre », « champion en cours »). **Ancre TOUJOURS dans le temps** : « aux Finales NBA 2016 », « à l'EuroBasket 2013 », « lors de la saison 1995-96 », « aux JO de Paris 2024 ». Une question doit rester vraie dans 10 ans. Pas de résultat de compétition non encore disputée, pas de « franchise actuelle » d'un joueur en activité (les transferts périment — ancre par saison).
7. **Sources** — **Wikipédia ≤25%** (le reste DOIT venir d'ailleurs), **diversifie activement (~30 sources)** et choisis LA PLUS PERTINENTE ; privilégie le primaire/spécialisé.
   - **Instances officielles** : **NBA.com** ⭐ (stats, historique) · **FIBA.basketball** ⭐ (compétitions internationales, EuroBasket, Mondiaux) · **FFBB (ffbb.com)** (basket français, équipes de France) · **EuroLeague (euroleaguebasketball.net)** · **WNBA.com** · **LNB (lnb.fr)** (Betclic Élite, historique Pro A).
   - **Stats / data / archives ⭐ (anti-hallucination — privilégie pour chiffres et records)** : **Basketball-Reference (basketball-reference.com)** ⭐ · **RealGM** · **Proballers (proballers.com)** (carrières européennes) · archives LNB · Olympedia (olympedia.org) pour l'historique olympique.
   - **Presse spécialisée FR** : L'Équipe (rubrique Basket) · BeBasket (bebasket.fr) · Basket Le Mag · First Team / TrashTalk (culture NBA FR, pour les angles, pas pour les chiffres).
   - **Presse spécialisée internationale** : ESPN NBA · The Athletic · HoopsHype · Eurohoops (eurohoops.net) · Sportando.
8. **INTÉGRITÉ DE L'ÉNONCÉ (RÈGLE DURE — priorité absolue ; tout manquement rend la question INUTILISABLE)** :
   - **Vérité littérale de l'énoncé** : la `correct_answer` doit satisfaire CHAQUE prédicat de l'énoncé — nationalité, poste, année, nombre, franchise/club, compétition, tour, saison. Si l'énoncé dit « Quel Français… », la réponse EST française ; s'il dit « pivot », la réponse EST pivot ; s'il dit « 3 titres consécutifs », la réponse en a bien 3 ; s'il dit « qui bat X en finale », la réponse est le VAINQUEUR. **Interdiction absolue de fabriquer une fausse prémisse** (mauvaise nationalité/poste/année/nombre) pour tendre un piège.
   - **Le piège vit UNIQUEMENT dans les `wrong_answers`** (les leurres danger=3), JAMAIS dans l'énoncé ni dans la `correct_answer`. L'énoncé dit toujours la vérité ; seuls les distracteurs tendent le piège.
   - **L'`explanation` CONFIRME la `correct_answer`** : elle ne nomme JAMAIS une autre entité comme vraie réponse, ne se corrige pas, ne doute pas, ne raisonne pas à voix haute. Mots BANNIS dans l'explication : « piège », « à reformuler », « mea culpa », « en fait c'est… », « aucun match », « oups », « on s'est trompé ». **Si tu ressens le besoin d'écrire l'un de ces mots pour justifier ta question, l'énoncé est CASSÉ → jette-le et génères-en un autre.**
   - **Abstention si incertain** : si tu n'es pas certain du fait (nom, date, chiffre, résultat, nationalité, franchise à une saison donnée), NE génère PAS la question — change d'angle ou de sujet. **N'invente JAMAIS un résultat, une nationalité ou une statistique.** Mieux vaut moins de questions que des questions fausses.
   - **Pas de « ça dépend »** : la bonne réponse doit être unique et incontestable.
9. **Énoncé concis** : 10-140 chars (HARD CAP mobile, > 140 = REJECTED). Variété : ne commence pas > 40% des questions par « Quel/Quelle ». Alterne : « Qui détient / a remporté » · « Combien de points » · « En quelle année » · « Dans quelle franchise » · « Face à qui ». ~un quart d'angles narratifs contextualisés (« 1993, un club français sur le toit de l'Europe : lequel ? »). Esprit chambreur SYSTÉMATIQUE dans `explanation`.
10. **`correct_answer` concis** : 1-80 chars, entité unique. PAS de parenthèses (pas `"Jordan (6 titres)"`), PAS de multi-entités (`"Jordan et Pippen"`) — choisis UNE entité ou reformule. Une marque seule (« 100 points », « 6,75 m ») est une entité valide.
11. **NE PAS générer de questions sur événements futurs incertains**. Pas de `correct_answer` « Aucun » / « À déterminer ».

# Format `wrong_answers` (pool 7-10 distractors, danger 1-3)

Chaque `wrong_answers[i]` = `{ "text": "...", "danger": 1|2|3 }`. Le client tire 3 wrong au hasard à chaque rendu (anti-mémorisation). **Pool 7-10 distractors OBLIGATOIRE.**

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

**Le tag qui fait foi est le slug du thème imposé** (cf. directive « THÈME UNIQUE IMPOSÉ ») — utilise-le tel quel sur CHAQUE question. Slugs basket seedés (migration 0226, gérés en base) :

- **Compétitions** : `nba`, `basket-francais`, `equipe-de-france-basket`, `euroleague-europe`.
- **Transverses** : `legendes-basket`, `records-basket`, `regles-basket`.

Règles tagging :
- **1 batch = 1 thème principal.** Tague CHAQUE question avec le slug du thème imposé.
- 2ᵉ tag transverse **uniquement** s'il décrit réellement la question (une question NBA sur le 100 points de Wilt → `["nba", "records-basket"]` ; une campagne des Bleus centrée sur une carrière → `["equipe-de-france-basket", "legendes-basket"]`). Pas de sur-tagging décoratif.
- ⚠️ **Pas de tag JO cross-sport** : `jeux-olympiques` appartient à l'athlétisme en base. Un fait olympique basket vit sous `equipe-de-france-basket` (campagnes des Bleus/Braqueuses) ou `records-basket` (Dream Team 1992).

# Format de sortie strict

Retourne **uniquement** un JSON array d'objets (pas de markdown wrapper, pas de préface, pas de commentaire). Aucun champ supplémentaire.

```json
[
  {
    "sport_slug": "basket",
    "difficulty": "argent",
    "text": "En 2007, quel Français est MVP des Finales NBA avec les Spurs ?",
    "correct_answer": "Tony Parker",
    "wrong_answers": [
      { "text": "Boris Diaw", "danger": 3 },
      { "text": "Nicolas Batum", "danger": 2 },
      { "text": "Manu Ginobili", "danger": 3 },
      { "text": "Tim Duncan", "danger": 2 },
      { "text": "Joakim Noah", "danger": 2 },
      { "text": "Rudy Gobert", "danger": 1 },
      { "text": "Evan Fournier", "danger": 1 },
      { "text": "Victor Wembanyama", "danger": 2 }
    ],
    "themes": ["nba", "equipe-de-france-basket"],
    "source_url": "https://www.basketball-reference.com",
    "explanation": "Balayage 4-0 des Cavs de LeBron, et TP repart avec le trophée. Premier Européen MVP des Finales. Duncan a validé, LeBron un peu moins."
  }
]
```

> ℹ️ Exemple = argent (8 wrong : 2×d1 + 4×d2 + 2×d3). Pour bronze : 7 wrong (5×d1+2×d2). Pour or : 10 wrong (2×d1+4×d2+4×d3). **Suis la table de distribution.**

{{exclusion_block}}
# Exclusions strictes

- ❌ Aucune question sur paris sportifs, cotes, scandales liés aux paris.
- ❌ Aucune question sur dopage ou drama personnel non sportif (le fait sportif ancré est ok ; l'attaque gratuite non).
- ❌ Aucun champ JSON en dehors de la spec ci-dessus.
- ❌ Aucun `themes` vide : chaque question porte au moins le slug du thème imposé.
- ❌ Aucun usage de marque/logo comme label commercial (référence factuelle historique OK, cf. règle 2).

---

## Changelog

- **v1.1** (2026-08-19) — Epic Basket. **Porte le bloc « INTÉGRITÉ DE L'ÉNONCÉ » (règle 8)** des prompts live v1.2 (tennis/vélo/athlé) et v3.4 (foot), absent de la v1.0 : la v1.0 avait été clonée sur le FICHIER athlé v1.1 du repo, alors que la version publiée en base était déjà la v1.2. Sans ce bloc, rien n'interdisait la fausse prémisse dans l'énoncé, l'explication qui se corrige toute seule, ou l'invention d'un résultat quand le modèle n'est pas sûr — le mode d'échec exact de la purge citations-fun (batch à 66 % de faux passé en actif). Adapté au registre basket (pivot, franchise à une saison donnée). Ajoute aussi la précision danger=3 : le piège ne vit QUE dans les distracteurs. Rien d'autre ne bouge (sources, thèmes, rotation ≤40 % NBA, ~30 % féminin, contrat de sortie identiques).
- **v1.0** (2026-08-13) — Epic Basket, Story B1.3. Premier prompt basket (prompt_templates sport-scopé). Cloné sur `question_gen_athletisme_v1.md`, recalibré basket : registre parquet (buzzer beater, money time, raquette, Game 7), rotation des compétitions (≤ 40 % NBA, basket FR/Europe/sélections mobilisés — Limoges 1993, Braqueuses 2012, TP 2007), ~30 % féminin (WNBA/EDF), sources primaires (**NBA.com ⭐, FIBA ⭐, Basketball-Reference ⭐**, FFBB, LNB, EuroLeague), anti-périssable spécifique transferts (ancrer par saison, jamais « franchise actuelle »), interdiction du tag JO cross-sport (guard 0149 — l'olympisme basket vit sous `equipe-de-france-basket`/`records-basket`), tags = 7 thèmes seedés migration 0226. Contrat de sortie identique aux autres sports (text ≤140, pool 7-10, explanation 50-300). À publier via /admin/prompts (sport=basket) sur DEV, puis PROD sur GO Louca + content gate. Prérequis : `'basket'` dans PROMPT_SPORT_SLUGS (Story B1.2, fait).
