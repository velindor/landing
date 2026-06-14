# AGENTS.md - Velindor landing

## Project in 10 lines
This repo is the Velindor landing site: a small Jekyll site served by GitHub Pages
at velindor.com, structured like codatus.com. A shared layout and includes provide
the head, header, and footer; one stylesheet holds all the CSS; and the content
pages carry front matter. There is no JS framework, no bundler, and no CSS
preprocessor.

The site exists to run one thing: a pre-launch smoke test for a net-worth tracker.
A landing page captures an email and hands off to a checkout page that offers an
optional founding seat (€49 via Stripe). That is the whole product right now.

The full project brief is the source of truth. The guardrails below are the parts
of it an agent must not get wrong. When in doubt, ask the user, do not guess.

## The three guardrails that override everything

These are not style preferences. They are the reason this project is shaped the way
it is, and two previous products died from ignoring them.

1. **Do not build the product.** The only authorized work is the landing page, the
   checkout page, the legal pages, and the wiring around them. The actual tracker
   (snapshot grid, pricing engine, scenario engine) is gated and unbuilt. Do not
   start it, do not scaffold it, do not "just sketch" it. The gate is: 5 or more
   founding payments within roughly 3 weeks of ads starting. Email signups are not
   payments. Many signups with zero payments means stop, not "almost."

2. **Do not relitigate locked decisions.** The architecture and scope are fixed:
   snapshot-based not transaction-based, no imports of any kind, never any bank or
   account linking, local-first, ECB FX, deterministic scenario engine. There is an
   explicit scope floor of things that are NOT in v1 (Monte Carlo, tax engines,
   importers, mobile apps, a free tier). If a task asks you to add one of these,
   stop and surface it rather than quietly complying.

3. **Follow the voice rules in all new copy.** Plain, specific, numerate, honest.
   Hard bans: no em dashes anywhere (use commas, colons, periods, parentheses), no
   stacked three-fragment phrases, no "not X but Y" reframes, no banned marketing
   vocabulary (seamless, robust, leverage, unlock, effortless, and the like). Use
   digits for numbers. No fake testimonials, counters, or scarcity. Pre-order
   framing is always explicit ("Product preview, launches Q3 2026"). The em-dash ban
   applies to code, comments, and commit messages too, not only page copy.

## Operating protocol (must follow)

This protocol assumes you are running inside the Claude Code VS Code extension,
talking to the user in chat. All design briefs, questions, and approvals happen in
chat. If run non-interactively with no chat channel, halt at any approval gate and
surface the brief or summary as output rather than proceeding.

### Phase 1 - Clarify and design (no code)

Skip the design brief only for changes that cannot affect what a visitor sees or
does: typo fixes in comments, formatting, a dependency bump. Anything that touches
copy, layout, the funnel wiring, the legal pages, or the deploy config gets a brief,
even a three-sentence one.

**Step 1: Clarify inputs.** If the task is missing information (intended copy,
which page, the exact behavior), ask the user short questions and wait. If the task
is clear, skip to Step 2.

**Step 2: Present a design brief in chat.** Make it concrete enough that the user
can predict the result. Use these headers:

1) Problem statement: what is changing and why (1 to 3 short paragraphs).
2) Pages and sections touched: exact files and the specific blocks involved (for
   example `index.html`, the hero form; `thanks.html`, the Stripe button).
3) Precedents to follow: the existing pattern you will match (the brand palette in
   the `:root` block, the existing form markup, the footer). Cite the file and the
   lines you looked at, not only the ones you adopt.
4) Proposed solution: the change end to end, including any copy, in full, so the
   user can read the exact words before they ship.
5) Funnel and wiring impact: does this affect the email handoff, the redirect
   target, the Stripe attribution params, analytics, or noindex. State "none" if so.
6) Verification plan: how you will confirm it works (see Phase 2.5). For copy
   changes this includes a voice-rules check.

End with: "Reply OK to proceed. If you want changes, tell me what to adjust." Do
not write code until the user replies with an explicit OK.

### Phase 2 - Implement

After approval:

1. Start from a clean, up-to-date main: `git checkout main && git pull origin main`,
   then create a new branch. Never reuse a branch from a previous PR. Branch name
   follows `type/short-description` using the same types as PR titles (for example
   `feat/founding-faq`, `fix/stripe-redirect`, `copy/hero-headline`).
