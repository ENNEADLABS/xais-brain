## Case facts
- **Branch** : `main`
- **Last commit** : `1c4eb8e` — docs(readme): rebrand around vault audit score (Axe A narrative)
- **Commits non pushés** : 0 (tout pushé, CI verte)
- **Open PRs** : none
- **Blocking issue** : none
- **Next concrete step** : `/XD-build specs/todo/06-integration-god-mode-patterns.md` piste 6D (Fact-Check-Log auto-alimenté côté `/file-intel`) — OU piste 5 (demo GIF VHS) si priorité marketing.

## HANDOFF — xais-brain — 2026-04-21

### Contexte & décisions

- **Problème traité** : Xavier a demandé prise de recul sur la différenciation de xais-brain (saturation marché PKM+AI : Obsidian+Copilot, Reor, Khoj, templates "Claude+Obsidian" pléthoriques). Identifier un positionnement défendable et l'exécuter.
- **Approche choisie (et pourquoi pas les autres)** :
  - **Choix : A-façade + C-moteur**. Axe A = "self-auditing second brain" (narrative, README, marketing). Axe C = "AI-native by design" (frontmatter enrichi, skills, CLAUDE.md vault optimisé LLM). Les deux se renforcent : sans C, pas d'audit mesurable ; sans A, positionnement banal.
  - **Rejeté : Axe C pur** — raison : saturé (Notion AI, Mem.ai, Reflect, Tana disent tous "AI-native"). Difficile de prouver qu'on est "plus AI-native" que les autres, pas viral, gagne sur qualité d'exécution pas positionnement.
  - **Rejeté : Axe A pur** — raison : sans moteur technique (frontmatter enrichi), l'audit ne serait pas mesurable. Pas de feature défendable derrière la narrative.
  - **Rejeté : intégration du repo god-mode tel quel** (décidé session 009) — architecture skills xais-brain reste souveraine, cherry-picking des patterns uniquement.
- **Décisions d'architecture clés** (actées §0.3 du plan 06) :
  - Ordre pistes révisé : **6B → 6E → 6D → 6F → 6C → 6G → (6H)** — `/vault-audit` passe de 4e à 2e position car feature différenciante
  - Rétrocompat frontmatter : **additif uniquement** + migration opt-in `xb audit --migrate`
  - `/vault-audit` = skill Claude **ET** CLI `xb audit` (les deux)
  - Frontmatter incomplet dans `xb audit` = **warning non bloquant** (exit 0) pour que les vaults existants gardent un score
  - Density rule (6H) = **GO** (renforce Axe A)
  - Pricing tokens (6F) = **fichier config séparé** `.claude/pricing.json`
  - Exclusion `99-Meta/` graphe Obsidian = **doc manuelle d'abord** (v2 : `.obsidian/app.json` auto)

### Findings critiques

- **Piste 6B livrée** (commit `f93a55a`) : frontmatter enrichi sur `/clip`, `/file-intel`, `/inbox-zero`. Champs : `source` (enum web/pdf/docx/...), `source_url`, `source_file`, `source_knowledge` (internal/web-checked), `verification_date`, `statut` (draft/verified/to-verify/archived), `importance` (low/medium/high/core). Breaking sémantique sur `source` (URL → enum) — mitigé par rétrocompat totale (notes existantes intactes). +12 tests (24/24 pytest verts).
- **Piste 6E livrée** (commit `c35084f`) : `scripts/vault_audit.py` (~597 lignes) + skill `/vault-audit` + CLI `xb audit`. 7 détections MVP : orphelines, anémiques (< 100 mots), doublons (titre exact), frontmatter incomplet, stale `to-verify` > 30j, tags incohérents, wikilinks cassés. Flags : `--migrate`, `--json`, `--output`. +22 tests (46/46 pytest verts au total).
- **README refait** (commit `1c4eb8e`) autour de la narrative Axe A : punch line "Le second brain qui s'auto-audite", hero section avec sample réel du rapport, 3 piliers (self-auditing, AI-native, local-first), `xb audit` en tête de la CLI. 288 → 351 lignes.
- **Sample rapport généré** à partir d'un vault démo réel dans `/tmp/xais-demo-vault-1776777004` (7 notes fictives couvrant chaque détection) — output authentique, pas fabriqué.
- **Bug mineur découvert mais NON corrigé** : le check "stale `to-verify`" liste aussi les notes avec `statut: archived` si leur `verification_date` est vieille. Devrait filtrer par statut. À fixer dans une itération future de `scripts/vault_audit.py` — ligne ~380 (fonction `detect_stale_to_verify`).
- **CI verte** sur tous les commits de la session : runs `24720258340` (6B), `24723151873` (6E), `24724342562` (README rebrand). Warnings Node 20 deprecated non bloquants (dette connue, deadline juin 2026).
- **Plan 00-plan-5-gaps-critiques.md archivé** dans `specs/done/` (4/5 pistes livrées, seule piste 5 demo GIF reste dans `specs/todo/`).

