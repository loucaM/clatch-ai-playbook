# Prompt versionné `question_gen_athletisme_v1.md` (ATHLÉTISME)

> Source de vérité humaine pour la génération de questions **athlétisme** via Claude (Epic Athlétisme, Story A1.4, 2026-07-19).
> L'athlé a son **propre prompt actif** en DB (`prompt_templates` sport-scopé : 1 ligne active par sport).
> **Publication** : copie le bloc « TEMPLATE À PUBLIER » ci-dessous dans `/admin/prompts` avec le **sélecteur de sport = athletisme**, version `v1.1`. DEV d'abord ; PROD seulement sur GO Louca.
> ⚠️ **Prérequis** : `'athletisme'` doit d'abord être présent dans `PROMPT_SPORT_SLUGS` (`admin/lib/schemas/create-prompt.ts:16`), sinon le `z.enum` du form de publication refuse le sport (Story A1.3, à faire AVANT de publier).
> Le foot garde son `question_gen_v3.md`, le tennis son `question_gen_tennis_v1.md`, le vélo son `question_gen_velo_v1.md`.

## Variables interpolées au runtime

L'EF `generate_questions_batch` remplace ces placeholders avant l'appel Anthropic (identique foot/tennis/vélo) :

| Placeholder | Rempli par | Contenu |
|---|---|---|
| `{{sport_slug}}` | EF (body) | `athletisme` |
| `{{difficulty}}` | EF (body) | `bronze` \| `argent` \| `or` |
| `{{count}}` | EF (body) | 1-50 |
| `{{theme_directive}}` | EF | Bloc « THÈME UNIQUE IMPOSÉ » (label + description) — vide si pas de thème |
| `{{exclusion_block}}` | EF (service-role) | Bloc « Questions DÉJÀ en base » (theme+difficulty) — vide si N=0 |

## Contrat de sortie (caps DURS — `_shared/schema.ts`, sinon REJECTED à l'import)

- `text` : **10-140 chars** (HARD CAP mobile ; sweet spot 80-100).
- `correct_answer` : 1-80 chars, **entité unique sans parenthèses ni multi-entités** (une marque « 9.58 s » ou « 8,95 m » est une entité valide).
- `wrong_answers` : pool 7-10 distractors `{ "text" ≤80, "danger": 1|2|3 }`.
- `explanation` : 50-300 chars, ton chambreur.
- `source_url` : URI 10-500.
- `themes` : ≥1 slug (le thème imposé), jamais vide. **Double-tag JO autorisé** (cf. section Tags).

---

## TEMPLATE À PUBLIER (copier tout ce qui suit dans /admin/prompts, sport = athletisme)

# Tâche — athlétisme v1.1

Génère **{{count}} questions** au format **JSON strict** spécifié plus bas, pour le sport `{{sport_slug}}` à la difficulté `{{difficulty}}`.

{{theme_directive}}
# Contexte projet

Clatch est un jeu mobile FR de quiz sportif, ton **chambreur (humour second degré)**. Cette série porte sur l'**ATHLÉTISME** (la piste ET les concours : sprint, haies, demi-fond, fond, marathon, sauts, lancers, épreuves combinées, marche). Format : `correct_answer + wrong_answers pool` (anti-mémorisation + danger calibré + tagging thématique).

# Règles éditoriales

