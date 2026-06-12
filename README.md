# Claude Build Walkthrough — Bee Defender 🐝

This repo documents an end-to-end workflow for building a complete, deployed web
game using Claude — from initial idea exploration in chat, to a written design
specification, to a fully automated build with Claude Code. The finished product
is **Bee Defender**, a browser-based physics puzzle: draw ink barriers to protect
a dog from swarming bees and survive the timer.

The code in this repo is the actual output of that process, unmodified.

## The Workflow

### Stage 1 — Concept research (Claude chat, Sonnet 4.6)

The project started as a conversation, not code. Using Claude Sonnet 4.6 in the
chat interface, we discussed existing versions of this style of game on the
internet — line-drawing defense games, their core mechanics, what makes them
fun, and what a self-hosted clone would need. This stage was purely exploratory:
no artifacts, just converging on a concept worth building.

**Takeaway:** let the model survey the genre with you before committing to a
design. It surfaces mechanics and pitfalls you would otherwise discover mid-build.

### Stage 2 — Design documentation (Claude chat, Sonnet 4.6)

Still in chat, Sonnet 4.6 was used to turn the concept into a complete game
design document: tech stack, file structure, module-by-module specifications,
physics constants, scoring formulas, API contracts, and a "Known Issues" section
listing bugs to avoid. The goal was a spec precise enough that a coding agent
could build the entire project from it without asking questions.

Key properties of a build-ready design doc:

- **Cross-file contracts spelled out** — shared constants, function signatures,
  and which file owns each one
- **Exact formulas** — difficulty scaling, scoring, seeded RNG chains
- **A known-bugs section** — mistakes from earlier attempts, documented so the
  rebuild avoids them
- **No ambiguity** — "Follow it exactly" beats "use your judgment" when the
  builder runs unattended

### Stage 3 — Project setup

A project folder was created on the build machine and the design document was
uploaded into it. That folder plus the doc was the entire starting state — no
boilerplate, no scaffolding.

```
bee-game/
└── design-doc.md
```

### Stage 4 — Automated build (Claude Code, Fable 5)

Claude Fable 5, running in Claude Code, was given a single prompt: read the
design doc in full, then build every file in a specified order, syntax-checking
each one before moving on, with no questions asked. The build sequence:

1. `package.json` → `npm install`
2. `database/db.js` → verified by requiring it in Node
3. Express server + API routes
4. HTML + CSS
5. The ten client JS modules, dependency-first (utils → level → physics → bee →
   input → renderer → leaderboard → share → ui → game)
6. Validation: `node --check` on every file, then live `curl` tests against both
   API endpoints
7. Process management: pm2 + `ecosystem.config.js`

The agent then created the GitHub repo, committed, and pushed. Post-build fixes
(also done by the agent) included a CSP misconfiguration that blocked the game
from loading over plain HTTP, and a sanitization pass: secrets moved out of the
code into environment/runtime configuration, internal docs removed from the
repo, and git history rewritten so none of it remains.

**Takeaway:** an explicit build order with a per-file verification step lets the
agent run end-to-end without supervision, and the Known Issues section of the
design doc genuinely prevented every listed bug from reappearing.

## The Game

- Vanilla HTML/CSS/JS frontend, Matter.js physics (served locally — no CDNs)
- Seeded procedural levels — deterministic and shareable via URL
- Node.js + Express backend with a SQLite (better-sqlite3) leaderboard
- No cookies, no tracking; IPs stored only as salted SHA-256 hashes, with the
  salt generated at runtime and never committed

### Run it locally

Requires Node.js 18+.

```bash
npm install
npm start
# open http://localhost:3000
```

### Production

```bash
npm install -g pm2
pm2 start ecosystem.config.js
```

The SQLite database and the generated salt live in `./data/` (gitignored).

## Repo Status / Roadmap

- [x] Stage 1–4 complete; game built, deployed, and published
- [ ] Sanitize the original design document (remove internal links and
      machine-specific details) and add it to this repo as the Stage 2 artifact
- [ ] Annotate the build transcript into a step-by-step tutorial
