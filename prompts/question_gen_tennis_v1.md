# Prompt versionné `question_gen_tennis_v1.md` (TENNIS)

> Source de vérité humaine pour la génération de questions **tennis** via Claude (Story 20-8, 2026-06-29).
> Le tennis a son **propre prompt actif** en DB (`prompt_templates` sport-scopé : 1 ligne active par sport).
> **Publication** : copie le bloc « TEMPLATE À PUBLIER » ci-dessous dans `/admin/prompts` avec le **sélecteur de sport = tennis**, version `v1.1`. DEV d'abord ; PROD seulement sur GO Louca.
> Tout ajustement = bump version + changelog en bas. Le foot garde son prompt `question_gen_v3.md` inchangé.

## Variables interpolées au runtime

L'EF `generate_questions_batch` remplace ces placeholders avant l'appel Anthropic (identique au foot) :

| Placeholder | Rempli par | Contenu |
|---|---|---|
| `{{sport_slug}}` | EF (body) | `tennis` |
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

## TEMPLATE À PUBLIER (copier tout ce qui suit dans /admin/prompts, sport = tennis)

# Tâche — tennis v1.1

Génère **{{count}} questions** au format **JSON strict** spécifié plus bas, pour le sport `{{sport_slug}}` à la difficulté `{{difficulty}}`.

{{theme_directive}}
# Contexte projet

Clatch est un jeu mobile FR de quiz sportif, ton **chambreur (humour second degré)**. Cette série porte sur le **TENNIS**. Format : `correct_answer + wrong_answers pool` (anti-mémorisation + danger calibré + tagging thématique).

# Règles éditoriales

1. **Ton chambreur dans `explanation`** : second degré, jamais méchant gratuit ni dégradant.
2. **Doctrine "référence nécessaire" IP** : noms d'athlètes informatif autorisé ; marques (raquettes, équipementiers, tournois) nominatif factuel uniquement ; pas d'allusion olympique protégée (loi 1992) — fallback générique en cas de doute.
3. **Politique éditoriale H/F (tennis)** : le tennis a un circuit féminin (**WTA**) à très forte visibilité — vise **~60/40 H/F**, valorise les championnes (Graf, Serena & Venus Williams, Navratilova, Evert, Court, Henin, Mauresmo, Bartoli, Świątek, Sabalenka). L'égalité des primes en Grand Chelem est un sujet valorisant. Zéro question dégradante ou stéréotypée.
4. **Difficulté calibrée** :
   - **bronze** : faits ultra-connus, fan occasionnel. Varie les angles (surface d'un tournoi, nombre de sets gagnants en Grand Chelem hommes, légendes évidentes, vainqueur récent d'un Majeur). Évite le trop trivial répété.
   - **argent** : fan averti. Angles subtils : vainqueur précis d'une finale, année du 1er Chelem d'un joueur, têtes de série, spécificités de surface, joueurs/joueuses cadres d'une époque (Mousquetaires français Tsonga/Monfils/Gasquet/Simon, génération Hingis/Davenport), nombre de semaines n°1.
   - **or** : érudit/expert. Stats précises (titres Masters 1000 exacts, H2H Fedal/Djokovic exact, palmarès en Coupe Davis), records méconnus (Isner–Mahut 70-68 au 5e à Wimbledon 2010, ace le plus rapide, plus jeune/vieux n°1), joueurs hors top stars, éditions anciennes, détails matériel/règles (intro du tie-break, du Hawk-Eye, format no-ad, super tie-break).
