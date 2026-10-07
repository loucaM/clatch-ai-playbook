# Prompt versionné `question_gen_v3.md`

> Source de vérité humaine pour la génération de questions via Claude — **mono-thème content-driving + anti-répétition** (Story 8.0k, 2026-05-22).
> Sync obligatoire en **3 endroits** : ce fichier ↔ const `QUESTION_GEN_V2_PROMPT` (`mobile/supabase/functions/_shared/prompt.ts`, fallback EF) ↔ row active DB `prompt_templates` (servie au runtime via `admin_publish_prompt`).
> Tout ajustement = bump version + changelog en bas.

## Variables interpolées au runtime

L'EF `generate_questions_batch` remplace ces placeholders avant l'appel Anthropic :

| Placeholder | Rempli par | Contenu |
|---|---|---|
| `{{sport_slug}}` | EF (body) | `foot` (V1) |
| `{{difficulty}}` | EF (body) | `bronze` \| `argent` \| `or` |
| `{{count}}` | EF (body) | 1-50 |
| `{{theme_directive}}` | EF (body `theme_slug`/`theme_label`/`theme_description`) | Bloc « THÈME UNIQUE IMPOSÉ » (label + description) — vide si pas de thème |
| `{{exclusion_block}}` | EF (service-role query) | Bloc « Questions DÉJÀ en base » (jusqu'à 120 énoncés theme+difficulty) — vide si N=0 |

> ⚠️ `{{theme_directive}}` et `{{exclusion_block}}` sont construits **côté EF** (service-role, **hors** du champ `extra_instruction` qui est capé à 500 chars). C'est ce qui permet l'anti-répétition (D2) et le mono-thème content-driving (D1) sans saturer le cap.

## Tâche

Génère **{{count}} questions** au format **JSON strict** spécifié plus bas, pour le sport `{{sport_slug}}` à la difficulté `{{difficulty}}`.

`{{theme_directive}}` ← le bloc mono-thème est injecté ici par l'EF. Exemple rendu :

```
# ⚠️ THÈME UNIQUE IMPOSÉ (mono-thème content-driving)

Les {{count}} questions portent **EXCLUSIVEMENT** sur : **CdM 1998** — La Coupe du Monde 1998 organisée en France, sacre des Bleus.
Chaque question traite le CONTENU de ce thème (pas seulement un tag décoratif).
Tag obligatoire sur CHAQUE question : `cdm-1998` (+ parent `mondiaux` si applicable).
```

## Règles éditoriales

1. **Ton chambreur dans `explanation`** : second degré, pas misogyne, pas méchant gratuit.
2. **Doctrine "référence nécessaire" IP** : athlètes informatif autorisé ; marques sportives nominatif uniquement ; anneaux olympiques + "Olympique/JO" protection ultra-stricte (loi 1992 art. L141-5) → fallback noms génériques.
3. **Politique éditoriale H/F** (foot V1) : ~80/20 H/F. Zéro question dégradante envers une joueuse ou compétition féminine. Pas de stéréotype.
4. **Difficulté calibrée** : bronze (faits ultra-connus, angles variés, fan occasionnel) · argent (fan averti, angles subtils) · or (érudit, stats précises, records méconnus, joueurs hors top stars).
5. **ANTI-RÉPÉTITION / VARIÉTÉ OBLIGATOIRE (v3.3)** : ne ressors PAS systématiquement les faits ultra-célèbres d'un thème (ex: pas 50× Zidane 98). Le bloc « Questions DÉJÀ en base » = ce qui existe → contenu **strictement nouveau** (ni ces faits, ni ces réponses ; reformulation d'un fait déjà présent = doublon). **Fais TOURNER les axes sur le batch** : compétitions (L1, C1/C3, Euro, CdM, CAN, Copa, clubs ET sélections) · postes (gardiens→attaquants→entraîneurs) · géographie (Europe + Amérique du Sud + Afrique + féminin) · époques (70-90 oubliées → actuel). Creuse les angles rares : hors top stars, dates/records/minutes méconnus, seconds rôles, éditions oubliées.
6. **Sources** — **Wikipédia ≤25%** des questions (le reste DOIT venir d'autres sources), diversifie activement (~50 sources : L'Équipe/So Foot/FIFA/UEFA/Transfermarkt/FBref/RSSSF⭐/worldfootball.net/11v11.com⭐/BBC/ESPN FC/Marca/Gazzetta/sources féminin dédiées/sites officiels clubs). Privilégie les sources spécialisées/primaires à Wikipédia.
7. **Pas de question piégeuse** sur "ça dépend" — bonne réponse incontestable.
8. **Énoncé concis** : 10-200 chars. Variété formulations : ne pas commencer >40% des questions par "Quel/Quelle" ; ~un quart d'angles narratifs contextualisés.
9. **`correct_answer` concis** : 1-80 chars, entité unique sans parenthèses ni multi-entités.
10. **NE PAS générer de questions sur événements futurs incertains**. Pas de `correct_answer` "Aucun" / "À déterminer".

## Format `wrong_answers` (pool 7-10 distractors danger 1-3)

| Difficulté | Total wrong | danger=1 | danger=2 | danger=3 |
|---|---|---|---|---|
| bronze | 7 | 5 | 2 | 0 |
| argent | 8 | 2 | 4 | 2 |
| or | 10 | 2 | 4 | 4 |

Anti-collision : aucun `wrong_answers[*].text` = `correct_answer` (case+accent-insensitive). Pas de doublons internes.

