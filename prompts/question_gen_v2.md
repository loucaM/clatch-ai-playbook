# Prompt versionné `question_gen_v2.md`

> Source de vérité pour génération de questions via Claude — **format pivot 2026-05-05** (Story 2.1b).
> Utilisable en Phase 1 Pro Max browser (copy-paste) ET Phase 2 EF Anthropic Batch API (interpolation runtime).
> Tout ajustement = bump version + changelog en bas.

## Contexte projet

SPQ est un jeu mobile FR de quiz sportif, ton **chambreur (humour second degré)**, V1 **foot uniquement** (calé Coupe du Monde 2026 juin-juillet). Pivot 2026-05-05 : nouveau format `correct_answer + wrong_answers pool` (anti-mémorisation FR84 + danger calibré FR85 + tagging multi-thèmes FR86 via table `themes`).

## Tâche

Génère **{{count}} questions** au format **JSON strict** spécifié plus bas, pour le sport `{{sport_slug}}` à la difficulté `{{difficulty}}`.

> ℹ️ **Variables interpolées au runtime** par l'EF `generate_questions_batch` (Phase 2) : `{{sport_slug}}`, `{{difficulty}}`, `{{count}}`. En Phase 1 Pro Max browser, remplace manuellement avant de paster dans claude.ai.

## Règles éditoriales

1. **Ton chambreur dans `explanation`** : second degré, pas misogyne, pas méchant gratuit (cf. PRD Politique éditoriale).
2. **Doctrine "référence nécessaire" IP** :
   - Athlètes : usage informatif factuel autorisé (comme tout média sportif).
   - Marques sportives (clubs, tournois) : usage nominatif informatif uniquement, pas de logos décrits.
   - Anneaux olympiques + nom "Olympique/JO" : protection ultra-stricte (loi 1992 art. L141-5) → fallback noms génériques en cas de doute.
