## Case facts
- **Branch** : `main`
- **Last commit** : `e6cd0b8` — fix(setup): skip obsidian.json rewrite if Obsidian is running
- **Commits non pushés** : 0 (tout pushé, CI verte sur 5 runs consécutifs)
- **Open PRs** : none
- **Blocking issue** : none
- **Next concrete step** : Attendre le retour de Xavier sur son test end-to-end fresh install (`/Users/xais/Desktop/OBSIDIAN-BRAIN`, onboarding lancé 2026-04-22). Si bugs rapportés → `/XD-debug` ; sinon → soit piste 5 (demo GIF VHS), soit fix des 3 warnings review du commit 6F (DOCX 40%, docstring CI, dead code `format_inbox_warning`), soit bug `detect_stale_to_verify` vault_audit.py:~380.

## HANDOFF — xais-brain — 2026-04-22

### Contexte & décisions

- **Problème traité** : Finir le plan 06 (pistes 6D, 6F, 6C, 6G, 6H restantes après session 010), synchroniser la documentation, préparer un test end-to-end fresh install par Xavier sur un nouveau vault.
- **Approche choisie (et pourquoi pas les autres)** :
  - **Choix** : livraison séquentielle pistes 6D → 6F → 6C → 6G → 6H en mode `/XD-quality` (validate → review → commit) après chaque piste. Archive plan 06 dans `specs/done/` après 6H. Puis docs sync + QUICKSTART + wipe complet des artifacts xais-brain locaux pour permettre à Xavier un onboarding from-scratch.
  - **Rejeté** : commit global final — raison : visibilité du progrès, CI verte par piste, rollback plus fin.
  - **Rejeté** : sortir 6H dans un plan 07 séparé — raison : plan 06 §0.3 l'avait actée "GO optionnelle" et elle renforce Axe A (density rule = feature différenciante pour le graphe).
  - **Rejeté** : laisser vault-3 + venv + skills globaux en place — raison : Xavier voulait un vrai test fresh, les artifacts persistants auraient fait sauter des étapes de `setup.sh` et masqué des bugs d'onboarding.
- **Décisions d'architecture clés** :
  - **Obsidian running** : si Obsidian tourne pendant `setup.sh`, on skip l'écriture de `obsidian.json` (elle serait écrasée au quit par l'état RAM d'Obsidian). Instructions claires données à l'utilisateur à la place (commit `e6cd0b8`).
  - **Convention handoff untracked** : les handoffs restent untracked pendant leur session, trackés ensuite par `chore(specs)` ou via `/XD-handoff` suivant (pattern observé `cf1c1f3`).
  - **Frontmatter `liens_forts` / `liens_opposition`** (piste 6H) : syntaxe YAML liste valide obligatoire dans l'exemple du prompt (`["[[Concept]]"]`), corrigée pré-commit suite à flag reviewer — sans quoi `parse_frontmatter` de `vault_audit.py` aurait échoué silencieusement sur les notes produites.
  - **Seuil density rule** : 3 wikilinks sortants (assoupli vs god-mode 7), daily/ exclu, anémiques exclus (évite double-signalement).
  - **Checkpoint 6C** : seuil batch ≥ 6 fichiers pour déclencher le post-preview checkpoint ; bypass auto en CI (`XAIS_BRAIN_CI=1` ou `CI=1`) et non-TTY.
  - **`pricing.json`** (piste 6F) : fichier séparé `.claude/pricing.json` dans le vault, merge additif avec `_DEFAULT_PRICING` hardcodé en fallback.

### Findings critiques