1. **Ton chambreur dans `explanation`** : second degré, jamais méchant gratuit ni dégradant. Registre athlé : couloir, faux départ, blocs, finish, photo-finish, record du monde, dernier virage, ligne droite, ordre de passage.
2. **Doctrine "référence nécessaire" IP + OLYMPISME** : noms d'athlètes = informatif factuel autorisé. **Les Jeux olympiques peuvent être cités FACTUELLEMENT** (« aux Jeux de Pékin 2008 », « champion olympique du 400 m en 1996 ») — c'est de la référence historique nécessaire, pas du branding. **MAIS** : n'implique JAMAIS un partenariat/soutien officiel, n'utilise PAS les marques protégées comme label commercial ni les anneaux/logos ; en cas de doute sur une marque déposée (équipementiers, meetings sponsorisés), reste factuel ou fallback générique.
3. **Politique éditoriale H/F (athlétisme)** : l'athlé féminin est **au sommet mondial, à parité de prestige**. Valorise **~50/50 H/F**, zéro question dégradante ou stéréotypée. Championnes à mobiliser activement : Florence Griffith-Joyner, Marie-José Pérec, Allyson Felix, Shelly-Ann Fraser-Pryce, Elaine Thompson-Herah, Faith Kipyegon, Sifan Hassan, Nafissatou Thiam, Yelena Isinbayeva, Jackie Joyner-Kersee, Hicham… (non, lui c'est un homme — reste vigilant), Wilma Rudolph, Nawal El Moutawakel, Kathrine Switzer (marathon).
4. **Difficulté calibrée** :
   - **bronze** : faits ultra-connus, fan occasionnel. Varie les angles (recordman connu du 100 m, distance d'un marathon, ce qu'est le décathlon, discipline d'une star, pays d'un champion olympique récent). Évite le trop trivial répété.
   - **argent** : fan averti. Angles subtils : médaillé précis d'une finale olympique/mondiale, année d'un record, épreuve d'un athlète, records de France, cols… (non — pour l'athlé : stades mythiques, meetings de la Ligue de Diamant), enchaînements de titres.
   - **or** : érudit/expert. Marques exactes (9.58 s Berlin 2009, 8,95 m de Powell, 2,45 m de Sotomayor, 6,23 m de Duplantis), records méconnus, éditions anciennes, athlètes hors stars, détails **règlement/mesures** (poids du poids H = 7,26 kg / F = 4 kg, javelot H = 800 g / F = 600 g, hauteur des haies du 110 m H = 106,7 cm, 10 épreuves du décathlon, faux départ = disqualification directe depuis 2010).
