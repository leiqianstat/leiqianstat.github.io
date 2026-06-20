# Agent Guidelines for al-folio (v1.x)

`al-folio` is the **starter repo** for the pluginized v1 architecture.

## Read This First

- Start with `.github/copilot-instructions.md` for architecture, ownership boundaries, and CI expectations.
- Use `docs/BOUNDARIES.md` as the source of truth for starter-vs-plugin ownership.
- Use `.agents/skills/al-folio-bootstrap/SKILL.md` for new-site setup tasks.
- Use `.agents/skills/al-folio-v1-migration/SKILL.md` for customized fork migrations.
- `.codex/skills` and `.claude/skills` are symlinks to `.agents/skills` for agent-specific discovery.

## What This Repo Owns

- Starter wiring (`Gemfile`, `_config.yml`)
- Starter content and documentation
- Cross-plugin integration tests
- Visual regression tests

Runtime/component logic belongs in owning plugin repos (`al_folio_core`, `al_folio_distill`, `al_search`, `al_icons`, `al_cookie`, and other `al-*` gems).
Long-form documentation lives in `docs/`; keep this root file as the short discovery entry point for coding agents.

## Validated Local Command Set

Run from repo root:

```bash
npm ci
npm run lint:prettier
npm run lint:style-contract
bundle exec jekyll build --baseurl /al-folio
bash test/integration_comments.sh
bash test/integration_plugin_toggles.sh
bash test/integration_distill.sh
bash test/integration_bootstrap_compat.sh
bash test/integration_upgrade_cli.sh
npx playwright install chromium webkit
npm run test:visual
bundle exec al-folio upgrade audit
bundle exec al-folio upgrade overrides audit
bundle exec al-folio upgrade report
docker compose up -d
curl -fsS http://127.0.0.1:8080/al-folio/ >/dev/null
docker compose logs --tail=80
docker compose down
```

Docker note: v1 uses `/srv/jekyll/bin/entry_point.sh` and serves from container-local `/tmp/_site` to avoid host bind-mount write deadlocks.

## Agent Routing Rules

- If change is starter wiring/docs/integration/visual testing: edit here.
- If change is runtime feature behavior: route to owning plugin repo.
- Do not add starter-local npm build scripts for theme/runtime assets.
- Keep docs aligned with pluginized v1 ownership.
- If you create or keep local overrides of plugin-owned files, run `bundle exec al-folio upgrade overrides audit` and commit `.al-folio-overrides.yml` after review.

## Cursor Cloud specific instructions

Environment is pre-provisioned by the startup update script (`bundle install`, `npm ci`, `pip3 install --user nbconvert`). Standard commands live in the "Validated Local Command Set" above and in `CLAUDE.md`; this section only covers non-obvious caveats.

- Toolchain: system Ruby 3.2.3 + Bundler 4.0.6 (CI uses 3.3.5; 3.2.3 builds/serves fine here), Node 22, Python 3.12. `bundle` is on the default PATH and Bundler installs gems to `~/.bundle-gems` via a global `~/.bundle/config` — no `GEM_HOME` or shell init is required for `bundle`/`bundle exec`.
- UTF-8 locale is required for `bundle exec jekyll build`/`serve` (content contains emoji/accents). The VM default `LANG=en_US.UTF-8` works; if a stripped env throws `invalid byte sequence in US-ASCII`, run with `LANG=C.UTF-8 LC_ALL=C.UTF-8`.
- `nbconvert` lives in `~/.local/bin` (on PATH via `~/.bashrc`); it is optional — if missing, builds continue and skip notebook rendering with a warning. `rendercv`/`scholarly` from `requirements.txt` are not installed by default (only needed to regenerate CV PDFs / scholar citations).
- Dev server: `bundle exec jekyll serve --host 0.0.0.0 --port 4000 --baseurl /al-folio` → http://localhost:4000/al-folio/ (note the `/al-folio` baseurl). Content edits hot-reload; `_config.yml` edits require a server restart. Harmless `listen` "directory is already being watched" warnings come from the `.codex/skills`→`.agents/skills` symlink and can be ignored.
- Pre-existing failures unrelated to environment setup (do not "fix" via env changes): `npm run lint:style-contract` fails because the repo intentionally commits local overrides (`_includes/header.liquid`, `_includes/footer.liquid`, `_layouts/about.liquid`); `test/integration_comments.sh` and `test/integration_distill.sh` fail because this snapshot has no `_posts/` sample content (the `blog/2021/distill` and `blog/2022/giscus-comments` pages are never generated). The other integration tests, `lint:prettier`, and `jekyll build` pass.
