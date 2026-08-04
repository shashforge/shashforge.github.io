# ShashForge — daily AI engineering log

Live at **https://shashforge.dev** · Built with Jekyll · Hosted on GitHub Pages

## Publish a new entry (daily, ~2 minutes)

1. Copy `_drafts/TEMPLATE-daily-post.md` into `_posts/` and rename it
   `YYYY-MM-DD-short-slug.md` (e.g. `2026-08-05-eval-harness.md`).
2. Edit the front matter: `title`, `date`, `status`, `topics`.
3. Write the entry.
4. Commit and push (or edit directly on github.com and hit **Commit changes**).
   GitHub Pages rebuilds automatically — live in about a minute.

You can do all of this from the GitHub website or mobile app — no local
tooling required.

## One-time deployment setup

1. Create a **public** GitHub repo (e.g. `shashforge`), push these files to `main`.
2. Repo → **Settings → Pages** → Source: *Deploy from a branch* → `main` / root.
3. In **Settings → Pages → Custom domain**, enter `shashforge.dev`
   (the `CNAME` file in this repo keeps it set).
4. Tick **Enforce HTTPS** once the domain check passes.

### Namecheap DNS (Domain List → shashforge.dev → Advanced DNS)

| Type  | Host | Value                 |
|-------|------|-----------------------|
| A     | @    | 185.199.108.153       |
| A     | @    | 185.199.109.153       |
| A     | @    | 185.199.110.153       |
| A     | @    | 185.199.111.153       |
| CNAME | www  | `<username>.github.io.` |

Remove any parking/URL-redirect records Namecheap added by default.
DNS + HTTPS certificate can take from a few minutes up to a couple of hours
on first setup.

## Local preview (optional)

```bash
bundle install
bundle exec jekyll serve   # http://localhost:4000
```
