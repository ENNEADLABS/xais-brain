## Case facts
- **Branch** : `main`
- **Last commit** : `55d32e6` — fix(health): batches 1-5 du plan 07 — P0 bloquants + sécu + docs
- **Commits non pushés** : 0 (tout pushé, CI verte run `24797537998`)
- **Open PRs** : none
- **Blocking issue** : none
- **Next concrete step** : Créer `specs/todo/07.1-hardening-ssrf-env-packaging.md` pour traiter les 4 findings critical/warning restants de la review plan 07 (SSRF redirect bypass + DNS rebinding TOCTOU dans `web_clip.py`, TOCTOU `.env` dans `setup.sh`, `pytest` dans requirements prod). Sinon : feedback user sur le vault test `/Users/xais/Desktop/OBSIDIAN-BRAIN` (QUICKSTART), ou piste 5 (demo GIF VHS).

## HANTOFF — xais-brain — 2026-04-22

### Contexte & décisions

- **Problème traité** : suite à la session 011 qui a livré le plan 06 (8/8 pistes) + premier onboarding fresh par Xavier, un `/XD-health` a remonté 30 findings (1 HAUTE OWASP, 3 MOYENNE, 2 BASSE + 24 structurels). Session 012 = planifier + livrer les batches 1-5 du plan 07 qui en découle.
- **Approche choisie (et pourquoi pas les autres)** :
  - **Choix** : plan 07 structuré en 5 batches thématiques (P0 bloquants → docs désync → dette review → sécu P1 → fondations dev) + Batch 6 refacto repoussé en spec 08 séparée. Livraison en 1 `/XD-build` global (pas par batch), 1 commit global avec scope `fix(health)` listant les 5 batches dans le body.
  - **Rejeté : livraison batch par batch avec 5 commits atomiques** — raison : cohérence du plan 07 comme unité, Xavier n'a pas demandé split, et les batches sont interdépendants (Batch 1 révèle tests cachés, Batch 5 factorise la plomberie utilisée par les autres). Un seul commit est plus lisible qu'un chain de 5.
  - **Rejeté : intégrer Batch 6 (refacto setup.sh + vault_audit.py)** — raison : risque régression trop élevé pour mélanger avec les fix de bugs ; spec 08 séparée planifiée.
  - **Rejeté : fixer les 2 critical SSRF dans ce commit** — raison : découverts uniquement par la review, pas dans le plan 07 initial. Seraient du scope creep ; ils vont dans un plan 07.1 de durcissement.
