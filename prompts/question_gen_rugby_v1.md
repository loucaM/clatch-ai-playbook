# Prompt versionné `question_gen_rugby_v1.md` (RUGBY)

> Source de vérité humaine pour la génération de questions **rugby** via Claude (Epic Rugby, 2026-08-31).
> Le rugby a son **propre prompt actif** en DB (`prompt_templates` sport-scopé : 1 ligne active par sport).
> **Publication** : copie le bloc « TEMPLATE À PUBLIER » ci-dessous dans `/admin/prompts` avec le **sélecteur de sport = rugby**, version `v1.2`. DEV d'abord ; PROD seulement sur GO Louca.
> 🔴 **Incident v1.0 (2026-09-01)** : la première version publiée s'arrêtait à la règle « Sources » — il MANQUAIT la section `# Format wrong_answers` (table de distribution), la règle d'intégrité de l'énoncé, la section Tags et l'exemple JSON de sortie. Résultat : le batch `msgbatch_013HbePs9BFvEEk7QjWqwWHy` a rendu 15 questions à **3 distracteurs** au lieu de 7-10 → **15/15 rejetées** à la validation (`wrong_answers: Expected >=7 but received 3`), zéro insérée, zéro pollution. Leçon : un prompt de sport se relit **section par section contre celui du basket**, pas seulement sur ses règles éditoriales. La v1.1 les rétablit.
> 🔴 **Incident v1.1 (2026-09-01)** : batch `msgbatch_01KNpuLgXFLYrn6JFUHtdS6C` — 15/15 insérées, faits justes, pools 7-10 conformes… mais **0 % d'accents** (« annee », « Nouvelle-Zelande »), contre 79-100 % sur les autres sports. Cause : l'exemple JSON du template était écrit sans accents et le modèle imite l'exemple avant d'obéir aux règles. La v1.2 réaccentue l'exemple ET ajoute la règle 13. Leçon générale : **dans un prompt, l'exemple pèse plus lourd que la consigne** — il doit être exemplaire, pas juste correct.
> ⚠️ **Prérequis** : `'rugby'` doit être présent dans `PROMPT_SPORT_SLUGS` (`admin/lib/schemas/create-prompt.ts`) — fait dans ce lot — sinon le `z.enum` du form de publication refuse le sport.
> ⚠️ **Prérequis EF** : `'rugby'` doit être dans le picklist Valibot **ET** dans l'enum JSON Schema de `mobile/supabase/functions/_shared/schema.ts` — fait dans ce lot — **et les edge functions `generate_questions_batch` / `poll_batch_results` doivent être REDÉPLOYÉES**. Le repo à jour ne suffit pas : l'incident basket du 2026-08-28 (123 questions orphelines) venait d'un bundle PROD resté vieux alors que le fichier était correct.
> ⚠️ **Différence avec les épics précédents** : le rugby est passé `sports.status = 'live'` (migration 0275, arbitrage Louca). Une question rugby **validée** tombe donc dans le pool du **Daily mixte**, `rugby_enabled` OFF ou non. Le robinet, c'est la validation admin — une question laissée en `pending_review` n'est jamais servie.
>
> Le foot garde son `question_gen_v3.md`, le tennis son `question_gen_tennis_v1.md`, le vélo son `question_gen_velo_v1.md`, l'athlé son `question_gen_athletisme_v1.md`, le basket son `question_gen_basket_v1.md`.

## Variables interpolées au runtime

L'EF `generate_questions_batch` remplace ces placeholders avant l'appel Anthropic (identique foot/tennis/vélo/athlé/basket) :

| Placeholder | Rempli par | Contenu |
|---|---|---|
| `{{sport_slug}}` | EF (body) | `rugby` |
| `{{difficulty}}` | EF (body) | `bronze` \| `argent` \| `or` |
| `{{count}}` | EF (body) | 1-50 |
| `{{theme_directive}}` | EF | Bloc « THÈME UNIQUE IMPOSÉ » (label + description) — vide si pas de thème |
| `{{exclusion_block}}` | EF (service-role) | Bloc « Questions DÉJÀ en base » (theme+difficulty) — vide si N=0 |

## Thèmes disponibles (migration 0275, 8 feuilles)

`coupe-du-monde-rugby` · `tournoi-six-nations` · `xv-de-france` · `top-14-rugby-francais` · `europe-des-clubs-rugby` · `legendes-rugby` · `records-rugby` · `regles-rugby`

## Contrat de sortie (caps DURS — `_shared/schema.ts`, sinon REJECTED à l'import)

- `text` : **10-140 chars** (HARD CAP mobile ; sweet spot 80-100).
- `correct_answer` : 1-80 chars, **entité unique sans parenthèses ni multi-entités** (une marque « 15 points » ou « 1995 » est une entité valide).
- `wrong_answers` : pool 7-10 distractors `{ "text" ≤80, "danger": 1|2|3 }`.
- `explanation` : 50-300 chars, ton chambreur.
- `source_url` : URI 10-500.
- `themes` : ≥1 slug (le thème imposé), jamais vide.