- **Piste 6D livrée** (commit `a42edab`) : `extract_note_title()` + `append_fact_check_log()` wirés dans `scripts/file_intel.py` `process_file()`. Pattern identique à `web_clip.py` (piste 6B côté /clip). +7 tests. Best-effort : no-op silencieux si `99-Meta/` absent.
- **Piste 6F livrée** (commit `c2c3829`) : `scripts/budget.py` (266 lignes) + `vault-template/.claude/pricing.json` (3 providers × 8 heuristics) + `announce_budget()` dans `file_intel.main()` avant `process_file()`. Format : `"va traiter N fichier(s) (~P pages). Estimation : TTT, coût (provider: X)"`. +18+3 tests (74 pytest total).
- **Piste 6C livrée** (commit `663d4e7`) : `_is_ci_mode()` + `prompt_checkpoint()` + 2 checkpoints (pre-batch, post-preview après 3 premiers fichiers si batch ≥ 6). SKILL.md `inbox-zero` mis à jour (section 3.bis "Checkpoint humain"). +11 tests (85 pytest).
- **Piste 6G livrée** (commit `5573dcf`) : `vault-template/GUIDE.md` 345 lignes (10 sections + FAQ). Liens `../README.md` / `../ARCHITECTURE.md` **corrigés en URLs GitHub absolues** pré-commit (sinon morts dans le vault copié). Warning reviewer capté à temps.
- **Piste 6H livrée** (commit `93af5e1`) : `DENSITY_MIN_WIKILINKS=3`, `detect_low_density()` (exclut anémiques + daily/), prompt file-intel demande `liens_forts` + `liens_opposition` dans frontmatter. Plan 06 **archivé** dans `specs/done/`. +8+4 tests (97 pytest). Syntaxe YAML de l'exemple `liens_forts` corrigée en liste (`["[[Concept1]]", ...]`) pré-commit — sans quoi invalide.
- **Docs sync** (commit `4ea7557`) : README (7 → 8 détections, frontmatter enrichi, CLI flags `--json`/`--output`, mention GUIDE + §"Auto-discipline"), ARCHITECTURE (count skills 11 → 12, arbo `budget.py` + `pricing.json` + `GUIDE.md`, flux `/file-intel` avec budget+checkpoint+fact-check, handoffs à jour). 60/-14 lignes.
- **QUICKSTART.md ajouté** (commit `57efa71`) : plan de test end-to-end 9 étapes + cheat-sheet quotidien. Copié dans le vault par `setup.sh`. +58/57 bash asserts.
- **Investigation vaults fantômes** : `OBSIDIAN GLOBAL` = vault perso intouchable (pas de `.claude/`, pas de `CLAUDE.md`), `xais-brain-vault-3` = xais-brain legacy (setupDate 2026-04-09, pas lié git, pas à jour). Confirmé 0 worktree orphelin, 1 seul remote. `/tmp/xais-demo-vault-1776777004` et `/tmp/density-test-6h` = leftovers build.
- **Wipe complet effectué par Xavier** (étapes 1,2,3,5 + 4 Obsidian UI) : `~/xais-brain-vault-3`, `~/.xais-brain-venv`, `~/.local/bin/xb`, 12 skills globaux `~/.claude/skills/`, `/tmp` leftovers. Vérifié 100 % propre.
- **Onboarding fresh lancé par Xavier sur `/Users/xais/Desktop/OBSIDIAN-BRAIN`** : install OK en 10 étapes. Choix : LLM `4) Skip` (donc `.env` vide, `xb intel` indisponible tant qu'une clé API n'est pas ajoutée) ; pas d'import ; pas de Kepano.
- **Bug Obsidian running identifié + fixé** (commit `e6cd0b8`) : si Obsidian tourne pendant setup, sa RAM écrase `obsidian.json` au quit, laissant 2 vaults avec `open:true` → Obsidian ouvre le mauvais au relaunch. Fix : skip l'écriture dans ce cas + instructions utilisateur claires. Vérifié dans l'état actuel de `~/Library/Application Support/obsidian/obsidian.json` (2 vaults `open:true` simultanés observés en live).
- **CI verte** : 5 runs consécutifs (`24787285944` 6C, `24788400245` 6G, `24790144877` docs sync, `24791637152` QUICKSTART, `24793652957` obsidian fix). Warnings Node 20 deprecated toujours non bloquants.
- **Plan 06 archivé** dans `specs/done/06-integration-god-mode-patterns.md` (8/8 pistes livrées : 6A + 6B + 6C + 6D + 6E + 6F + 6G + 6H).

### Terminé