2. Make the change in the right place: shared chrome in `_includes/`, all CSS in
   `style.css`, page content in the page file. Do not reintroduce per-page `<style>`
   blocks or a second copy of the header. The two funnel pages (`index.html`,
   `thanks.html`) are tuned artifacts; touch only what the brief covers.
3. Do not commit yet. Leave changes uncommitted so the user can review the diff in
   their IDE.

### Phase 2.5 - Verification

There are no unit tests here. Verification is direct observation. Before presenting
the summary, do the relevant checks and report what you saw:

- **Visual and structural:** open the changed page and confirm it renders, the
  layout holds at mobile width (the pages have breakpoints around 560 to 760px),
  and the change matches the brief. Note anything off.
- **Funnel integrity (if the wiring was touched):** walk the path. The landing form
  subscribes the email to Kit (field `email_address`, form id in the gate-1 script),
  then redirects to the checkout carrying `?e=<email>`. On the checkout, the
  `STRIPE_BASE` constant and the button `href` must stay byte-identical, and the
  Stripe link is built with a `client_reference_id` (`founding` for the immediate
  path, `founding-confirmed` after double opt-in) plus a prefilled email. `THANKS_URL`
  in `index.html` must resolve to the checkout (`thanks`). Both checkout states render
  from `_includes/checkout.html`. The checkout (`thanks` / `confirmed`) and the
  post-payment page (`welcome`) stay `noindex`.
- **Copy:** read the new words against the voice rules above. Confirm no em dashes,
  no banned vocabulary, explicit pre-order framing, digits for numbers.
- **Links:** internal links are extensionless (`/privacy`, `/terms`, `/thanks`,
  `/welcome`), no `.html` (GitHub Pages and `jekyll serve` both serve those). Footer
  Privacy/Terms resolve; the wordmark links home.

Report each check with what you actually observed, not a claim that it "should"
work. If you could not run a check, say so.

### Phase 3 - Review, commit, and open PR

1. Present a summary of all changes: files touched and what each does.
2. Wait for the user to review the uncommitted diff in their IDE. If they request
   changes, apply them and re-verify before proceeding.
3. Once the user approves, commit to the feature branch with a message following the
   PR title convention.
4. Explicitly ask: "Ready to push and open the PR?"
5. **Do not push or open a PR until the user confirms.** If the user declines, ask
   what needs to change. Pushing without an explicit go-ahead is never allowed.
6. After confirmation, push the branch and open a pull request targeting `main`.

**PR requirements:**
- **Title** follows conventional commits: `type: description` (for example
  `feat: add founding FAQ entry`, `fix: correct checkout redirect target`,
  `copy: tighten hero subhead`). Allowed types: `feat`, `fix`, `copy`, `docs`,
  `style`, `chore`.
- **Description** is self-contained: it restates the problem, the approach, and the
  files changed, so the PR makes sense without reading the chat.
- **Do not** add `Co-Authored-By` trailers to commits.
- **Do not** add "Generated with Claude Code" or any AI attribution to commits or
  PR descriptions. No agent attribution appears anywhere in this repo, its history,
  or its pull requests.

### Handling PR review comments

- Push follow-up commits to the same branch, one concern at a time.
- Do not force-push or squash on the branch. The user needs to see what changed
  since their review.
- If a comment requires a real change of approach (not just a tweak), explain the
  updated plan in chat before implementing it.
- If a comment seems wrong or based on a misread, say so with your reasoning before
  changing anything. Do not silently implement a change you believe is wrong.
  Correctness is the goal, not compliance.

### Stop conditions

Halt and ask rather than continuing if any of these happen:
- A task would start building the actual product (any item in Phase 8 of the brief).
- A task would add something from the locked scope floor, or reopen a locked
  architecture decision.
- A change would alter the funnel wiring in a way the brief did not describe (the
  redirect, the Stripe attribution, the double opt-in posture, the noindex).
- You would need to make a legal claim on the privacy or terms page that you cannot
  source (for example anything about escrow, or a guarantee Stripe does not make).
- A change touches the carefully tuned funnel pages beyond what the brief covers.

Anything that silently expands scope or adds risk is a pause, not a best guess.