---

## TEMPLATE À PUBLIER (copier tout ce qui suit dans /admin/prompts, sport = rugby)

# Tâche — rugby v1.2

Génère **{{count}} questions** au format **JSON strict** spécifié plus bas, pour le sport `{{sport_slug}}` à la difficulté `{{difficulty}}`.

{{theme_directive}}
# Contexte projet

Clatch est un jeu mobile FR de quiz sportif, ton **chambreur (humour second degré)**. Cette série porte sur le **RUGBY** (Coupe du Monde, Tournoi des 6 Nations, XV de France, Top 14 et rugby français, Champions Cup, rugby féminin, histoire du jeu). Format : `correct_answer + wrong_answers pool` (anti-mémorisation + danger calibré + tagging thématique).

# Règles éditoriales

1. **Ton chambreur dans `explanation`** : second degré, jamais méchant gratuit ni dégradant. Registre rugby : la mêlée, la touche, le ruck, la chandelle, le cadrage-débordement, l'en-avant, le drop, la transformation, le Bouclier de Brennus, la troisième mi-temps, le paquet d'avants, la ligne d'avantage.

2. **Doctrine « référence nécessaire » IP** : noms de joueuses/joueurs, clubs, fédérations et compétitions = informatif factuel autorisé (« lors de la finale 2011 », « au Tournoi 2022 », « champions d'Europe 2018 »). **MAIS** : n'implique JAMAIS un partenariat officiel, n'utilise PAS logos ni marques comme label commercial ; en cas de doute (équipementiers, naming de stades, sponsors du Tournoi), reste factuel ou fallback générique.

3. **Politique éditoriale H/F (rugby)** : le rugby féminin compte (Coupes du Monde, Tournoi féminin, les Bleues, le Sevens olympique). Vise une **part significative de questions féminines (~30 %)**, zéro question dégradante ou stéréotypée. À mobiliser : Jessy Trémoulière, Gaëlle Hermet, Romane Ménager, Portia Woodman, Sarah Hunter, Emily Scarratt, Ruby Tui, Fanny Horta, Safi N'Diaye, Caroline Drouin.

4. **XV et XIII, Sevens** : le rugby à XV est le cœur (~80 % du volume). Le **rugby à XIII** (Coupe du Monde XIII, les Dragons Catalans, l'histoire du « rugby interdit » sous Vichy) et le **rugby à 7** (l'olympisme depuis Rio 2016, les World Series, Antoine Dupont médaillé d'or à Paris 2024) sont des angles rares à mobiliser — **mais précise TOUJOURS la discipline dans l'énoncé** quand ce n'est pas le XV (« en rugby à XIII », « en rugby à 7 ») : la taxonomie ne les sépare pas, l'énoncé doit lever l'ambiguïté sous peine de rendre la question fausse.

5. **Difficulté calibrée** :
   - **bronze** : faits ultra-connus, fan occasionnel. Varie les angles (nombre de points d'un essai, nombre de joueurs sur le terrain à XV, pays hôte d'une CdM récente, nom du trophée du championnat de France, poste d'Antoine Dupont, couleur du maillot des All Blacks). Évite le trop trivial répété.
   - **argent** : fan averti. Angles subtils : finaliste d'une CdM précise, vainqueur d'un Grand Chelem, palmarès d'un club français, campagne des Bleus, capitaine d'une édition, meilleur réalisateur d'un Tournoi, club d'un joueur à une saison donnée.
   - **or** : érudit/expert. Marques exactes (le score de la finale 1995, le nombre de points de Jonny Wilkinson en 2003, les 6 essais de Marc Ellis en 1995), éditions anciennes (premiers Tournois, la tournée de 1905, le rugby aux JO 1900-1924), joueurs hors stars, détails **règlement** (l'en-avant volontaire, le carton rouge de 20 minutes, la règle du 50:22, la mêlée simulée, les 22 mètres, la valeur historique de l'essai passée de 3 à 4 puis 5 points).

6. **ANTI-RÉPÉTITION / VARIÉTÉ OBLIGATOIRE** : ne ressors PAS que Dupont, Wilkinson et les All Blacks. Le bloc « Questions DÉJÀ en base » fourni plus bas liste ce qui existe : génère du contenu **strictement nouveau** (ni ces faits, ni ces réponses, ni de simple reformulation — un même record/joueur/édition sous un autre angle compte comme un doublon). **Fais TOURNER les axes sur l'ensemble du batch** :
   - **Compétitions** : alterne Coupe du Monde, Tournoi, XV de France, Top 14 / Pro D2, Champions Cup, tournées d'été, Sevens. Vise ≤ 40 % de Coupe du Monde sur un batch générique.
   - **Géographie** : ne surpondère pas la France et la Nouvelle-Zélande — mobilise l'Afrique du Sud, l'Angleterre, l'Irlande, le pays de Galles, l'Écosse, l'Italie, l'Argentine, l'Australie, les Fidji, le Japon (l'exploit de 2015 contre les Springboks, l'édition 2019).
   - **Époques** : mélange les ères (le rugby amateur d'avant 1995, les années Blanco, la génération 1999, l'ère professionnelle, les 2010, aujourd'hui), pas seulement l'actu.
   - **Genre** : ~30 % féminin (cf. règle 3).
   - **Angles rares à privilégier** : le Grand Chelem 1968, la demi-finale 1999 contre les All Blacks, l'essai du bout du monde (1994), Limoges du rugby — Agen, Béziers et leurs Brennus des années 70-80, le rugby à XIII interdit sous Vichy, l'Italie entrée dans le Tournoi en 2000, les Fidji championnes olympiques de Sevens.

7. **ANTI-PÉRISSABLE (CRITIQUE)** : bannis tout superlatif battable ou ancré au présent (« le meilleur marqueur ACTUEL », « le tenant du titre », « le capitaine en poste »). **Ancre TOUJOURS dans le temps** : « lors de la finale de la Coupe du Monde 2011 », « au Tournoi 2022 », « sur la saison 2013-2014 », « aux JO de Paris 2024 ». Une question doit rester vraie dans 10 ans. **Aucun résultat d'une compétition non encore disputée.** Pas de « club actuel » d'un joueur en activité (les transferts périment — ancre par saison). Attention particulière aux **records de sélections et de points** : ils tombent, ancre-les (« à sa retraite internationale en 2011 », « au terme de la CdM 2023 »).

8. **Sources** — **Wikipédia ≤25 %** (le reste DOIT venir d'ailleurs), **diversifie activement (~30 sources)** et choisis LA PLUS PERTINENTE ; privilégie le primaire/spécialisé.
   - **Instances officielles** : **World Rugby (world.rugby)** ⭐ (compétitions internationales, CdM, règlement, classement) · **Six Nations (sixnationsrugby.com)** ⭐ (le Tournoi, palmarès) · **FFR (ffr.fr)** (XV de France, rugby français, les Bleues) · **LNR (lnr.fr)** (Top 14, Pro D2, Brennus) · **EPCR (epcrugby.com)** (Champions Cup, Challenge Cup) · fédérations nationales (RFU, SARU, NZR, IRFU, WRU).
   - **Presse spécialisée** : **Midi Olympique** ⭐ · **Rugbyrama** · **L'Équipe (rubrique rugby)** · **RugbyPass** · **The Rugby Paper** · **ESPN Scrum**.
   - **Archives / statistiques** : **ESPN Statsguru (rugby)** ⭐ (feuilles de match, records ancrés) · **INA** (archives audiovisuelles françaises) · **Rugby Archive**.
   - Une source par question, la plus proche du fait. Un fait de 1968 ne se source pas sur un site d'actualité de 2026.

9. **INTÉGRITÉ DE L'ÉNONCÉ (RÈGLE DURE — priorité absolue ; tout manquement rend la question INUTILISABLE)** :
   - **Vérité littérale de l'énoncé** : la `correct_answer` doit satisfaire CHAQUE prédicat de l'énoncé — nationalité, poste, année, nombre, club, compétition, tour, édition. Si l'énoncé dit « Quel troisième ligne… », la réponse EST troisième ligne ; s'il dit « Grand Chelem », c'est bien un Grand Chelem et pas un simple Tournoi gagné ; s'il dit « qui bat X en finale », la réponse est le VAINQUEUR. **Interdiction absolue de fabriquer une fausse prémisse** pour tendre un piège.
   - **Le piège vit UNIQUEMENT dans les `wrong_answers`** (les leurres danger=3), JAMAIS dans l'énoncé ni dans la `correct_answer`.
   - **L'`explanation` CONFIRME la `correct_answer`** : elle ne nomme JAMAIS une autre entité comme vraie réponse, ne se corrige pas, ne doute pas. Mots BANNIS dans l'explication : « piège », « à reformuler », « mea culpa », « en fait c'est… », « aucun match », « oups », « on s'est trompé », « correction : ». **Si tu ressens le besoin d'écrire l'un de ces mots, l'énoncé est CASSÉ → jette-le et génères-en un autre.**
   - **Abstention si incertain** : si tu n'es pas certain du fait (nom, date, score, nationalité, club à une saison donnée), NE génère PAS la question. **N'invente JAMAIS un résultat ni une statistique.**
   - **Pas de « ça dépend »** : la bonne réponse doit être unique et incontestable.

10. **Énoncé concis** : 10-140 chars (HARD CAP mobile, > 140 = REJECTED). Variété : ne commence pas > 40 % des questions par « Quel/Quelle ». Alterne : « Qui a soulevé / inscrit » · « Combien de points » · « En quelle année » · « Dans quel club » · « Face à qui » · « À quelle minute ». ~un quart d'angles narratifs contextualisés (« Cardiff, 2007 : les Bleus sortent les All Blacks en quart. Qui inscrit l'essai de la gagne ? »). Esprit chambreur SYSTÉMATIQUE dans `explanation`.

11. **`correct_answer` concis** : 1-80 chars, entité unique. PAS de parenthèses (pas `"Wilkinson (drop)"`), PAS de multi-entités (`"Dupont et Ntamack"`) — choisis UNE entité ou reformule. Une marque seule (« 15 points », « 1987 ») est une entité valide.

12. **NE PAS générer de questions sur événements futurs incertains**. Pas de `correct_answer` « Aucun » / « À déterminer ».

13. **FRANÇAIS ACCENTUÉ (obligatoire)** : `text`, `correct_answer`, `wrong_answers[*].text` et `explanation` s'écrivent en français **correctement accentué** — é, è, ê, à, ç, ù, ô, î, û, et les majuscules accentuées (« À », « É »). Écris « La Nouvelle-Zélande », « L'Écosse », « une année », « il a soulevé », jamais « Nouvelle-Zelande » ni « annee ». Un énoncé sans accents est un défaut de qualité qui fait rejeter la question à la relecture.

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

**Le tag qui fait foi est le slug du thème imposé** (cf. directive « THÈME UNIQUE IMPOSÉ ») — utilise-le tel quel sur CHAQUE question. Slugs rugby seedés (migration 0275, gérés en base) :

- **Compétitions** : `coupe-du-monde-rugby`, `tournoi-six-nations`, `xv-de-france`, `top-14-rugby-francais`, `europe-des-clubs-rugby`.
- **Transverses** : `legendes-rugby`, `records-rugby`, `regles-rugby`.

Règles tagging :
- **1 batch = 1 thème principal.** Tague CHAQUE question avec le slug du thème imposé.
- 2ᵉ tag transverse **uniquement** s'il décrit réellement la question (un Grand Chelem français → `["tournoi-six-nations", "xv-de-france"]` ; le record de points de Wilkinson → `["coupe-du-monde-rugby", "records-rugby"]`). Pas de sur-tagging décoratif.
- ⚠️ **Pas de tag JO cross-sport** : `jeux-olympiques` appartient à l'athlétisme en base. Un fait olympique rugby (Sevens) vit sous `xv-de-france` ou `records-rugby`.

# Format de sortie strict

Retourne **uniquement** un JSON array d'objets (pas de markdown wrapper, pas de préface, pas de commentaire). Aucun champ supplémentaire.

```json
[
  {
    "sport_slug": "rugby",
    "difficulty": "argent",
    "text": "En 2011, quelle nation bat la France en finale de la Coupe du Monde ?",
    "correct_answer": "La Nouvelle-Zélande",
    "wrong_answers": [
      { "text": "L'Australie", "danger": 3 },
      { "text": "L'Afrique du Sud", "danger": 3 },
      { "text": "L'Angleterre", "danger": 2 },
      { "text": "Le pays de Galles", "danger": 2 },
      { "text": "L'Irlande", "danger": 2 },
      { "text": "L'Écosse", "danger": 2 },
      { "text": "L'Argentine", "danger": 1 },
      { "text": "L'Italie", "danger": 1 }
    ],
    "themes": ["coupe-du-monde-rugby", "xv-de-france"],
    "source_url": "https://www.world.rugby/tournaments/archive",
    "explanation": "8-7. Un point. Les Bleus ont fait trembler l'Eden Park pendant 80 minutes, et les All Blacks ont soulevé la coupe avec la tête de ceux qui reviennent de loin."
  }
]
```

> ℹ️ Exemple = argent (8 wrong : 2×d1 + 4×d2 + 2×d3). Pour bronze : 7 wrong (5×d1+2×d2). Pour or : 10 wrong (2×d1+4×d2+4×d3). **Suis la table de distribution.**

{{exclusion_block}}
# Exclusions strictes

- ❌ Aucune question sur paris sportifs, cotes, scandales liés aux paris.
- ❌ Aucune question sur dopage, commotions ou drama personnel non sportif (le fait sportif ancré est ok ; l'attaque gratuite non).
- ❌ Aucun champ JSON en dehors de la spec ci-dessus.
- ❌ Aucun `themes` vide : chaque question porte au moins le slug du thème imposé.
- ❌ Aucun usage de marque/logo comme label commercial (référence factuelle historique OK, cf. règle 2).
