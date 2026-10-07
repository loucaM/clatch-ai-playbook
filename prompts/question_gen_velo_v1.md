# Prompt versionné `question_gen_velo_v1.md` (CYCLISME)

> Source de vérité humaine pour la génération de questions **cyclisme** via Claude (Epic Cyclisme, Story C1.3, 2026-07-16).
> Le vélo a son **propre prompt actif** en DB (`prompt_templates` sport-scopé : 1 ligne active par sport).
> **Publication** : copie le bloc « TEMPLATE À PUBLIER » ci-dessous dans `/admin/prompts` avec le **sélecteur de sport = velo**, version `v1.1`. DEV d'abord ; PROD seulement sur GO Louca.
> ⚠️ **Prérequis** : `'velo'` doit d'abord être présent dans `PROMPT_SPORT_SLUGS` (`admin/lib/schemas/create-prompt.ts:14`), sinon le `z.enum` du form de publication refuse le sport (Story C1.2, à faire AVANT de publier).
> Tout ajustement = bump version + changelog en bas. Le foot garde son `question_gen_v3.md`, le tennis son `question_gen_tennis_v1.md`.

## Variables interpolées au runtime

L'EF `generate_questions_batch` remplace ces placeholders avant l'appel Anthropic (identique foot/tennis) :

| Placeholder | Rempli par | Contenu |
|---|---|---|
| `{{sport_slug}}` | EF (body) | `velo` |
| `{{difficulty}}` | EF (body) | `bronze` \| `argent` \| `or` |
| `{{count}}` | EF (body) | 1-50 |
| `{{theme_directive}}` | EF | Bloc « THÈME UNIQUE IMPOSÉ » (label + description) — vide si pas de thème |
| `{{exclusion_block}}` | EF (service-role) | Bloc « Questions DÉJÀ en base » (theme+difficulty) — vide si N=0 |

## Contrat de sortie (caps DURS — `_shared/schema.ts`, sinon REJECTED à l'import)

- `text` : **10-140 chars** (HARD CAP mobile — toute question > 140 est rejetée, coût LLM perdu ; sweet spot 80-100).
- `correct_answer` : 1-80 chars, **entité unique sans parenthèses ni multi-entités**.
- `wrong_answers` : pool 7-10 distractors `{ "text" ≤80, "danger": 1|2|3 }`.
- `explanation` : 50-300 chars, ton chambreur.
- `source_url` : URI 10-500.
- `themes` : ≥1 slug (le thème imposé), jamais vide.

---

## TEMPLATE À PUBLIER (copier tout ce qui suit dans /admin/prompts, sport = velo)

# Tâche — cyclisme v1.1

Génère **{{count}} questions** au format **JSON strict** spécifié plus bas, pour le sport `{{sport_slug}}` à la difficulté `{{difficulty}}`.

{{theme_directive}}
# Contexte projet

Clatch est un jeu mobile FR de quiz sportif, ton **chambreur (humour second degré)**. Cette série porte sur le **CYCLISME (vélo sur route)**. Format : `correct_answer + wrong_answers pool` (anti-mémorisation + danger calibré + tagging thématique).

# Règles éditoriales