## Tags — slugs autoritaires (sinon rejet par EF import)

Universels : `mercato` · `tactiques` · `legendes` · `buts` · `ligue1` · `c1` · `euro` · `bleus-de-france`.
Compétitions clubs : `coupe-de-france` · `ligue-europa`. Féminin : `bleues` · `euro-feminin` · `cdm-feminine`.
Mondiaux : `mondiaux` · `cdm-1998` · `cdm-2006` · `cdm-2018` · `cdm-2022`. CAN : `can`.
Transverses : `gardiens` · `entraineurs` · `derbies` · `transferts-records` · `stades-mythiques`.

> ℹ️ Slugs événementiels (`cdm-2026`, `can-*`, championnats européens) et tout NOUVEAU thème créé via `/admin/themes` sont injectés via la `{{theme_directive}}` au runtime (ADR-011 + Story 8.0k). La liste autoritaire ci-dessus reste valide pour le tagging secondaire.

### Règles tagging (v3.0 mono-thème — D1)

- **1 batch = 1 thème principal.** Tague CHAQUE question avec le slug du thème imposé (cf. `{{theme_directive}}`). Sous-thème → ajoute le parent.
- 2ᵉ tag transverse autorisé **seulement** s'il décrit réellement la question (pas de sur-tagging décoratif).
- ❌ **Retrait v3.0** de la règle v2.x « une question peut avoir plusieurs thèmes — c'est encouragé » (causait la dérive multi-thème + dilution du sujet).

## Format de sortie strict

JSON array d'objets uniquement (pas de markdown wrapper, pas de préface). Champs : `sport_slug`, `difficulty`, `text`, `correct_answer`, `wrong_answers[]`, `themes[]`, `source_url`, `explanation`. Aucun champ supplémentaire.

`{{exclusion_block}}` ← le bloc anti-répétition est injecté ici par l'EF. Exemple rendu :

```
## Questions DÉJÀ en base sur ce thème (NE reproduis NI ces faits NI ces réponses, génère du NOUVEAU) :
- Quel joueur a inscrit un doublé en finale de la CdM 1998 ?
- Dans quel stade s'est jouée la finale de la CdM 1998 ?
- …
```

## Exclusions strictes

- ❌ Paris sportifs, cotes, scandales liés aux paris.
- ❌ Drama personnel non sportif.
- ❌ Champ JSON hors spec.
- ❌ Slug `themes` hors liste autoritaire / hors thème injecté.

---

## Changelog

- **v3.3** (2026-07-27) — Anti-boucle (même passe qu'athlé/tennis/vélo v1.1). (a) **Sources** (règle 6) : +worldfootball.net, +11v11.com⭐ (archives), +ESPN FC. (b) **Anti-répétition renforcée** (règle 5) : rotation OBLIGATOIRE des axes sur le batch (compétitions clubs ET sélections, postes, géo Europe/Amérique du Sud/Afrique/féminin, époques) + reformulation d'un fait déjà en base = doublon. Sync les 3 endroits : ce fichier ✅ ↔ const `QUESTION_GEN_V2_PROMPT` (`_shared/prompt.ts`) ✅ ↔ **row DB à republier** (DEV puis PROD sur GO). Se combine avec Opus 4.7→4.8 + `EXCLUSION_LIMIT` 120→250. ⚠️ Fallback code changé → **redéployer l'EF** `generate_questions_batch`.
- **v3.1** (2026-05-22) — Story 8.0k itération (retours Louca). **(1) Thèmes DYNAMIQUES** : `_shared/schema.ts` ne fige plus les slugs en enum (`picklist`/`enum` → string kebab) → un thème créé via `/admin/themes` est désormais générable ET importable (la table `themes` = autorité, validée à l'import via `lookupThemeIds`). Section Tags reformulée : le slug du thème imposé fait foi, la liste devient des exemples non exhaustifs. **(2) Wikipédia 40% → 25%.** **(3) EF `suggest_themes`** : modèle Opus 4.7 → **Sonnet 4.6** (timeout synchrone 30s dépassé par Opus) + schéma Structured Outputs racine OBJET `{themes:[]}` (l'array racine n'est pas supporté par la grammaire). Redéploy generate + import + suggest DEV+PROD via CLI, republish v3.1 DB.
- **v3.0** (2026-05-22) — Story 8.0k. **(1) Mono-thème content-driving** : nouveau placeholder `{{theme_directive}}` rempli par l'EF avec label + description du thème → les questions portent EXCLUSIVEMENT sur le CONTENU du thème, pas seulement le tag. **Retrait** de la règle « multi-thème encouragé » (cause de la dilution du sujet). **(2) Anti-répétition** : nouveau placeholder `{{exclusion_block}}` rempli par l'EF (service-role) avec jusqu'à 120 énoncés déjà en base (theme+difficulty) → consigne « ne reproduis NI ces faits NI ces réponses » ; + règle éditoriale 5 renforcée « évite les faits les plus évidents, creuse des angles rares ». Nouveaux params EF : `theme_slug`/`theme_label`/`theme_description`. Sync const `QUESTION_GEN_V2_PROMPT` + publish DB DEV+PROD.
- **v2.9** (2026-05-15) — Story 8.0e. ADR-011 évergreen mono-thème (tag-level). Retrait section « Coverage cible » + slugs événementiels injectés via `extra_instruction`. (Historique complet : voir `question_gen_v2.md`.)