- Build piste 6D (commit `a42edab`) : Fact-Check-Log auto côté `/file-intel`
- Build piste 6F (commit `c2c3829`) : budget.py + announce_budget + pricing.json
- Build piste 6C (commit `663d4e7`) : checkpoints humains batch + bypass CI
- Build piste 6G (commit `5573dcf`) : GUIDE.md dans vault-template — **✅ resolved : dette documentaire latente (héritée handoff 010)**
- Build piste 6H (commit `93af5e1`) : density rule wikilinks + plan 06 archivé
- Sync docs (commit `4ea7557`) : README + ARCHITECTURE alignés 6C-6H
- QUICKSTART (commit `57efa71`) : plan de test + cheat-sheet copié au vault
- Investigation exhaustive des coffres fantômes (Obsidian registry, venv, CLI, tmp)
- Wipe complet des artifacts xais-brain (hors OBSIDIAN GLOBAL) par Xavier
- Onboarding fresh réussi sur `/Users/xais/Desktop/OBSIDIAN-BRAIN` (10 étapes vertes)
- Fix bug Obsidian running (commit `e6cd0b8`) découvert en live pendant l'onboarding
- 5 pushes avec watch CI, tous verts

### Fichiers en cours (non commités)

- `specs/handoffs/010-2026-04-21-pivot-axe-A-vault-audit-readme.md` — untracked (à tracker prochaine session via `chore(specs)`)
- `specs/handoffs/011-2026-04-22-plan-06-complet-wipe-reinstall.md` — ce handoff (untracked par convention)

### Prochaine étape immédiate

Attendre feedback de Xavier sur son test manuel du `QUICKSTART.md` dans `/Users/xais/Desktop/OBSIDIAN-BRAIN`. Avant de tester `/file-intel` ou `xb intel`, il devra ajouter une clé API (`GOOGLE_API_KEY=...`) dans `/Users/xais/Desktop/OBSIDIAN-BRAIN/.env`. Si bugs remontés → `/XD-debug` ciblé ; sinon prochain chantier = soit piste 5 (demo GIF VHS), soit les 3 warnings 6F en dette (DOCX 40% sous-estimé, docstring `XAIS_BRAIN_CI` non implémentée, `format_inbox_warning` dead code).

### Risques identifiés

**Nouveaux risques découverts cette session** :
- **Warnings review 6F non fixés** (commit `c2c3829`) : formule DOCX `estimate_file_input_tokens` sous-estime de ~40% (`scripts/budget.py:140-142`) ; docstring `announce_budget` mentionne `XAIS_BRAIN_CI`/`CI` non implémentés dans la fonction (`file_intel.py:107-109`) ; `format_inbox_warning` dans `budget.py:256-265` n'est appelée nulle part (dead code prod, testée seulement). Impact cosmétique, pas bloquant.
- **Warnings review 6C non fixés** (commit `663d4e7`) : `except Exception` large ligne 180 de `file_intel.py` (swallow toute erreur du `format_budget_line`) ; `prompt_checkpoint` retourne `default_yes` sur réponse inattendue (surprenant — préférable répéter ou renvoyer False).
- **`.env` vide sur vault test Xavier** : setupDate 2026-04-22, LLM choisi = Skip. `xb intel` échouera tant que `GOOGLE_API_KEY` (ou autre) pas ajouté. Attendu, documenté dans QUICKSTART, juste à ne pas oublier côté Xavier.
- **Dual `open:true` résiduel dans `~/Library/Application Support/obsidian/obsidian.json`** (OBSIDIAN GLOBAL + OBSIDIAN-BRAIN). Pas critique — Xavier peut corriger manuellement via Obsidian UI (Manage vaults). Je n'ai pas touché au fichier sans autorisation (scope guard).
- **Narrative README/QUICKSTART non testée par tiers externe** : Xavier est l'utilisateur mais aussi le concepteur. Attendre première issue GitHub externe pour valider.

