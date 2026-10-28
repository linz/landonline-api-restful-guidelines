LINZ's RESTful API guidelines — a fork of Zalando's `restful-api-guidelines`, merged with NZ Government API Standards. This is a documentation repo (AsciiDoc), not application code: it builds an HTML/PDF/EPUB guidelines site, published to GitHub Pages.

## Layout

- `index.adoc` — top-level document that includes all chapters.
- `chapters/*.adoc` — one file per guideline chapter (e.g. `http-status-codes-and-errors.adoc`, `pagination.adoc`, `security.adoc`, `deprecation.adoc`). Each rule is anchored with `[#<id>]` and must have a **unique** numeric ID — this is enforced by `make check-rules`.
- `models/` — reusable OpenAPI schema fragments (e.g. `problem-1.0.x.yaml`, `money-1.0.0.yaml`) referenced by the guidelines and copied into the built site.
- `legacy/` — previously published guideline versions, copied verbatim into `output/` so old links keep working.
- `scripts/generate-rules-json.sh` — extracts all rule IDs/metadata from `chapters/*.adoc` into `output/rules.json`, consumed by tooling like Lilly ([landonline-openapi-linter-lilly](https://github.com/linz/landonline-openapi-linter-lilly)).
- `scripts/new-rule-id.sh` / `Makefile`'s `next-rule-id` — get the next free rule ID when adding a new rule.
- `sass/`, `zalando.css` — site styling; `scripts/build-css.sh` regenerates CSS from the Zalando stylesheet-factory.

## Development commands

```bash
./build.sh              # recommended way to build the HTML site -> ./docs/index.html
make clean; make html    # legacy make-based HTML build (needs Docker)
make all                 # HTML + PDF + EPUB3 + rules.json into ./output (needs Docker + jq)
make check-rules         # validate rule ID anchors are unique and correctly formatted — run before committing chapter changes
npm install -g markdownlint-cli && make lint    # markdownlint chapters/*.adoc
```

The Docker-based `make` targets run `asciidoctor/docker-asciidoctor` — no local Ruby/asciidoctor install needed, just Docker.

## Key conventions

**Rule ID uniqueness**: every rule anchor must be `[#<number>]` (not `[123]` or `[ #123 ]`) and IDs must not be duplicated across chapters — `make check-rules` (run in CI before every build) fails the build otherwise. Get a new ID with `make next-rule-id` rather than guessing one.

**SHA pinning**: every `uses:` in workflows must pin a full commit SHA with the version as a trailing comment (e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # 4.2.2`). Never reference a tag or branch directly.

**Publishing**: `.github/workflows/build.yml` builds on every push/PR to `main` and deploys to GitHub Pages only from `main`, via `actions/upload-pages-artifact` + `actions/deploy-pages`.

**LINZ additions vs. upstream Zalando content**: this repo is a living fork — when extending guidelines, prefer adding LINZ/NZ Government-specific content clearly rather than silently rewriting inherited Zalando chapters, to keep future upstream diffs reviewable.