- **Décisions d'architecture clés** :
  - **CI `pytest tests/ -v`** : remplace `pytest tests/test_file_intel.py -v` qui ne tournait qu'un test file sur 4. Impact immédiat : 103 tests au lieu de 18. Risque découvert rétrospectivement : aucun test n'était rouge → mais la CI mentait depuis ~2 semaines (tout commit pouvait passer sans que 65/83 tests ne tournent).
  - **`scripts/web_clip.py` `_is_safe_url()`** : blocklist (rejette privées/loopback/link_local/reserved/multicast/unspecified + schemes ≠ http/https). Pas d'allowlist CIDR explicite (MVP). Review signale que `follow_redirects=True` contourne la protection au premier 302 → dette à reprendre en 07.1.
  - **`setup.sh` .env** : `umask 077` + write + `chmod 600`. Review signale TOCTOU entre write et chmod (fenêtre ~ms sur FS multi-user). Pattern canonique `mktemp + mv` à faire en 07.1.
  - **`requirements.txt` pytest** : choix simple "tout dans requirements.txt" (spec §5.2 l'a acté). Review remonte que `pytest` en prod pollue les installs utilisateur — à extraire vers `requirements-dev.txt` en 07.1.
  - **Kepano pin SHA `fa1e131a...`** : checkout après clone plein (perdu `--depth=1`). Review remonte le coût perf de clone sans shallow — alternative `git init + fetch --depth=1 origin <SHA>` à envisager.
  - **Plan 07 archivé** dans `specs/done/` immédiatement après `/XD-build` (pattern identique à 06 en session 011).

### Findings critiques

- **Plan 07 livré 5/5 batches** (commit `55d32e6`, 23 fichiers modifiés, +584/-87 lignes) : Batch 1 (CI + .env + detect_stale_to_verify + README + ARCHITECTURE), Batch 3 (docs GUIDE/99-Meta/Audit.md/SKILL.md à 8 détections), Batch 2 (DOCX formula + format_inbox_warning supprimé + seconds_per_file_avg retiré + docstring + except + prompt_checkpoint), Batch 4 (SSRF basic + injection vault-cli + exception leak + unset API_KEY + Kepano pin), Batch 5 (conftest.py + pytest req + settings.json allow mkdir/mv/chmod/cp + deny rm).
- **CI passe de 18 → 103 tests en CI** (`.github/workflows/ci.yml:46,87`). Aucun test caché n'était rouge : la suite est stable malgré le mensonge structurel des semaines précédentes. Premier run complet : `24797537998` PASS.
- **Bug hérité `detect_stale_to_verify` corrigé** (`scripts/vault_audit.py`) : ajout `archived` dans le set des statuts skippés + test `test_stale_ignores_archived`. Dette traînait depuis handoff 010.
- **Nouveau module SSRF** `scripts/web_clip.py:_is_safe_url()` + 5 tests. Protège contre `file://`, `gopher://`, IPs privées/loopback à la première requête. **NE PROTÈGE PAS** contre les redirects `302 → IP interne` (`follow_redirects=True` non intercepté) — critical flagged par review.
- **Review APPROVED_WITH_NOTES** avec 2 critical sécu résiduels (SSRF redirect bypass + DNS rebinding TOCTOU dans `web_clip.py`) + 12 warnings/info. Net gain sécu du commit reste positif : de "0 SSRF protection" à "protection basique + edge cases à durcir".
- **CI verte** sur le commit (`24797537998`) — run le plus long et le plus révélateur de la série (premier avec les 103 tests).
- **Handoffs 010 + 011 toujours untracked** (convention `chore(specs)` séparé) — à tracker à la prochaine session ou dans un batch housekeeping. Ce handoff 012 aussi, à tracker ensuite.

### Terminé

- Audit `/XD-health` complet (focus `all`) : 6 OWASP + 24 structurels, verdict NEEDS ATTENTION → plan 07 créé
- Plan 07 rédigé dans `specs/todo/07-health-audit-fixes.md` (5 batches ordonnés + critères succès + métriques)
- `/XD-build` du plan 07 : 21 fichiers modifiés + `tests/conftest.py` créé, archivage du plan en `specs/done/`
- `/XD-quality` chain complète : validate (103/103 pytest, 58/58 bash, JSON OK) → review (APPROVED_WITH_NOTES, 2 critical/4 warning/8 info) → commit (`55d32e6`) → push → CI verte
- **✅ resolved (session 012)** : Bug hérité `detect_stale_to_verify` (handoff 010 puis 011) — fixé via `scripts/vault_audit.py` + test dédié
- **✅ resolved (session 012)** : Warnings review 6F hérités handoff 011 — DOCX 40%, docstring `announce_budget`, `format_inbox_warning` dead code, `seconds_per_file_avg` dead config — tous traités Batch 2 du plan 07
- **✅ resolved (session 012)** : Warnings review 6C hérités handoff 011 — `except Exception` large L180, `prompt_checkpoint` default_yes sur réponse inattendue — tous traités Batch 2 du plan 07
- **✅ resolved (session 012)** : Sécu HAUTE OWASP A05 (`.env` chmod 600) + MOYENNE (injection vault-cli, SSRF basic, supply chain Kepano) + BASSE (exception leak, API_KEY mémoire shell) — tous traités Batch 1 + 4
- **✅ resolved (session 012)** : CI menteuse — run 83 tests au lieu de 18 désormais

### Fichiers en cours (non commités)

- `specs/handoffs/010-2026-04-21-pivot-axe-A-vault-audit-readme.md` — untracked depuis session 011 (à tracker via `chore(specs)`)
- `specs/handoffs/011-2026-04-22-plan-06-complet-wipe-reinstall.md` — untracked depuis session 011
- `specs/handoffs/012-2026-04-22-plan-07-health-fixes-batches-1-5.md` — ce handoff, untracked par convention

### Prochaine étape immédiate

Créer `specs/todo/07.1-hardening-ssrf-env-packaging.md` regroupant les 4 findings critical/warning résiduels de la review plan 07 (SSRF redirect bypass httpx + DNS rebinding TOCTOU dans `web_clip.py:_is_safe_url` + `fetch_and_extract`, TOCTOU `.env` mktemp+mv dans `setup.sh:step7_llm_config`, extraction `pytest` vers `requirements-dev.txt`). Durée estimée : 1h. Alternative si priorité marketing : lancer piste 5 (demo GIF VHS, nécessite `brew install vhs`) pour capitaliser sur le README + QUICKSTART stabilisés.

### Risques identifiés

**Nouveaux risques découverts cette session** :
- **SSRF redirect bypass** (`scripts/web_clip.py:72-86`) : `_is_safe_url()` valide l'URL initiale, mais `httpx.get(..., follow_redirects=True)` suit les 3xx sans revalider. Un serveur public peut renvoyer `302 Location: http://169.254.169.254/...` et contourner la protection. **Critical**. Fix : `follow_redirects=False` + boucle manuelle qui re-valide chaque `Location`, ou transport httpx custom.
- **DNS rebinding TOCTOU** (`scripts/web_clip.py`) : `socket.getaddrinfo()` dans `_is_safe_url` puis nouvelle résolution dans `httpx.get` — fenêtre d'attaque avec TTL=0. **Critical**. Fix : résoudre une fois, forcer httpx à se connecter à cette IP via transport custom.
- **TOCTOU `.env`** (`setup.sh:step7_llm_config`) : entre `> "$VAULT_PATH/.env"` (create avec umask 077) et `chmod 600` qui suit, il y a une fenêtre courte mais réelle sur FS multi-user. Fix : pattern `mktemp + chmod 600 + mv` atomique.
- **`pytest` dans `requirements.txt` prod** (Batch 5 §5.2) : pollue les installs utilisateur (`~/.xais-brain-venv`). Fix : extraire vers `requirements-dev.txt`, setup.sh ne l'installe pas.
- **`socket.getaddrinfo()` sans timeout** (`scripts/web_clip.py`) : peut pendre un batch ou la CI si DNS ne répond pas. Fix : `socket.setdefaulttimeout(5)` local.
- **`ValueError(f"URL refusée : {url}")`** (`scripts/web_clip.py`) : ré-embarque l'URL utilisateur dans le message d'erreur. Incohérent avec l'effort `type(exc).__name__` fait dans `file_intel.py`. Fix : message générique.
- **Kepano `git clone` sans `--depth=1`** (`setup.sh:704` après le pin SHA) : on télécharge tout l'historique pour juste checkout un commit. Impact perf install. Fix : `git init + fetch --depth=1 origin <SHA> + checkout`.
- **Test live DNS `test_is_safe_url_accepts_public_https`** : appelle un vrai `getaddrinfo("example.com")` → flake possible en CI sandboxé/offline. Fix : mocker `socket.getaddrinfo`.
- **`paragraphs_per_docx_page: 30` dead clé** dans `_DEFAULT_PRICING` et `vault-template/.claude/pricing.json` : plus lue depuis §2.1 (formule DOCX alignée sur PDF). Confuse un user qui tune son pricing.
- **`Bash(rm:*)` + `Bash(rm -rf:*)` doublon potentiel** dans `settings.json` deny : si Claude Code matche par prefix, `rm -rf` est couvert par `rm`. À vérifier ; retirer le doublon si prefix match.
- **Pas de `pyproject.toml` / packaging** : `conftest.py` factorise `sys.path.insert` mais masque l'absence de packaging propre. Batch 8 refacto pourrait introduire `pip install -e .` avec `[tool.setuptools.packages.find]`.
- **Dual `open:true` résiduel dans `~/Library/Application Support/obsidian/obsidian.json`** (OBSIDIAN GLOBAL + OBSIDIAN-BRAIN) — inchangé depuis handoff 011, Xavier doit le corriger manuellement via Obsidian UI.

**Risques hérités encore ouverts** (repris du handoff 011, re-vérifiés) :
- **Dette infra héritée** (handoff 011 / 010 / 005 / 006) : migration GitHub Actions → Node 24 (deadline juin 2026), `$XAIS_BRAIN_REPO` toujours non fixée, skills `/client` et `/project` non testés runtime, `setup.sh` désormais ~1020 lignes (plus gros qu'en handoff 011 suite à Batch 1 + 4). Inchangé — partiellement aggravé (setup.sh plus long).
- **Piste 5 (demo GIF VHS)** toujours reportée depuis handoff 008 (plus de 10 jours). Inchangée — feature marketing non livrée.
- **Narrative README/QUICKSTART non testée par tiers externe** (handoff 011) : inchangé — Xavier reste seul utilisateur. `/Users/xais/Desktop/OBSIDIAN-BRAIN` non utilisé depuis son onboarding.
- **`.env` vide sur vault test Xavier `/Users/xais/Desktop/OBSIDIAN-BRAIN`** : inchangé — aucune clé API ajoutée, `xb intel` toujours indisponible.

### Prêt pour la prod ?
- [x] Tests OK (103 pytest + 58 bash asserts, CI macOS + Linux verte sur run `24797537998`)
- [x] Build OK (CI complète 1ère fois avec `pytest tests/ -v`)
- [ ] Testé manuellement — `/XD-build` a produit le code, pas exécuté un onboarding E2E. Le vault test de Xavier (`/Users/xais/Desktop/OBSIDIAN-BRAIN`) n'a pas été mis à jour ; il n'a pas relancé `setup.sh` pour valider que les nouveaux scripts tournent.
- [x] Pas de debug oubliés
- **Décision** : **PARTIEL** — batches 1-5 poussés, CI verte, findings audit remediés à 90 %. Les 4 critical/warning résiduels (SSRF redirect, DNS rebinding, TOCTOU .env, pytest prod) sont non bloquants pour un user casual mais doivent être traités avant un audit externe. Pas de test manuel end-to-end sur le nouveau code.

### Fichiers clés

**Créés/modifiés cette session** :
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/specs/todo/07-health-audit-fixes.md` — plan créé puis archivé en `specs/done/` après livraison
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/specs/done/07-health-audit-fixes.md` — plan 07 complet, référence historique pour les critères de succès
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/scripts/web_clip.py` — `_is_safe_url()` (SSRF basic) + `fetch_and_extract` qui lève `ValueError` sur URL refusée. **2 critical review à durcir en 07.1 : redirects httpx + DNS TOCTOU**.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/scripts/vault_audit.py` — fix `detect_stale_to_verify` (skip `archived`). Dette handoff 010/011 résolue.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/scripts/budget.py` — formule DOCX alignée sur PDF, `format_inbox_warning` supprimé, `seconds_per_file_avg` retiré. Dette review 6F résolue.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/scripts/file_intel.py` — docstring `announce_budget` clarifiée, `except Exception` log `type(exc).__name__`, `prompt_checkpoint` retourne `False` sur réponse ambiguë. Dette review 6C résolue.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/setup.sh` — `.env` chmod 600 + umask 077 + `unset API_KEY`, Kepano pin SHA. **TOCTOU résiduel ligne 586 à durcir en 07.1**.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/vault-cli.sh` — 4 invocations `python3 -c` passent `$cfg` via argv (anti-injection).
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/.github/workflows/ci.yml` — `pytest tests/ -v` (macOS + Linux). CI révèle enfin la vérité.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/requirements.txt` — `pytest>=7.0` ajouté. **À extraire vers requirements-dev.txt en 07.1**.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/tests/conftest.py` — nouveau, factorise `sys.path.insert`.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/vault-template/.claude/settings.json` — allow `Bash(mkdir/mv/chmod/cp:*)`, deny `Bash(rm:*)`.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/vault-template/.claude/skills/import-vault/SKILL.md` — 8 → 12 skills canoniques partout.
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/vault-template/GUIDE.md` — 8 détections dont "Notes peu liées".
- `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/README.md` + `ARCHITECTURE.md` — docs alignées (10 étapes, pyyaml retiré, budget.py au install manuel).

**Spec à créer pour la prochaine session** : `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/specs/todo/07.1-hardening-ssrf-env-packaging.md` pour les 4 critical/warning résiduels.

**Handoff précédent** : `/Users/xais/Dev/XAIS_2ND_BRAIN/xais-brain/specs/handoffs/011-2026-04-22-plan-06-complet-wipe-reinstall.md` — contexte onboarding fresh + bug Obsidian running + wipe des artifacts.