5. **ANTI-RÉPÉTITION / VARIÉTÉ OBLIGATOIRE (renforcée)** : ne ressors PAS que Bolt et Owens. Le bloc « Questions DÉJÀ en base » fourni plus bas liste ce qui existe : génère du contenu **strictement nouveau** (ni ces faits, ni ces réponses, ni de simple reformulation d'un fait déjà présent — un même record/athlète/édition sous un autre angle compte comme un doublon). **Fais TOURNER les axes sur l'ensemble du batch** — ne reste pas collé au sprint et aux stars :
   - **Épreuves** : alterne sprint, haies, demi-fond, fond/marathon, marche, sauts (longueur, hauteur, perche, triple), lancers (poids, disque, javelot, marteau), épreuves combinées (déca/heptathlon). Vise ≤ 30 % de sprint sur un batch générique.
   - **Géographie** : ne surpondère pas USA/Jamaïque — mobilise le demi-fond/fond est-africain (Kenya, Éthiopie), l'Europe, la France, les Caraïbes, l'Océanie.
   - **Époques** : mélange ères (années 60-80 « oubliées », 90-2000, 2010+), pas seulement l'actu.
   - **Genre** : ~50/50 H/F (cf. règle 3).
   - **Angles rares à privilégier** : perchistes, lanceurs, marcheurs, marathoniennes, records de France oubliés, meetings historiques, médaillés « hors podium médiatique ».
6. **ANTI-PÉRISSABLE (CRITIQUE)** : bannis tout superlatif battable ou ancré au présent (« le meilleur sprinteur ACTUEL », « recordman en titre », « champion en cours »). **Ancre TOUJOURS dans le temps** : « aux Jeux de Pékin 2008 », « au 100 m de Berlin 2009 », « championne olympique en 1996 », « record du monde établi en 1988 ». Une question doit rester vraie dans 10 ans. Pas de résultat de compétition non encore disputée.
7. **Sources** — **Wikipédia ≤25%** (le reste DOIT venir d'ailleurs), **diversifie activement (~30 sources, ne retombe pas toujours sur les 3 mêmes)** et choisis LA PLUS PERTINENTE selon le contexte ; privilégie le primaire/spécialisé. _NB : « IAAF » est l'ancien nom de World Athletics (rebaptisée 2019) — même instance, cite « World Athletics »._
   - **Instances officielles** : **World Athletics (worldathletics.org)** ⭐ (fédération mondiale, ex-IAAF — records, résultats, la bible data) · **Olympics.com / CIO** (résultats & histoire olympiques) · **FFA (athle.fr)** (athlétisme français) · **European Athletics (european-athletics.com)** · confédérations continentales (CAA Afrique, Asian Athletics, CONSUDATLE Amérique du Sud).
   - **Circuits & compétitions** : **Wanda Diamond League (diamondleague.com)** ⭐ (meetings d'élite) · **World Marathon Majors (worldmarathonmajors.com)** (les 6 majeurs marathon) · Continental Tour · Championnats du monde / d'Europe (sites World/European Athletics).
   - **Stats / data / archives ⭐ (anti-hallucination — privilégie pour chiffres, marques, records)** : **World Athletics Toplists & records** ⭐ · **Tilastopaja (tilastopaja.eu)** ⭐ · **All-Athletics** · **Track & Field News (trackandfieldnews.com)** · **Power of 10 (thepowerof10.info)** (perfs/archives) · **ARRS — road-race statisticians (arrs.run)** (records route/marathon) · Olympedia (olympedia.org) pour l'historique olympique.
   - **Presse spécialisée FR** : L'Équipe (rubrique Athlétisme) · Ouest-France Athlé · Jogging International (fond/marathon).
   - **Presse spécialisée internationale** : Athletics Weekly (athleticsweekly.com) · LetsRun.com (demi-fond/fond) · FloTrack · Runner's World (route/marathon) · World Athletics Magazine.
8. **Pas de question piégeuse** sur « ça dépend » — la bonne réponse doit être incontestable.
9. **Énoncé concis** : 10-140 chars (HARD CAP mobile, > 140 = REJECTED). Variété : ne commence pas > 40% des questions par « Quel/Quelle ». Alterne : « Qui détient / a remporté » · « Combien mesure / pèse » · « Sur quelle distance » · « En quelle année » · « Dans quelle épreuve ». ~un quart d'angles narratifs contextualisés (« Séoul 1988, un chrono jamais battu depuis chez les femmes : qui ? »). Esprit chambreur SYSTÉMATIQUE dans `explanation`.
10. **`correct_answer` concis** : 1-80 chars, entité unique. PAS de parenthèses (pas `"Bolt (9.58 s)"`), PAS de multi-entités (`"Bolt et Gay"`) — choisis UNE entité ou reformule. Une marque seule (« 9.58 s », « 42,195 km ») est une entité valide.
11. **NE PAS générer de questions sur événements futurs incertains**. Pas de `correct_answer` « Aucun » / « À déterminer ».

# Format `wrong_answers` (pool 7-10 distractors, danger 1-3)

Chaque `wrong_answers[i]` = `{ "text": "...", "danger": 1|2|3 }`. Le client tire 3 wrong au hasard à chaque rendu (anti-mémorisation). **Pool 7-10 distractors OBLIGATOIRE.**

- **danger = 1 (décor)** : évident, peu probable de tromper.
- **danger = 2 (plausible)** : crédible mais inexact, peut tromper un fan moyen.
- **danger = 3 (piège fort)** : très proche de la bonne réponse, piège un fan averti.

**Distribution attendue** (suis strictement) :

| Difficulté | Total wrong | danger=1 | danger=2 | danger=3 |
|---|---|---|---|---|
| bronze | 7 | 5 | 2 | 0 |
| argent | 8 | 2 | 4 | 2 |
| or | 10 | 2 | 4 | 4 |

**Anti-collision** : aucun `wrong_answers[*].text` ne doit égaler `correct_answer` (insensible casse/accents). Pas de doublons internes au pool.

# Tags (thème imposé prioritaire + PRÉ-TAG JO)

**Le tag qui fait foi est le slug du thème imposé** (cf. directive « THÈME UNIQUE IMPOSÉ ») — utilise-le tel quel sur CHAQUE question. Slugs athlé seedés (migration 0185, gérés en base) :

- **Disciplines** : `sprint-vitesse`, `fond-demi-fond`, `sauts`, `lancers`.
- **Transverses** : `jeux-olympiques`, `records-histoire`, `regles-athletisme`, `anecdotes-citations`, `legendes-athletisme`, `athletisme-francais`.

Règles tagging :
- **1 batch = 1 thème principal.** Tague CHAQUE question avec le slug du thème imposé.
- **⭐ PRÉ-TAG JO (règle athlé spécifique)** : si la question porte sur un **fait olympique** (une performance, médaille, édition, record réalisés AUX Jeux olympiques), **AJOUTE `jeux-olympiques` au tableau `themes[]` EN PLUS du thème imposé** — sauf si `jeux-olympiques` EST déjà le thème imposé. Exemple : batch « Le Sprint », question sur Bolt à Pékin 2008 → `themes: ["sprint-vitesse", "jeux-olympiques"]`.
- **Faits historiques** : olympique → double-tag `jeux-olympiques` ; hors-JO (record du monde, première mondiale) → le fait vit plutôt sous `records-histoire` ; centré sur une carrière → `legendes-athletisme`.
- 2ᵉ tag transverse **uniquement** s'il décrit réellement la question. Pas de sur-tagging décoratif.

# Format de sortie strict

Retourne **uniquement** un JSON array d'objets (pas de markdown wrapper, pas de préface, pas de commentaire). Aucun champ supplémentaire.

```json
[
  {
    "sport_slug": "athletisme",
    "difficulty": "argent",
    "text": "En quelle année Usain Bolt établit-il son record du monde du 100 m à Berlin ?",
    "correct_answer": "2009",
    "wrong_answers": [
      { "text": "2008", "danger": 3 },
      { "text": "2011", "danger": 3 },
      { "text": "2012", "danger": 2 },
      { "text": "2010", "danger": 2 },
      { "text": "2007", "danger": 1 },
      { "text": "2013", "danger": 1 },
      { "text": "2005", "danger": 1 },
      { "text": "2015", "danger": 1 }
    ],
    "themes": ["sprint-vitesse"],
    "source_url": "https://worldathletics.org",
    "explanation": "9 secondes 58 aux Mondiaux de Berlin. Pékin 2008, c'était le 9.69 en levant les bras. Là, il n'a levé personne. Toujours pas battu."
  }
]
```

> ℹ️ Exemple = argent (8 wrong : 2×d1 + 4×d2 + 2×d3). Pour bronze : 7 wrong (5×d1+2×d2). Pour or : 10 wrong (2×d1+4×d2+4×d3). **Suis la table de distribution.**
> ℹ️ Double-tag JO : si Berlin 2009 était une finale olympique (ce n'est pas le cas — Mondiaux), on aurait mis `themes: ["sprint-vitesse", "jeux-olympiques"]`.

{{exclusion_block}}
# Exclusions strictes

- ❌ Aucune question sur paris sportifs, cotes, scandales liés aux paris.
- ❌ Aucune question sur dopage traité comme drama personnel (le fait sportif ancré est ok ; l'attaque gratuite non).
- ❌ Aucune question sur drama personnel non sportif.
- ❌ Aucun champ JSON en dehors de la spec ci-dessus.
- ❌ Aucun `themes` vide : chaque question porte au moins le slug du thème imposé.
- ❌ Aucun usage de marque/logo olympique comme label commercial (référence factuelle historique OK, cf. règle 2).

---

## Changelog

- **v1.1** (2026-07-27) — Anti-boucle. (a) **Sources élargies** (règle 7) : ajout Diamond League ⭐, World Marathon Majors, Power of 10, ARRS (route/marathon), confédérations continentales, Runner's World ; note « IAAF = ex-World Athletics » pour lever la confusion. (b) **Anti-répétition renforcée** (règle 5) : rotation OBLIGATOIRE des axes sur le batch (épreuves ≤ 30 % sprint, géographie est-africaine/Europe/Caraïbes, époques mêlées, ~50/50 H/F) + définition élargie du doublon (reformulation d'un fait déjà en base = doublon). À republier via /admin/prompts (sport=athletisme, v1.1) sur DEV puis PROD sur GO. Se combine avec l'upgrade modèle Opus 4.7→4.8 et `EXCLUSION_LIMIT` 120→250 (EF `generate_questions_batch`).
- **v1.0** (2026-07-19) — Epic Athlétisme, Story A1.4. Premier prompt athlé (prompt_templates sport-scopé). Cloné sur `question_gen_velo_v1.md`, recalibré athlétisme : registre piste/concours (couloir, faux départ, finish, photo-finish, blocs), politique H/F **~50/50** (athlé féminin à parité de prestige — Flo-Jo/Pérec/Felix/Fraser-Pryce/Kipyegon/Thiam), sources athlé primaires (**World Athletics ⭐**, Olympics.com/CIO, FFA, Tilastopaja ⭐, Track & Field News, Olympedia), règle 2 **doctrine olympisme** (référence factuelle des JO autorisée, branding/logos interdits — loi 1992 CIO), difficulté athlé (mesures/règlement en `or` : poids 7,26 kg, haies 106,7 cm), **⭐ PRÉ-TAG JO** (double-tag `jeux-olympiques` sur tout fait olympique, via `question_themes` N-N), tags = thèmes seedés migration 0185. Contrat de sortie identique foot/tennis/vélo (text ≤140, pool 7-10, explanation 50-300). À publier via /admin/prompts (sport=athletisme) sur DEV, puis PROD sur GO Louca + content gate (plancher A1.6). Prérequis : `'athletisme'` dans PROMPT_SPORT_SLUGS (Story A1.3).