5. **ANTI-RÉPÉTITION / VARIÉTÉ OBLIGATOIRE (renforcée)** : ne ressors PAS systématiquement les faits ultra-célèbres (ex : pas 50× « Nadal et la terre battue »). Le bloc « Questions DÉJÀ en base » fourni plus bas liste ce qui existe : génère du contenu **strictement nouveau** (ni ces faits, ni ces réponses, ni une simple reformulation d'un fait déjà présent — même joueur/tournoi/édition sous un autre angle = doublon). **Fais TOURNER les axes sur l'ensemble du batch** :
   - **Surfaces** : alterne terre, gazon, dur, indoor — ne surpondère pas terre battue/Nadal.
   - **Tournois** : au-delà des 4 Majeurs, mobilise Masters 1000, ATP/WTA Finals, Coupe Davis / BJK Cup, tournois 250/500 emblématiques, ex-tournois disparus.
   - **Époques** : mélange l'ère open 70-80 (Borg, Connors, Evert, Navratilova), 90-2000 (Sampras, Graf, Hingis), 2010+ et actuel — pas seulement les stars du moment.
   - **Genre** : ~60/40 H/F (cf. règle 3), valorise le circuit WTA (Graf, Serena, Navratilova, Henin, Świątek, Sabalenka).
   - **Angles rares à privilégier** : seconds rôles, doubles, dates/records méconnus, éditions oubliées, matériel/règles, nationalités hors top-tennis.
6. **Sources** — **Wikipédia ≤25% des questions** (le reste DOIT venir d'ailleurs), **diversifie activement (~40 sources)** et choisis LA PLUS PERTINENTE selon le contexte ; privilégie le primaire/spécialisé à Wikipédia :
   - **Circuits & instances officiels** : ATP Tour (atptour.com) · WTA (wtatennis.com) · ITF (itftennis.com) · FFT (fft.fr).
   - **Grands Chelems (sites officiels)** : Roland-Garros (rolandgarros.com) · Wimbledon (wimbledon.com / AELTC) · US Open (usopen.org / USTA) · Open d'Australie (ausopen.com / Tennis Australia).
   - **Compétitions par équipes & Masters** : Coupe Davis (daviscup.com) · Billie Jean King Cup (billiejeankingcup.com) · Laver Cup · United Cup · ATP Finals / WTA Finals.
   - **Stats / data / archives ⭐ (anti-hallucination — privilégie pour chiffres, H2H, records)** : **Tennis Abstract (tennisabstract.com)** ⭐ (base de matchs Jeff Sackmann + Match Charting Project — l'équivalent de RSSSF au foot) · **Ultimate Tennis Statistics (ultimatetennisstatistics.com)** · datasets Jeff Sackmann `tennis_atp` / `tennis_wta` (GitHub, résultats bruts historiques) · **CoreTennis (coretennis.net)** (palmarès/archives) · classements/archives ATP & WTA · International Tennis Hall of Fame (tennisfame.com) · record books officiels des 4 Majeurs.
   - **Presse spécialisée FR** : L'Équipe (rubrique Tennis) · We Are Tennis (BNP Paribas) · Welovetennis.fr · Tennis Magazine (FR) · Le Figaro Sport Tennis.
   - **Presse spécialisée internationale** : tennis.com · Tennis Channel · ESPN Tennis · BBC Tennis · The Guardian Tennis · Sky Sports Tennis · Eurosport Tennis · The Tennis Gazette · Baseline (magazine RG) · Behind The Racquet.
   - **Tennis féminin (WTA) dédié** : WTA Insider · couvertures WTA officielles (valorise Graf/Serena/Navratilova/Henin/Świątek/Sabalenka).
7. **Pas de question piégeuse** sur « ça dépend » — la bonne réponse doit être incontestable.
8. **Énoncé concis** : 10-140 chars (HARD CAP mobile, > 140 = REJECTED). Variété des formulations : ne commence pas > 40% des questions par « Quel/Quelle ». Alterne : « Qui a/est/détient » · « Combien de » · « Dans quel tournoi / Sur quelle surface » · « Contre qui » · « En quelle année ». ~un quart d'angles narratifs contextualisés (« Wimbledon 2008, près de 5h de finale dans la pénombre : qui s'impose ? »). Esprit chambreur SYSTÉMATIQUE dans `explanation`.
9. **`correct_answer` concis** : 1-80 chars, entité unique. PAS de parenthèses (pas `"Nadal (14 RG)"`), PAS de multi-entités (`"Federer et Nadal"`) — choisis UNE entité ou reformule.
10. **NE PAS générer de questions sur événements futurs incertains** (tournoi/tirage à venir). Pas de `correct_answer` « Aucun » / « À déterminer ».

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

**Le tag qui fait foi est le slug du thème imposé** (cf. directive « THÈME UNIQUE IMPOSÉ ») — utilise-le tel quel sur CHAQUE question. Exemples de slugs tennis transverses (NON exhaustifs — gérés en base, pas figés) :
`grand-chelem`, `legendes-tennis`, `tennis-francais`, `terre-battue`, `rivalites-cultes`, `materiel-regles`, `wimbledon`, `moments-cultes`, `records-stats`, `circuit-atp-wta`.

Règles tagging :
- **1 batch = 1 thème principal.** Tague CHAQUE question avec le slug du thème imposé. Si sous-thème, ajoute le parent.
- 2ᵉ tag transverse autorisé **seulement** s'il décrit réellement la question. Pas de sur-tagging décoratif.

# Format de sortie strict

Retourne **uniquement** un JSON array d'objets (pas de markdown wrapper, pas de préface, pas de commentaire). Aucun champ supplémentaire.

```json
[
  {
    "sport_slug": "tennis",
    "difficulty": "bronze",
    "text": "Sur quelle surface se joue le tournoi de Roland-Garros ?",
    "correct_answer": "Terre battue",
    "wrong_answers": [
      { "text": "Gazon", "danger": 1 },
      { "text": "Dur (hard)", "danger": 1 },
      { "text": "Moquette (indoor)", "danger": 1 },
      { "text": "Béton poreux", "danger": 1 },
      { "text": "Parquet", "danger": 1 },
      { "text": "Terre battue verte (Har-Tru)", "danger": 2 },
      { "text": "Terre battue artificielle", "danger": 2 }
    ],
    "themes": ["terre-battue"],
    "source_url": "https://www.rolandgarros.com",
    "explanation": "L'ocre parisienne, cauchemar des serveurs-volleyeurs et terrain de chasse des glisseurs. Roland = terre battue depuis toujours."
  }
]
```

> ℹ️ Exemple = bronze (7 wrong, 5×d1 + 2×d2). Pour argent : 8 wrong (2×d1+4×d2+2×d3). Pour or : 10 wrong (2×d1+4×d2+4×d3). **Suis la table de distribution.**

{{exclusion_block}}
# Exclusions strictes

- ❌ Aucune question sur paris sportifs, cotes, scandales liés aux paris.
- ❌ Aucune question sur drama personnel non sportif.
- ❌ Aucun champ JSON en dehors de la spec ci-dessus.
- ❌ Aucun `themes` vide : chaque question porte au moins le slug du thème imposé.

---

## Changelog

- **v1.1** (2026-07-27) — Anti-boucle. (a) **Sources élargies** (règle 6) : ajout datasets Jeff Sackmann `tennis_atp`/`tennis_wta` (GitHub), CoreTennis, record books officiels des 4 Majeurs. (b) **Anti-répétition renforcée** (règle 5) : rotation OBLIGATOIRE des axes sur le batch (surfaces, tournois au-delà des Majeurs, époques ère open→actuel, ~60/40 H/F, angles rares) + définition élargie du doublon (reformulation d'un fait déjà en base = doublon). À republier via /admin/prompts (sport=tennis, v1.1) sur DEV puis PROD sur GO. Se combine avec l'upgrade modèle Opus 4.7→4.8 et `EXCLUSION_LIMIT` 120→250 (EF `generate_questions_batch`).
- **v1.0** (2026-06-29) — Story 20-8. Premier prompt tennis (prompt_templates sport-scopé). Calibration difficulté tennis, politique H/F ~60/40 (WTA valorisé), sources tennis primaires (ATP/WTA/ITF, 4 Majeurs officiels, Tennis Abstract ⭐ anti-hallucination, Ultimate Tennis Statistics), tags = thèmes seedés migration 0134. Contrat de sortie identique au foot (text ≤140, pool 7-10, explanation 50-300). À publier via /admin/prompts (sport=tennis) sur DEV, puis PROD sur GO Louca + content gate ≥42 questions.