1. **Ton chambreur dans `explanation`** : second degré, jamais méchant gratuit ni dégradant. Registre course : maillot jaune, échappée, flamme rouge, bordure, gruppetto.
2. **Doctrine "référence nécessaire" IP** : noms de coureurs informatif autorisé ; marques (vélos, équipementiers, organisateurs ASO/RCS) nominatif factuel uniquement ; pas d'allusion olympique protégée (loi 1992) — fallback générique en cas de doute.
3. **Politique éditoriale H/F (cyclisme)** : le cyclisme féminin est en **forte croissance** (Tour de France Femmes depuis 2022, Women's WorldTour). Le palmarès masculin domine historiquement la culture, mais **valorise activement les championnes** : Jeannie Longo (légende FR), Marianne Vos, Annemiek van Vleuten, Demi Vollering, Pauline Ferrand-Prévot, Lizzie Deignan, Beryl Burton. Vise **~70/30 H/F**, zéro question dégradante ou stéréotypée.
4. **Difficulté calibrée** :
   - **bronze** : faits ultra-connus, fan occasionnel. Varie les angles (couleur d'un maillot distinctif, pays d'une épreuve, discipline sprinteur/grimpeur, vainqueur multiple récent, ce que récompense le maillot à pois / vert). Évite le trop trivial répété.
   - **argent** : fan averti. Angles subtils : vainqueur précis d'une édition, année du 1er grand tour d'un coureur, équipe d'un champion, cols mythiques (Ventoux, Galibier, Tourmalet, Alpe d'Huez, Mortirolo), classiques (« l'Enfer du Nord » = Paris-Roubaix, le Ronde), coureurs cadres d'une époque.
   - **or** : érudit/expert. Stats précises (victoires d'étapes de Merckx = 34, record de l'heure exact, palmarès en Grands Tours), records méconnus, éditions anciennes, coureurs hors stars, détails matériel/règles (arrivée du dérailleur, oreillettes, bonifications, classement par points, catégories de cols HC/1-4, ravitaillement).
5. **ANTI-RÉPÉTITION / VARIÉTÉ OBLIGATOIRE (renforcée)** : ne ressors PAS systématiquement les faits ultra-célèbres (ex : pas 50× « Merckx le Cannibale » ou « Poulidor éternel second »). Le bloc « Questions DÉJÀ en base » fourni plus bas liste ce qui existe : génère du contenu **strictement nouveau** (ni ces faits, ni ces réponses, ni une reformulation d'un fait déjà présent — même coureur/épreuve/édition sous un autre angle = doublon). **Fais TOURNER les axes sur l'ensemble du batch** :
   - **Épreuves** : alterne Grands Tours (Tour/Giro/Vuelta), Monuments (Sanremo, Ronde, Roubaix, Liège, Lombardie), classiques secondaires, championnats (monde/national), contre-la-montre, piste — ne reste pas bloqué sur le Tour de France.
   - **Profils** : grimpeurs, sprinteurs, rouleurs, puncheurs, équipiers de l'ombre — pas seulement les leaders.
   - **Géographie** : FR, Belgique, Italie, Espagne, Pays-Bas, mais aussi Colombie, Slovénie, GB, Australie.
   - **Époques** : mêle l'ère héroïque (Coppi/Bartali/Anquetil), 70-90 (Merckx/Hinault/Fignon), 2000s, 2010+ et actuel.
   - **Genre** : ~70/30 H/F (cf. règle 3), valorise le circuit féminin.
   - **Angles rares à privilégier** : classiques oubliées, éditions anciennes, records de côte, matériel/règles.
6. **ANTI-PÉRISSABLE (CRITIQUE — le Tour 2026 est EN COURS)** : bannis tout superlatif battable ou ancré au présent (« le maillot jaune ACTUEL », « meilleur grimpeur en cours », « recordman actuel »). **Ancre TOUJOURS dans le temps** : « au Tour 2026 (étape X) », « vainqueur du Tour 2023 », « sur le Ventoux en 2021 », « édition 1989 ». Une question doit rester vraie après la course. Pas de résultat d'étape non encore disputée.
7. **Sources** — **Wikipédia ≤25% des questions** (le reste DOIT venir d'ailleurs), **diversifie activement (~40 sources)** et choisis LA PLUS PERTINENTE selon le contexte ; privilégie le primaire/spécialisé à Wikipédia :
   - **Instances & organisateurs officiels** : ASO / letour.fr (Tour) · UCI (uci.org) · FFC (ffc.fr) · RCS Sport / giroditalia.it (Giro) · lavuelta.es (Vuelta) · Flanders Classics.
   - **Grands Tours & Monuments (officiels)** : letour.fr · giroditalia.it · lavuelta.es · Paris-Roubaix · Ronde van Vlaanderen (Tour des Flandres) · Milano-Sanremo · Liège-Bastogne-Liège · Il Lombardia.
   - **Stats / data / archives ⭐ (anti-hallucination — privilégie pour chiffres, palmarès, records)** : **ProCyclingStats (procyclingstats.com)** ⭐ (la bible data du cyclisme, l'équivalent de Tennis Abstract) · **BikeRaceInfo (bikeraceinfo.com)** · **FirstCycling (firstcycling.com)** · Mémoire du cyclisme (memoire-du-cyclisme.eu) · CQ Ranking · CyclingArchives (cyclingarchives.com) · The Inner Ring (inrng.com, analyse/histoire).
   - **Presse spécialisée FR** : L'Équipe (rubrique Cyclisme) · Cyclism'Actu · DirectVelo · Vélo Magazine · Le Gruppetto · Ouest-France Cyclisme.
   - **Presse spécialisée internationale** : Cyclingnews (cyclingnews.com) · Velo / VeloNews · Rouleur · GCN · Escape Collective · Het Nieuwsblad (sportwereld) · La Gazzetta dello Sport · Cycling Weekly.
   - **Cyclisme féminin dédié** : Tour de France Femmes (officiel) · couvertures Women's WorldTour (valorise Vos / van Vleuten / Vollering / Ferrand-Prévot / Longo / Deignan).
8. **Pas de question piégeuse** sur « ça dépend » — la bonne réponse doit être incontestable.
9. **Énoncé concis** : 10-140 chars (HARD CAP mobile, > 140 = REJECTED). Variété des formulations : ne commence pas > 40% des questions par « Quel/Quelle ». Alterne : « Qui a/est/détient » · « Combien de » · « Dans quel col / Sur quelle épreuve » · « Contre qui » · « En quelle année ». ~un quart d'angles narratifs contextualisés (« Tour 1989, 8 secondes d'écart au général sur les Champs : qui l'emporte ? »). Esprit chambreur SYSTÉMATIQUE dans `explanation`.
10. **`correct_answer` concis** : 1-80 chars, entité unique. PAS de parenthèses (pas `"Merckx (34 étapes)"`), PAS de multi-entités (`"Merckx et Hinault"`) — choisis UNE entité ou reformule.
11. **NE PAS générer de questions sur événements futurs incertains** (étape/parcours à venir). Pas de `correct_answer` « Aucun » / « À déterminer ».

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

# Tags (thème imposé prioritaire)

**Le tag qui fait foi est le slug du thème imposé** (cf. directive « THÈME UNIQUE IMPOSÉ ») — utilise-le tel quel sur CHAQUE question. Slugs vélo seedés (migration 0156, NON exhaustifs — gérés en base) :
`tour-de-france`, `legendes-cyclisme`, `cyclisme-francais`, `grands-tours`, `classiques-velo`, `montagne-cols`.

Règles tagging :
- **1 batch = 1 thème principal.** Tague CHAQUE question avec le slug du thème imposé. Si sous-thème, ajoute le parent.
- 2ᵉ tag transverse autorisé **seulement** s'il décrit réellement la question. Pas de sur-tagging décoratif.

# Format de sortie strict

Retourne **uniquement** un JSON array d'objets (pas de markdown wrapper, pas de préface, pas de commentaire). Aucun champ supplémentaire.

```json
[
  {
    "sport_slug": "velo",
    "difficulty": "bronze",
    "text": "Quelle couleur porte le leader du classement général du Tour de France ?",
    "correct_answer": "Jaune",
    "wrong_answers": [
      { "text": "Vert", "danger": 2 },
      { "text": "À pois rouges", "danger": 2 },
      { "text": "Blanc", "danger": 1 },
      { "text": "Bleu", "danger": 1 },
      { "text": "Rouge", "danger": 1 },
      { "text": "Arc-en-ciel", "danger": 1 },
      { "text": "Noir", "danger": 1 }
    ],
    "themes": ["tour-de-france"],
    "source_url": "https://www.letour.fr",
    "explanation": "Le maillot jaune, hérité du papier jaune du journal L'Auto qui a créé la course. Le vert récompense le sprint, les pois la montagne. Range ta boîte de crayons."
  }
]
```

> ℹ️ Exemple = bronze (7 wrong, 5×d1 + 2×d2). Pour argent : 8 wrong (2×d1+4×d2+2×d3). Pour or : 10 wrong (2×d1+4×d2+4×d3). **Suis la table de distribution.**

{{exclusion_block}}
# Exclusions strictes

- ❌ Aucune question sur paris sportifs, cotes, scandales liés aux paris.
- ❌ Aucune question sur dopage traité comme drama personnel (le fait sportif ancré est ok ; l'attaque gratuite non).
- ❌ Aucune question sur drama personnel non sportif.
- ❌ Aucun champ JSON en dehors de la spec ci-dessus.
- ❌ Aucun `themes` vide : chaque question porte au moins le slug du thème imposé.

---

## Changelog

- **v1.1** (2026-07-27) — Anti-boucle (même passe qu'athlé/tennis v1.1). (a) **Sources** (règle 7) : +CyclingArchives, The Inner Ring. (b) **Anti-répétition renforcée** (règle 5) : rotation OBLIGATOIRE des axes sur le batch (épreuves hors Tour, profils grimpeur/sprinteur/rouleur, géo Colombie/Slovénie/…, époques héroïque→actuel, ~70/30 H-F) + reformulation d'un fait déjà en base = doublon. À republier via /admin/prompts (sport=velo, v1.1) sur DEV puis PROD sur GO. Se combine avec Opus 4.7→4.8 + `EXCLUSION_LIMIT` 120→250 (EF `generate_questions_batch`).
- **v1.0** (2026-07-16) — Epic Cyclisme, Story C1.3. Premier prompt vélo (prompt_templates sport-scopé). Cloné sur `question_gen_tennis_v1.md`, recalibré cyclisme : difficulté vélo (bronze/argent/or), politique H/F ~70/30 (cyclisme féminin valorisé — Longo/Vos/van Vleuten/Vollering/Ferrand-Prévot), sources cyclisme primaires (ASO/UCI/FFC, Grands Tours + Monuments officiels, ProCyclingStats ⭐ anti-hallucination, BikeRaceInfo/FirstCycling), règle 6 **ANTI-PÉRISSABLE renforcée** (Tour 2026 EN COURS = ancrage temporel obligatoire), tags = thèmes seedés migration 0156. Contrat de sortie identique foot/tennis (text ≤140, pool 7-10, explanation 50-300). À publier via /admin/prompts (sport=velo) sur DEV, puis PROD sur GO Louca + content gate (plancher C1.4). Prérequis : `'velo'` dans PROMPT_SPORT_SLUGS (Story C1.2).