3. **Politique éditoriale H/F (foot V1)** : ~80/20 H/F (couvrir CdM féminine + D1 Arkema + équipe France F). **Zéro question dégradante** envers une joueuse ou compétition féminine. Pas de stéréotype de genre.
4. **Difficulté calibrée (RENFORCÉE v2.4 — pousse le curseur d'exigence)** :
   - **bronze** : faits ultra-connus mais varie les angles. Évite les questions trop triviales du type "Qui a gagné la CdM X ?" — préfère des angles plus spécifiques tout en restant accessibles : lieu d'un match, score d'une finale célèbre, joueur clé d'un événement, anecdote très médiatisée, surnom de club populaire, faits récents (3-5 dernières années) sur les stars actuelles. Reste accessible à un fan occasionnel.
   - **argent** : connaissance fan averti. Pas juste "qui a gagné" mais des angles plus subtils : 1er buteur d'une finale, minute d'un but célèbre, contre quelle équipe une demi-finale, scores exacts de finales clés, transferts marquants avec montants/clubs précédents, palmarès d'une décennie, joueurs cadres d'une époque (ex: ailier de Manchester United 2008-2013), entraîneurs des sacres récents. Filtre les fans casuals.
   - **or** : fan érudit/expert. Statistiques précises (nombre exact de buts, minutes jouées, passes décisives en finale), records méconnus, joueurs des grands clubs hors top stars (le n°9 d'Inter Milan 1989, l'arrière gauche de Liverpool 2005), anecdotes pointues (qui a sorti telle équipe en 1/8 d'une compétition spécifique), entraîneurs adjoints célèbres, derbies historiques méconnus (Roma-Lazio 1979, Boca-River 1962), tactiques d'époque (catenaccio, total football, gegenpressing innovations), records africains/sud-américains spécifiques (CAN 1968, Copa Libertadores 1976), stadia particuliers, anciennes compétitions (Coupe Intercontinentale, Coupe des Coupes, Mitropa Cup).
5. **Sources obligatoires — pool ÉLARGI v2.6 (~50 sources)** : varie activement les URL — **Wikipédia ne doit pas représenter plus de 40% des sources**. Pool autoritaire à utiliser :

   **🇫🇷 France :** L'Équipe (lequipe.fr) · So Foot (sofoot.com) · France Football (francefootball.fr) · Footmercato (foot01.com) · L'Équipe TV
   
   **🌐 Officiel international :** FIFA.com · UEFA.com · CAFonline.com (CAN) · Conmebol.com (Copa America/Libertadores)
   
   **📊 Stats avancées :** Transfermarkt (transferts/effectifs historiques) · FBref (xG/avancé) · WhoScored.com · Sofascore.com · Opta Stats · StatsBomb · Squawka · Understat (xG) · Soccerwiki · Footballdatabase.eu · Soccerway
   
   **📜 Archives historiques :** **RSSSF.com** ⭐ (Rec.Sport.Soccer Stats Foundation — référence anti-hallucination pour records anciens 1900+) · IFFHS.com (records mondiaux) · ESPN Stats and Info
   
   **🇬🇧 Angleterre :** BBC Sport · The Guardian Football · The Athletic · Mirror Football · Daily Mail Sport · The Sun Sport · Football365 · Sky Sports · 90min.com
   
   **🇺🇸 USA :** ESPN FC · Bleacher Report
   
   **🇪🇸 Espagne :** Marca · AS · Sport · Mundo Deportivo
   
   **🇮🇹 Italie :** Gazzetta dello Sport · Corriere dello Sport · Football Italia
   
   **🇩🇪 Allemagne :** Bild · Kicker
   
   **🇵🇹 Portugal :** A Bola · Record
   
   **🇧🇷 Brésil :** Globo Esporte · LANCE! · Folha de São Paulo
   
   **🇦🇷 Argentine :** Olé · Clarín · La Nación
   
   **🇳🇱 Pays-Bas :** Voetbal International · Trouw
   
   **🇸🇪 Suède :** Sportbladet · Aftonbladet
   
   **🇹🇷 Turquie :** Hürriyet Sport
   
   **🌍 Afrique anglo :** KingFut.com (Égypte) · Soccernet Nigeria · BBC Sport Africa
   
   **👩 Foot féminin (sources dédiées) :** BBC Sport Women's Football · The Guardian Women's Football · HerFootballHub · WSL.com (officiel Angleterre) · NWSL.com (officiel USA) · FIFA Women's World Cup · UEFA Women's · Equipo Femenino (Espagne)
   
   **🏟️ Sites officiels clubs :** psg.fr, realmadrid.com, fcbarcelona.com, manutd.com, ajax.nl, slbenfica.pt, etc.

   **Règle de diversification stricte** : choisis la source la plus pertinente selon le contexte (ex : question sur Ajax → ajax.nl ou Voetbal International ; question sur Galatasaray → Hürriyet Sport ; question sur stats avancées Real Madrid → FBref ou WhoScored ; question féminine sur Putellas → Equipo Femenino ; record CdM 1958 → RSSSF.com).
6. **Pas de question piégeuse** sur une réponse "ça dépend" — la bonne réponse doit être incontestable.
7. **Énoncé concis** : max 200 chars, min 10 chars.

7bis. **Variété des formulations OBLIGATOIRE (NEW v2.7)** — ne pas commencer >40% des questions par "Quel/Quelle". Alterne **activement** les structures interrogatives + angles narratifs :

   **Structures interrogatives** (varie) :
   - "Qui a..." / "Qui est..." / "Qui détient..."
   - "Quel/Quelle..." (à utiliser <40% du temps)
   - "Combien de..." / "À combien..."
   - "Dans quel..." / "Dans quelle..."
   - "Contre quelle équipe..." / "Face à quel adversaire..."
   - "À quelle minute..." / "En quelle année..." / "Lors de quelle édition..."
   - "Sur quel terrain..." / "Dans quel stade..."
   - "Avec quel club..." / "Sous quel maillot..."
   - "Pour quelle raison..." / "Suite à quel événement..."

   **Angles narratifs / contextualisés** (utilise environ un quart des questions) :
   - "L'Histoire retient que..." (suivi d'une question)
   - "On l'appelle 'X' (surnom célèbre). Sa véritable identité ?"
   - "Surnommé 'X', ce joueur a marqué l'Histoire en..." → identifier le joueur
   - "Cette finale est entrée dans la légende. À quelle équipe revient le trophée ?"
   - "Devant 80 000 spectateurs, c'est lui qui plante le but vainqueur..." → qui ?
   - Questions au présent narratif ("Saint-Denis, 12 juillet 1998. À la 27e minute, qui ouvre le score ?")
   - Questions inversées ("Ce club portugais, plus ancien que ses rivaux, a vu Cristiano Ronaldo débuter pro en 2002. Lequel ?")
   - Anecdotes ("Il a refusé un transfert à Manchester United pour rester fidèle à son club. Qui ?")

   **Évite** :
   - Questions trop sèches/comptables sans contexte ("Combien de joueurs sur le terrain ?" → trivial bronze, à varier)
   - Toujours commencer par "Quel pays/club/joueur..." (au-delà de 40% des questions = monotone)
   - Reformulations identiques entre questions (varier le verbe principal)

   **Esprit chambreur** : injecte une légère touche d'humour second degré DANS le `text` quand naturel (pas forcé), et SYSTÉMATIQUEMENT dans `explanation`.
8. **`correct_answer` concis** : max 80 chars, min 1 char, non vide. **OBLIGATOIRE — entité unique sans qualifier** : pas de parenthèses (ex : pas `"Mané (tab)"` ni `"Mandžukić (csc)"` ni `"Messi (2 buts)"`). Précise dans `explanation` à la place. Pas non plus de réponse multi-entités (`"Messi et Di María"`) — choisir UNE entité ou reformuler la question.
9. **NE PAS générer de questions sur des événements futurs incertains** : si le tirage CdM 2026 n'a pas eu lieu à la date de référence, ne pose PAS la question "Dans quel groupe est la France ?". De même pour les autres événements à venir avec résultats inconnus. Choisis des sujets historiques ou actuels factuellement résolus.
10. **PAS de réponse "Aucun" / "Le tirage n'a pas eu lieu" / "À déterminer"** comme `correct_answer` — reformule la question pour obtenir une entité concrète.

## Format `wrong_answers` (NEW pivot 2026-05-05, range étendu 2026-05-07)

C'est un **pool de 7 à 10 distractors** (pas un array fixe de 3 comme l'ancien format). Le client tire 3 wrong au hasard à chaque rendu de la question → anti-mémorisation. Plus le pool est gros, plus les combinaisons sont diverses : C(7,3)=35, C(10,3)=120 combinaisons possibles. Chaque distractor a un **niveau de `danger` (1-3)** qui calibre la difficulté ressentie.

### Définitions `danger`

- **danger = 1 (décor)** : distractor évident, peu probable que l'utilisateur s'y trompe (ex : un nom hors-sport, ou une date manifestement fausse).
- **danger = 2 (plausible)** : distractor crédible mais inexact, peut tromper un fan moyen.
- **danger = 3 (piège fort)** : distractor très proche de la bonne réponse, piège un fan averti (ex : confusion entre deux finales, deux joueurs d'une même époque, deux dates voisines).

### Distribution attendue par difficulté

| Difficulté | Total wrong | danger=1 | danger=2 | danger=3 |
|---|---|---|---|---|
| **bronze** | 7 | 5 | 2 | 0 |
| **argent** | 8 | 2 | 4 | 2 |
| **or** | 10 | 2 | 4 | 4 |

> ⚠️ **Suis cette distribution strictement** — elle garantit la calibration de difficulté ressentie. Si la difficulté est ambiguë sur un fait, choisis le niveau supérieur (préférer or à argent, argent à bronze).

### `wrong_answers[i]` schema

```json
{ "text": "Distractor concis ≤80 chars", "danger": 1 }
```

- `text` : string non vide, ≤80 chars.
- `danger` : entier ∈ {1, 2, 3}.
- **Aucun champ supplémentaire**.

### Règle anti-collision

Les `wrong_answers[*].text` ne doivent **jamais** être égaux ni quasi-égaux à `correct_answer` (case-insensitive, ignore accents). Pas non plus de doublons internes au pool.

## Tags — slugs `themes` table V1 (autoritaire)

Chaque question doit avoir **`themes` = array de ≥1 slug**, choisis EXCLUSIVEMENT dans cette liste (autres slugs = rejet par EF `import_questions`) :

### Foot V1 — universels evergreen (8)

`mercato` · `tactiques` · `legendes` · `buts` · `ligue1` · `c1` · `euro` · `bleus-de-france`

### Foot V1 — compétitions clubs additionnelles (2, NEW v2.3)

`coupe-de-france` · `ligue-europa` (C3 UEFA — Sevilla, Mourinho/Roma 2022)

### Foot V1 — féminin (3, NEW v2.3 — parité H/F PRD)

`bleues` (équipe de France F — Renard, Henry sélectionneuse, Katoto, Cascarino, Bacha) · `euro-feminin` (Angleterre 2022, Espagne 2025) · `cdm-feminine` (sub-theme `mondiaux` — USA hégémonie, Espagne 2023, Hegerberg)

### Foot V1 — hub Mondiaux (5, v2.9)

`mondiaux` (parent générique) · `cdm-1998` · `cdm-2006` (Italie, Zidane coup de boule) · `cdm-2018` · `cdm-2022`

> ℹ️ v2.9 : `cdm-2026` et `cdm-2026-groupe-france` injectés via `extra_instruction` au runtime (ADR-011 mono-thème).

### Foot V1 — hub CAN — Coupe d'Afrique des Nations (1, v2.9)

`can` (parent générique)

> ℹ️ v2.9 : slugs `can-*` spécifiques injectés via `extra_instruction` au runtime (ADR-011 mono-thème).

### Foot V1 — transverses thématiques (5, NEW v2.3)

`gardiens` (Yashin, Buffon, Casillas, Neuer, Courtois, Lloris, Maignan, Donnarumma, Bounou) · `entraineurs` (Guardiola, Mourinho, Ancelotti, Klopp, Wenger, Ferguson, Sacchi, Bielsa, Trapattoni, Lobanovskyi) · `derbies` (Clásico, Manchester, Roma, Madrid, Milan, Buenos Aires Boca-River, Istanbul Galata-Fener, Pays-Bas Ajax-Feyenoord) · `transferts-records` (Neymar 222M€, Mbappé, Haaland, Trezeguet, Lukaku, Pogba, Bale) · `stades-mythiques` (Maracanã, Wembley, Bernabéu, Camp Nou, San Siro, Old Trafford, Anfield, Signal Iduna Park)

### Foot V1 — citations cultes (1, NEW v2.5)

`citations-fun` — phrases marquantes / anecdotes drôles : Cantona "When the seagulls follow the trawler", Mourinho "specialist in failure" / "the cup of the cup of tea", Aimé Jacquet "qu'est-ce que t'as Karembeu" / "tu trembles carcasse", Beckenbauer "wir sind Weltmeister", Simeone "huevos", Klopp "we are mentality monsters", Guardiola "the rainbow nation", Wenger "I didn't see it", Ferguson hairdryer treatment, Mathieu Valbuena phrases cultes, Ribéry "la routourne", Cruyff "playing football is very simple", Pelé "el rey", Maradona "tarjeta amarilla? Yo soy de Buenos Aires", Pochettino, Domenech "non" sur le plateau, etc.

### Règles tagging (v2.9 évergreen)

- Une question peut avoir **plusieurs thèmes** (ex : `["bleus-de-france", "cdm-1998", "mondiaux", "legendes"]`). C'est encouragé pour enrichir le tagging.
- Si tu cites un sous-thème (ex : `cdm-2018`), inclus aussi le parent (`mondiaux`) pour permettre browsing multi-niveau côté UI.
- Si l'`extra_instruction` précise un thème (ex : "Toutes les questions doivent inclure le tag : cdm-2026"), tagger TOUTES les questions avec ce slug.

> ℹ️ v2.9 ADR-011 : les coverages thématiques hardcodées sont supprimées. Le thème ciblé est injecté via `extra_instruction` au runtime par la route `/api/admin/batches` (workflow mono-thème). 1 batch = 1 thème principal.

## Format de sortie strict

Retourne **uniquement** un JSON array d'objets (pas de markdown wrapper, pas de préface, pas de commentaire). Aucun champ supplémentaire au-delà de ceux listés.

```json
[
  {
    "sport_slug": "foot",
    "difficulty": "argent",
    "text": "Quel joueur a inscrit le doublé en finale de la CdM 1998 contre le Brésil ?",
    "correct_answer": "Zinédine Zidane",
    "wrong_answers": [
      { "text": "Emmanuel Petit",     "danger": 3 },
      { "text": "Youri Djorkaeff",    "danger": 2 },
      { "text": "Thierry Henry",      "danger": 2 },
      { "text": "Christophe Dugarry", "danger": 1 },
      { "text": "Bernard Diomède",    "danger": 1 }
    ],
    "themes": ["bleus-de-france", "cdm-1998", "mondiaux", "legendes"],
    "source_url": "https://fr.wikipedia.org/wiki/Finale_de_la_Coupe_du_monde_de_football_1998",
    "explanation": "Zizou plante deux têtes en première mi-temps. Petit ferme le score à 90+3. Brésil KO."
  }
]
```

## Exclusions strictes

- ❌ Aucune question sur paris sportifs, cotes, scandales liés aux paris.
- ❌ Aucune question sur drama personnel non sportif (vie privée, scandales judiciaires hors carrière).
- ❌ Aucune marque ou produit en dehors de l'usage strictement informatif.
- ❌ Aucun champ JSON en dehors de la spec ci-dessus (`tags`, `choices`, `correct_index` legacy = exclus).
- ❌ Aucun slug `themes` hors de la liste autoritaire ci-dessus.

---

## Changelog

- **v2.9** (2026-05-15) : ADR-011 évergreen mono-thème. Retrait total section "Coverage cible" (≥15% cdm-2026, ≥10% premier-league, etc.). Retrait slugs championnats européens (premier-league, liga, serie-a, bundesliga, eredivisie, primeira-liga, super-lig) + cdm-2026 + cdm-2026-groupe-france + can-* spécifiques du corps du prompt — injectés via `extra_instruction` au runtime. 1 batch = 1 thème principal. Compact sources section. Prompt : 3000-7000 chars. Story 8.0e.
- **v2.8** (2026-05-13) : rename de l'ancien ton → "chambreur" dans tous les passages éditoriaux (`Ton`, `Doctrine`, `Esprit`) — concurrent trademark + brand differentiation. Aligné avec `mobile/supabase/functions/_shared/prompt.ts` qui injecte "chambreur" dans le string envoyé à Claude. Sémantique inchangée (humour second degré, second degré pas méchant). Aucun changement aux règles 1-10 ni au format JSON ni aux slugs `themes`.
- **v2.7** (2026-05-07) : variété de formulations obligatoire. Plafond 40% sur structures "Quel/Quelle". Push 20-30% angles narratifs/contextualisés (présent narratif, anecdotes, surnoms inversés). Touche chambreur humour second degré dans `text` quand naturel.
- **v2.6** (2026-05-07) : pool sources élargi à ~50 sources (15+ catégories : stats avancées, archives historiques RSSSF/IFFHS, presse internationale 6 pays additionnels, sources féminin dédiées, sources africaines anglo). Wikipédia ≤40% strict. Story 2.1c re-batch féminin + M3 prep round 2.
- **v2.5** (2026-05-07) : ajout slug `citations-fun` + boost coverage féminin 10% → 15%. EF generate_questions_batch supporte `body.extra_instruction` (override prompt pour batchs ciblés). CLI script supporte env `EXTRA_INSTRUCTION`. Total 42 slugs autoritaires. Story 2.1c sweep M2 + boost féminin.
- **v2.4** (2026-05-07) : difficulté renforcée (curseur d'exigence poussé sur argent/or — angles subtils, stats précises, records méconnus, derbies historiques, tactiques d'époque, joueurs hors top stars). Sources diversifiées (10+ sources au-delà de Wikipédia). Quota cdm-2026 baissé 30% → 15% (saturation observée M1). Story 2.1c milestone M3 prep.
- **v2.3** (2026-05-07) : +11 thèmes (`coupe-de-france`, `ligue-europa`, `bleues`, `euro-feminin`, `cdm-feminine`, `cdm-2006`, `gardiens`, `entraineurs`, `derbies`, `transferts-records`, `stades-mythiques`). Total 41 slugs autoritaires. Story 2.1c milestone M2 prep round 2.
- **v2.2** (2026-05-07) : 7 nouveaux championnats européens autoritaires (`premier-league`, `liga`, `serie-a`, `bundesliga`, `eredivisie`, `primeira-liga`, `super-lig`) + coverages élargies (top 5 + hors top 5 + stars modernes 15% + joueurs rétro 10%). Renforcement règles `correct_answer` (entité unique sans parenthèses) + interdiction questions sur événements futurs incertains. Story 2.1c milestone M2 prep.
- **v2.1** (2026-05-07) : extension pool wrong_answers 5-7 → **7-10** (anti-mémorisation FR84 renforcée). Distributions ajustées : bronze 7 (5×d1+2×d2), argent 8 (2×d1+4×d2+2×d3), or 10 (2×d1+4×d2+4×d3). Story 2.1b post-livraison patch.
- **v2.0** (2026-05-07) : pivot format `correct_answer + wrong_answers pool ≥5 avec danger 1-3`, tags via `themes` table (slugs autoritaires : universels + Mondiaux + CAN), variables interpolées `{{sport_slug}}/{{difficulty}}/{{count}}`. Story 2.1b.
- **v1.0** (2026-05-04) : version initiale (legacy `choices`/`correct_index`). Story 7.10. Conservée pour audit.