---

## Repository layout

This is a Jekyll site built by GitHub Pages, structured like codatus.com: a shared
layout and includes, one stylesheet, and content pages that carry front matter.

```
.
├── _layouts/
│   └── default.html    # the HTML shell: head include, header, {{ content }}, footer
├── _includes/
│   ├── head.html       # meta + canonical (extensionless) from front matter, favicon, GoatCounter, fonts, style.css
│   ├── header.html     # the wordmark + top CTA (single source of truth, same on every page)
│   ├── footer.html     # the footer (single source of truth, same on every page)
│   └── checkout.html   # shared founding-seat body for thanks + confirmed (gate-2 Stripe link build)
├── index.html          # landing page: front matter + content (hero, chart, beats, founding, FAQ, Kit gate-1 script)
├── thanks.html         # checkout, gate 2, unconfirmed state (renders checkout.html; noindex)
├── confirmed.html      # checkout after double opt-in (renders checkout.html, confirmed=true; noindex)
├── welcome.html        # post-payment confirmation page (noindex, body_class: checkout)
├── privacy.html        # privacy policy (noindex, body_class: doc)
├── terms.html          # terms (noindex, body_class: doc)
├── style.css           # the one stylesheet: brand tokens, chrome, and every page's components
├── velindor-logo.svg   # logo / favicon source
├── favicon-512.png     # raster favicon export
├── CNAME               # custom domain: velindor.com
├── _config.yml         # Jekyll config (excludes Gemfile, AGENTS.md, assets)
├── Gemfile             # github-pages gem
├── .gitignore
└── AGENTS.md
```

Each content page is front matter plus the inner HTML of `<main>`. The shell, head,
header, and footer come from the layout and includes, so the header and footer are
identical everywhere by construction. Page width is set by `body_class`: default
1040px, `checkout` 680px, `doc` 720px (see the top of `style.css`). Front matter
keys the layout reads: `title`, `description`, `noindex`, `body_class`.

## Style preferences (HTML / CSS)

- **Shared chrome, one stylesheet.** Header, footer, and head live in `_includes/`;
  all CSS lives in `style.css`. Do not reintroduce per-page `<style>` blocks or
  inline a second copy of the header. If a component is new, add its rules to
  `style.css` near the related section, reusing the brand tokens. Page-specific
  differences are handled with a `body_class` prefix (for example `.checkout .hero`),
  not a separate stylesheet.
- **No build framework.** No JS framework, no bundler, no CSS preprocessor. The page
  scripts (gate-1 redirect, gate-2 Stripe link) stay as small inline `<script>`
  blocks at the end of their page's content.
- **Brand palette** (defined as tokens in `:root` in `style.css`): navy `#0E1B4D`, ultramarine
  `#2B59E0`, deep `#2348BD`, cyan `#33C2FF`; paper `#FAFAF7`, ink `#161A1F`, grid
  `#EDEFE7`. Reuse these variables, do not hardcode new hex values.
- **Type:** Space Grotesk for display, Inter for body, IBM Plex Mono for labels and
  numbers. Loaded from Google Fonts in `_includes/head.html`.
- **Never use em dashes.** Hyphens, commas, colons, periods, parentheses only.
  Everywhere: copy, code, comments, commit messages.
- Keep markup semantic and accessible: real `<form>`, `<label>`/`aria-label`,
  visible focus styles (already present), and `prefers-reduced-motion` honored for
  the chart animation.

## Do-s
- Read the project brief before non-trivial work; treat its locked sections as fixed.
- Keep the funnel wiring honest and intact: matching redirect target, identical
  Stripe link in both spots, noindex on the checkout page.
- Surface disagreement with a review comment before implementing it.
- Run the verification checks and report what you actually observed.

## Don't-s
- Don't build the product or anything from the scope floor.
- Don't push or open a PR without the user's explicit say-so.
- Don't add AI attribution or `Co-Authored-By` trailers anywhere.
- Don't reintroduce per-page `<style>` blocks or duplicate the header/footer; the
  shared layout and includes are the single source of truth.
- Don't use em dashes, banned marketing vocabulary, or invent testimonials, counters,
  or scarcity.
- Don't make unsourced legal claims on the privacy or terms pages.
- Don't proceed past a stop condition without asking.