### Terminé

- Prise de recul + analyse concurrentielle → pivot stratégique **Axe A + C**
- Housekeeping specs (commit `cf1c1f3`) : 3 handoffs untracked trackés, plan 00 archivé
- Pivot plan 06 (commit `e7f1e67`) : §0 décisions, ordre révisé, Q1-Q8 tranchées
- Build piste 6B (commit `f93a55a`) : frontmatter enrichi 3 skills
- Sync README + ARCHITECTURE (commit `28d5a41`) : `99-Meta/`, 11 skills, `web_clip.py` ajouté au install manuel
- Build piste 6E (commit `c35084f`) : `/vault-audit` skill + `xb audit` CLI + 22 tests
- Rebrand README (commit `1c4eb8e`) : narrative Axe A en hero
- Génération d'un rapport d'audit réel sur vault démo pour le README

### Fichiers en cours (non commités)

- none (tout committed + pushed)

### Prochaine étape immédiate

Lancer `/XD-build specs/todo/06-integration-god-mode-patterns.md` piste 6D (Fact-Check-Log auto côté `/file-intel` — infrastructure côté `/clip` déjà en place via `append_fact_check_log()` en piste 6B). Alternative marketing : `/XD-build` piste 5 (demo GIF VHS, nécessite `brew install vhs`) pour capitaliser sur le README fraîchement refait.

### Risques identifiés

- **Bug `detect_stale_to_verify`** : liste les `archived` en plus des `to-verify`. Impact faible (cosmétique dans le rapport), mitigation = filtrer par statut à la prochaine itération. Ligne ~380 de `scripts/vault_audit.py`.
- **Narrative README non éprouvée** sur cible réelle : le pivot Axe A n'a pas encore été testé face à un utilisateur tiers. Risque de sur-promesse — mitigation : surveiller les premières reactions GitHub / issues.
- **Dette documentaire latente** : `GUIDE.md` utilisateur final (piste 6G) pas encore écrit. Le README est dense, peut décourager un novice. Mitigation = piste 6G planifiée.
- **Dette infra héritée** : migration GitHub Actions → Node 24 (deadline juin 2026, bloquant à terme), `$XAIS_BRAIN_REPO` toujours non fixée (hérité handoff 005), skills `/client` et `/project` non testés runtime (hérité handoff 006), `setup.sh` ~900+ lignes (refacto conseillée après toutes les pistes).
- **Piste 5 (demo GIF)** toujours reportée — reste la feature marketing la plus impactante non livrée.

### Prêt pour la prod ?

- [x] Tests OK (46 pytest + 51 bash, CI macOS + Linux verte sur 3 commits)
- [x] Build OK (CI run `24724342562` success)
- [x] Testé manuellement (`xb audit` exécuté en live sur vault démo, rapport généré)
- [x] Pas de debug oubliés (aucun `print` parasite, aucun code commenté)
- **Décision** : **OUI** — pivot stratégique acté, 2 pistes livrées (6B + 6E), narrative README alignée, tout pushé, CI verte. Prêt pour démonstration publique.

### Fichiers clés

**Créés/modifiés cette session** :
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/specs/todo/06-integration-god-mode-patterns.md` — plan maître 8 pistes. **§0 Pivot stratégique** acte positionnement A-façade + C-moteur + décisions Q1-Q8. Source de vérité pour les pistes restantes.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/scripts/vault_audit.py` — moteur CLI Python 597 lignes. **Bug `detect_stale_to_verify` ligne ~380** à fixer dans une prochaine itération.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/vault-template/.claude/skills/vault-audit/SKILL.md` — wrapper skill Claude FR, model haiku, user-invocable.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/scripts/web_clip.py` — `build_frontmatter()` + `append_fact_check_log()` best-effort (piste 6D côté `/clip` déjà posée).
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/scripts/providers/_prompts.py` — prompt LLM avec frontmatter enrichi + helper `source_type_from_filename()`. La piste 6D côté `/file-intel` devra ajouter l'append Fact-Check-Log ici ou dans `file_intel.py`.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/README.md` — refonte complète Axe A (351 lignes). Hero = sample du rapport audit.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/ARCHITECTURE.md` — section "Frontmatter standard (piste 6B)" + arbo `99-Meta/` + skill count 11 → 12.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/tests/test_vault_audit.py` — 22 tests (parsing, 7 détections, `--migrate`, CLI e2e). Référence pour ajouter de nouveaux tests.

**Référence vault démo** : `/tmp/xais-demo-vault-1776777004/` (7 notes fictives couvrant chaque détection). Peut servir pour re-générer des screenshots.

**Specs** :
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/specs/todo/05-demo-gif-readme.md` — piste 5 (demo GIF) toujours en attente de VHS
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/specs/todo/06-integration-god-mode-patterns.md` — 5 pistes restantes après 6E : 6D, 6F, 6C, 6G, 6H
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/specs/done/00-plan-5-gaps-critiques.md` — plan maître archivé (4/5 fait)