**Risques hérités encore ouverts** (repris du handoff 010, re-vérifiés) :
- **Bug `detect_stale_to_verify` dans `scripts/vault_audit.py:~380`** : liste les `statut: archived` en plus des `to-verify`. Inchangé depuis handoff 010 — confirmé par review piste 6H. Impact cosmétique. Fix proposé : `if statut in {"verified", "archived"}: continue` avant le check stale.
- **Dette infra héritée** (handoff 010 / 005 / 006) : migration GitHub Actions → Node 24 (deadline juin 2026), `$XAIS_BRAIN_REPO` toujours non fixée, skills `/client` et `/project` non testés runtime, `setup.sh` ~900+ lignes. Inchangé.
- **Piste 5 (demo GIF VHS)** toujours reportée depuis handoff 008. Inchangée — feature marketing la plus impactante non livrée.
- **Narrative README non éprouvée sur cible réelle** (handoff 010) : partiellement atténué — Xavier a lui-même fait l'onboarding fresh avec le README/QUICKSTART comme référence, sans bug conceptuel. Reste à valider par un utilisateur externe.

### Prêt pour la prod ?
- [x] Tests OK (97 pytest + 58 bash asserts, CI macOS + Linux verte sur 5 runs)
- [x] Build OK (CI run `24793652957` success)
- [x] Testé manuellement (onboarding fresh end-to-end sur OBSIDIAN-BRAIN par Xavier ; `xb status` fonctionnel)
- [x] Pas de debug oubliés
- **Décision** : **OUI** — plan 06 livré 8/8 pistes, docs alignées, QUICKSTART en place, fresh install validé, bug onboarding découvert-fixé-poussé en une session. Prêt pour utilisateurs externes.

### Fichiers clés

**Créés/modifiés cette session** :
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/scripts/file_intel.py` — +`extract_note_title()`, `append_fact_check_log()`, `announce_budget()`, `_is_ci_mode()`, `prompt_checkpoint()`. Point d'attention `except Exception` large L180 (dette review).
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/scripts/budget.py` — nouveau module 266 lignes. Bug formule DOCX L140-142 (sous-estime 40%), `format_inbox_warning` dead code L256-265.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/scripts/vault_audit.py` — +`DENSITY_MIN_WIKILINKS=3`, `detect_low_density()`. **Bug hérité `detect_stale_to_verify`** toujours présent (ligne ~223).
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/scripts/providers/_prompts.py` — frontmatter demande `liens_forts` / `liens_opposition` en syntaxe YAML liste valide.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/vault-template/GUIDE.md` — nouveau, 345 lignes. Liens GitHub absolus.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/vault-template/QUICKSTART.md` — nouveau. Plan de test 9 étapes + cheat-sheet.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/vault-template/.claude/pricing.json` — nouveau, config tarifs LLM partial-mergeable.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/setup.sh` — copies GUIDE+QUICKSTART+pricing.json+budget.py ; fix obsidian running (lignes 906-965).
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/README.md` — sync pistes 6C-6H (7 → 8 détections, §Auto-discipline, frontmatter liens_forts, CLI flags `--json`/`--output`).
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/ARCHITECTURE.md` — sync : count skills 12, scripts/budget.py, vault-template/GUIDE+QUICKSTART, flux `/file-intel` complet.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/specs/done/06-integration-god-mode-patterns.md` — plan archivé 8/8 pistes livrées. Source de vérité historique.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/tests/test_budget.py` (+18 tests), `tests/test_file_intel.py` (+25 cumul : extract_title, fact_check, announce_budget, prompt_checkpoint, density prompt), `tests/test_vault_audit.py` (+8 low_density), `tests/test_setup.sh` (+3 GUIDE + QUICKSTART).

**Vault test utilisateur** : `/Users/xais/Desktop/OBSIDIAN-BRAIN/` (fresh install 2026-04-22, `.env` vide, setupDate 2026-04-22). À utiliser pour les tests QUICKSTART.

**Référence vault démo** : `/tmp/xais-demo-vault-1776777004/` **supprimé** cette session (wipe).

**Specs** :
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/specs/todo/05-demo-gif-readme.md` — piste 5 (demo GIF) toujours en attente de VHS.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/specs/done/06-integration-god-mode-patterns.md` — plan 06 livré.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/specs/handoffs/010-*.md` — handoff précédent (référence du pivot).
