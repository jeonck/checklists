# IT Checklists

A practical, open library of IT checklists for engineers, SREs, and security teams —
production readiness, security reviews, cloud migrations, incident response, compliance,
and more.

**Live site:** <https://checklists.metacog.co.kr/>

Every checklist is plain Markdown. Fork it, cut what does not apply, and bring it into
your own runbooks.

## Stack

- [Hugo](https://gohugo.io) (extended) with the [Lotus Docs](https://github.com/colinwilson/lotusdocs) theme, installed as a Hugo Module
- Offline full-text search via FlexSearch
- Deployed to GitHub Pages by [`.github/workflows/hugo.yml`](.github/workflows/hugo.yml) on every push to `main`

## Running locally

Requires Hugo **extended** ≥ 0.140 and a Go toolchain (Hugo Modules need it to resolve the theme).

```bash
hugo mod get -u
hugo server
```

The site is served at <http://localhost:1313/>.

## Adding a checklist

1. Create `content/docs/<category>/<slug>.md`.
2. Copy the front matter and structure from an existing page, for example
   [`content/docs/devops/production-readiness.md`](content/docs/devops/production-readiness.md).
3. Pick a `weight` that is **unique across the whole site** and sits inside the
   category's block (security 110–199, devops 210–299, cloud 310–399, and so on).
   The theme builds Prev/Next navigation from a flat weight ordering, so a duplicate
   weight sends readers into a different category.
4. `icon` must be a valid [Google Material Symbols](https://fonts.google.com/icons)
   ligature name. An invalid name renders as blank space with no build error.

Checklist items use the form:

```markdown
- [ ] **What to verify** — why it matters and what failure it prevents.
```

Boxes are clickable in the browser and progress is saved per page in `localStorage`.

## Licence

Content and code are MIT licensed. See [LICENSE](LICENSE).
