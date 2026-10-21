# Git

> Bootstrap'ta ayarlandı (2026-10-19, barındırma 2026-10-21'de netleşti).
> Barındırma: GitHub — `origin` = `Okan-developer/polsanleacemanager`. Eğitim kaynağı
> `upstream` = `engindemirog/polsanleavemanager` yalnızca okunur; push URL'si bilerek devre dışı.

## Branching
- Varsayılan dal: `main` — GitHub'da **korumalı** (branch protection: doğrudan push yok, yalnızca PR ile merge, `check` ve `doctor` job'ları zorunlu).
- Bootstrap çıktısı tek seferlik `chore/bootstrap` dalından girdi; bundan sonra aşağıdaki türler dışında dal açılmaz.
- `feature/<spec-no>-<short-name>` — **no branch without a spec.** Also for CHANGE-lane mini-specs.
- Refactors: `refactor/<spec-no>-<short-name>` (mini-spec with Changed behavior = none).
- Fixes: `fix/<report-id>-<short-name>` (bug fixes carry a report, not a spec number);
  incidents: `incident/<date>-<short-name>`; trivial changes: `trivial/<short-name>` (see below).
- Dal adları İngilizce ve kebab-case'tir (`feature/0001-solution-skeleton`).

## Commits
- Conventional Commits, with a plan reference: `feat(leave): submit leave request [plan 0002/3]`.
  Scope = modül adı küçük harf (`organization`, `leave`, `calendar`, `web`, `shared`) ya da `build`/`docs`.
- Commit mesajları İngilizcedir (`docs/conventions.md`, Language).
- Agent commits follow the same standard: the agent writes the message, the human approves.

## Forbidden
- Direct commits to the default branch.
- Force push, history rewriting on shared branches. Undo = `git revert` (see recovery R-11).

## Trivial changes
<!-- Set at bootstrap. Applies only to requests the change-request triage rubric
     (workflows/change-request.md) classifies as TRIVIAL: no acceptance criterion changes, no shared code. -->
- Policy: **(1) PR + one reviewer, no spec** — seçildi (bootstrap, 2026-10-19). `trivial/<short-name>`
  dalından PR açılır, bir gözden geçiren onaylar; spec yazılmaz.
- Always: one narrow commit, `scripts/check` green. A second file or a new test means re-triage.

## Pull requests
- `.github/pull_request_template.md` kontrol listesi tamamlanır; en az **bir onaylayıcı**; CI'da
  `scripts/check` (tam profil) ve `scripts/doctor --strict` yeşil (`.github/workflows/check.yml`); **squash-merge**.
- PR açıklamasındaki kanıt `CHECK GREEN (full, …)` satırıdır; `(fast, …)` kanıt sayılmaz.
