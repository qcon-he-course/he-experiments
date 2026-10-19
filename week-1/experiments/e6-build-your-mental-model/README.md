# Experiment 6: build your mental model

Can you explain the system yourself, from the evidence, and show that the code agrees? This rung gives the agent two skills that draw what it finds: one user journey as a sequence diagram, and the parts of the system as a component view. Each arrow and box must carry its evidence.

## What changes

Rung 5, plus:

- **The `trace-journey` skill** (`layer/skills/trace-journey/`). It finds where a journey starts, follows it with the code graph and Serena, and marks the hops it can't confirm as inferred. It draws an archify sequence diagram with a file and line on every arrow, and writes notes beside it.
- **The `arch-map` skill** (`layer/skills/arch-map/`). The model can't call it by itself: you type `/arch-map`. It draws a component view from CodeBoarding's components.
- **archify**, as a skill and as the `archify` command.
- **Permission to write under `.harness/`** in the clone, and nowhere else (`layer/settings.json`, `layer/rules.md`).
- **CodeBoarding's output** (`codeboarding/`), copied into the clone as `.harness/codeboarding/`. We made it once, with a provider API key. A model named the components, so they are a claim too.

## Run it

From `experiments/ladder/`. Set up first: [SETUP.md](../../SETUP.md). E6 needs `../e5-require-evidence/AGENTS.md`, which is in the repo.

**The journey.**

1. Before the agent draws, write the journey yourself: an owner adds a new pet, from submitting the form to the row in the database. One line per step, with the class and the method.
2. `./run.sh e6`, and trust the folder.
3. Type `/context all` at the very start of the line. `trace-journey` and `archify` are among the skills. `arch-map` doesn't show, because only you can call it.
4. Type `/trace-journey an owner adds a new pet`.
5. When it says the diagram is written, type `/exit`. The diagram exists only in the container until Claude Code exits.
6. Open `../e6-build-your-mental-model/runs/<latest>/files/.harness/diagrams/journey-*.html` in a browser. The notes are in `files/.harness/notes/`.

**The component view.** In another `./run.sh e6`, type `/arch-map`. It writes `files/.harness/diagrams/architecture.html`, with the evidence for each node and edge beside it.

Approve read-only commands, `cgr graph` lookups and `archify`. Writes under `.harness/` don't ask. Deny any other write, build or network call.

**What to look for.** Compare the drawing with your own list and with the code. Four of the journey's steps are done by the framework: Spring calls the controller's `@ModelAttribute` methods before the handler, a formatter converts the pet type, `@InitBinder` installs the validator, and JPA's cascade writes the row. Does the drawing keep those steps, mark them inferred, or leave them out? Do the notes say more than the drawing? For the component view: does it show the order of anything, and where does it say CodeBoarding and the code disagree?

**Unattended.** `./batch.sh claude-opus-5-5 1 e6` runs `/trace-journey an owner adds a new pet` once, in `dontAsk` mode. `scoreboard.sh`'s quality columns are for the six questions, so keep your marks for a drawing in a note of your own.

**Your own CodeBoarding output (optional).** `../ladder/codeboarding.sh` regenerates `codeboarding/`. It calls a model for every component, so it needs exactly one provider API key in your environment: `ANTHROPIC_API_KEY` (a Console key), `OPENAI_API_KEY`, `OPENROUTER_API_KEY` or `GOOGLE_API_KEY`. That key is billed. Never use your Claude subscription token for it.

## What we saw

We ran E6 interactively on Opus 5.5 on 19 October, in the default permission mode: one `/trace-journey` run and two `/arch-map` runs. We ran no unattended batch, so there are no medians. Our two diagrams are in [`results/runs/e6-build-your-mental-model/`](../../results/runs/e6-build-your-mental-model/): open the `.html` files in a browser.

- **The journey.** 3 minutes 17 seconds, 29 tool calls (16 Bash, 6 to Serena, 2 to the graph), 1.44M tokens sent. It wrote only under `.harness/`. The drawing had 8 participants and 17 arrows, each with a file and line.
- **It missed or dashed the framework's steps.** It missed one: Spring calling the `@ModelAttribute` methods on the post. It drew the other three dashed, as inferred, and as arrows from the controller. A sequence diagram has no participant for Spring, so the agent explained the framework's part on a card.
- **The notes knew more than the drawing.** They give all seven steps, with the right mechanism for each, labelled VERIFIED or INFERRED. The format lost what the agent knew about who calls whom.
- **`AGENTS.md` gives part of the last step away.** Its line on the controllers saving the owner, with JPA cascading the rows, is the save step. Don't count that step as found unaided.
- **Two `/arch-map` runs, two maps.** The first drew 8 components and renamed one; the second drew 11 nodes, kept the names and split a component in three. The same skill on the same code gives a different view each time: a component view is a claim. Both said where CodeBoarding and the code disagree. The second found a CodeBoarding relation that points at the wrong class: `VisitController` calls `OwnerRepository` directly. The component views show the parts and what reaches the database and the cache, and none of the order.
- **A probable defect, found by following the evidence** (inferred, not tested). `PetController` turns a database duplicate into a form error only when the message names `unique_owner_pet_name`. MySQL's constraint is unnamed, so on MySQL such a duplicate would surface as an error. Only a race reaches it.

E6 is the most expensive rung. The journey run sent 1.44M tokens and the second `/arch-map` run 1.6M, against a median of 220k for an Opus e5 run. More in [findings](../../results/findings.md).

## Files

| File | What it is |
|---|---|
| `layer/` | What this rung adds to `/etc/claude-code`: the two skills, the `archify` marker file, `rules.md`, `settings.json` |
| `codeboarding/` | CodeBoarding's analysis of PetClinic: five components, from `../ladder/codeboarding.sh` |
| `runs/` | One folder per run, named `<date-time>-<model>`; git ignores it |
