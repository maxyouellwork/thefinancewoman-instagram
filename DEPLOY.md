# Deploying anything called The Finance Woman

**This file is identical in every TFW repo.** Whichever one you have landed in, this
is the whole picture — you are probably not in the repo you need.

Nine live things carry the TFW name and they sit in five repos. That is the reason
sessions kept deploying the wrong thing, or deploying by hand from a branch and
leaving no trace of it.

## The rule

Every one of them deploys with **one dispatch command**, and nothing deploys on push.

```bash
gh workflow run <workflow> --repo maxyouellwork/<repo>
gh run watch --repo maxyouellwork/<repo>     # follow it
```

## What is live, and the command for it

| Live | Repo (and directory) | Command |
|---|---|---|
| **thefinancewoman.co.uk** — the main site | `thefinancewoman-refresh` | `gh workflow run deploy.yml --repo maxyouellwork/thefinancewoman-refresh` |
| **go.thefinancewoman.co.uk** — checkup, partner pages, embed | `thefinancewoman` (repo root) | `gh workflow run deploy.yml --repo maxyouellwork/thefinancewoman` |
| **scorecard-nurture** — checkup API, emails, `/referrers` | `scorecard-nurture` | `gh workflow run deploy.yml --repo maxyouellwork/scorecard-nurture` |
| **tfw-briefing-editor** — Romany edits the monthly briefing | `thefinancewoman-workspace/newsletter-editor` | `gh workflow run deploy-newsletter-editor.yml --repo maxyouellwork/thefinancewoman-workspace` |
| **tfw-newsletter-review** — newsletter review pages | `thefinancewoman-workspace/newsletter-review` | `gh workflow run deploy-newsletter-review.yml --repo maxyouellwork/thefinancewoman-workspace` |
| **tfw-dashboard** | `thefinancewoman-workspace/dashboard` | `gh workflow run deploy-dashboard.yml --repo maxyouellwork/thefinancewoman-workspace` |
| **tfw-pet-expo-live** — the live wall for the 3 Oct talk | `thefinancewoman-workspace/pet-expo-live` | `gh workflow run deploy-pet-expo-live.yml --repo maxyouellwork/thefinancewoman-workspace` |

Two more that are not deployed from a workflow:

- **`scorecard-engine`** is a library, not a deployable. It has no deploy. It is a
  `file:../scorecard-engine` dependency of `thefinancewoman` and `scorecard-nurture`,
  so a change here is only live once **both of those are redeployed**. That is the
  step people forget.
- **`thefinancewoman-instagram`** is a local carousel-approval gallery. Nothing to
  deploy.

## The account trap, which is the one that actually bites

There are two Cloudflare accounts and deploying to the wrong one does not fail. It
quietly creates a second, empty project of the same name, and the real site carries
on serving the old build while you read a success message.

- **Personal `38be864b…`** — every TFW property. All seven above except one.
- **Firm `3ba7e551…`** — **`scorecard-nurture` only**, plus IBT and Off The Field.

> `claude-online/bin/set-deploy-secrets` defaults every repo to the personal account
> **and its repo list includes `scorecard-nurture`**. Running it unqualified points
> the nurture worker — the checkup API for a regulated firm — at the wrong account.
> Pass the firm account explicitly for that one repo.

`keys.env`'s unprefixed pair is mismatched: `CLOUDFLARE_API_TOKEN` is the personal
token, `CLOUDFLARE_ACCOUNT_ID` is the firm account. Always use the explicit
`CLOUDFLARE_PERSONAL_*` or `CLOUDFLARE_FIRM_*` pair.

Wrangler is pinned to 4.127.1 in every workflow. Below 4.124 it ignores
`CLOUDFLARE_ACCOUNT_ID` for Pages entirely and uses the token's default account.

## Deploying by hand

Each repo also has `npm run deploy`, which runs the same command the workflow does.
Use it when you do not want to wait on Actions. Two things it will not do for you:

- **It does not check the Functions shipped.** On the go. site, wrangler resolves
  `functions/` relative to the working directory, not the asset directory. Deploy
  from anywhere but the repo root and it uploads every file, reports success, and
  ships no API — the checkup then 404s on `/api/score`. The workflow POSTs to
  `/api/subscribe` afterwards and fails on a 404. Deploying by hand, you check that
  yourself.
- **It does not stop a migration going out.** Every workflow refuses a diff touching
  `migrations/`, because D1 migrations are applied by hand first, against live client
  data. Deploying by hand skips that guard.

## What is live is what is on `main`

It was not, until 1 Sep 2026, and that cost real time. The go. site was served from
`claude/romany-presentation-c4bozk` for three days while Cloudflare's dashboard
reported `branch=main` for those deploys — the deploy command passes `--branch main`,
so the label reflects the flag, not the working tree. Anyone who read the dashboard
and deployed `main` to "restore" it would have removed live pages while believing
they were doing nothing.

So: **branch, open a PR, merge to `main`, then dispatch.** If you find yourself
deploying from a branch, that is the smell.

## Two dead things, left alone deliberately

Neither has source in any repo, so nothing can rebuild them. They are listed so the
next session does not spend an hour working out what they are.

- **`tfw-pet-talk`** (Pages, personal) — a scratch preview of the pet talk from
  26–28 Aug, only on `.pages.dev`. Superseded by `go.thefinancewoman.co.uk/pet-talk/`.
- **`tfw-temp-share`** (Worker, personal) — created 14 Apr 2026, untouched since,
  returns 404.

Deleting them is Max's call, not a session's.
