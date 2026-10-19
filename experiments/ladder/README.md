# The ladder: experiments 0 and 2 to 6 in a plain Claude Code container

Each run starts Claude Code 2.1.281 in a container, on a fresh clone of seeded PetClinic at `b107a86`. What changes from rung to rung is what sits in `/etc/claude-code`, Claude Code's managed configuration:

| Rung | Command | Loads, on top of the rung below |
|---|---|---|
| 0 | `./run.sh e0` | The floor only |
| 2 | `./run.sh e2` | The instruction file, as the managed `CLAUDE.md` (`../e2-tell-it-how-to-behave/layer/rules.md`) |
| 3 | `./run.sh e3` | Serena (Java language server) as an MCP server, a routing rule, and permission to use its lookups without asking |
| 4 | `./run.sh e4` | code-graph-rag as an MCP server with Memgraph, and a routing rule; its natural-language and write tools are denied |
| 5 | `./run.sh init`, then `./run.sh e5` | The evidence rule, and an audited `AGENTS.md`, committed in the clone |
| 6 | `./run.sh e6` | The `trace-journey` and `arch-map` skills, archify, CodeBoarding's output in `.harness/codeboarding/`, and permission to write under `.harness/` |

Experiment 1 runs from its own folder, `../e1-limit-what-it-can-do/`.

Run every command on this page from this folder, with your token exported ([SETUP.md](../../SETUP.md)).

## The floor

The floor is the set of settings we fix for every run. They live in `/etc/claude-code`, as Claude Code's managed settings (`image/floor/managed-settings.json`), and the repository can't change them.

- **Ignored:** the repository's hooks, MCP servers, permission rules and skills. PetClinic ships one hook; it never runs.
- **Off:** auto memory, and the bypass and auto permission modes.
- **Still read:** the repository's files, `.env` included, and its `CLAUDE.md` or `AGENTS.md`.

## How layers work

Each layer is a folder, `<experiment>/layer/`, holding any of `rules.md`, `mcp.json`, `settings.json` and `skills/`. At start-up the container's entrypoint, as root, builds `/etc/claude-code` from the floor and the layers, in ladder order. It then starts Claude Code as the `node` user, so the agent can't change its own configuration.

Edit a layer's files and the next run picks them up, with no rebuild. That is the easiest way to try your own change: change one line in a rung's `rules.md`, and compare the runs before and after.

## Run a rung

1. Run `./run.sh <rung>`.
2. Check the `ladder:` lines it prints before Claude Code starts: the layers, rules, MCP servers and skills.
3. Trust the folder.
4. Check `/context` and `/mcp`.
5. Paste the questions. On macOS, `pbcopy < ../questions.txt` first; elsewhere, copy them from `../questions.txt`.
6. When the answer is in, type `/exit`. Don't press Ctrl-C: `run.sh` saves the agent's files and removes the clone only after Claude Code exits.
7. Run `./show-run.sh <rung>`.

| Rung | What to confirm |
|---|---|
| e0 | `/mcp` shows no servers; `/context` shows no memory files |
| e2 | `/context` lists `/etc/claude-code/CLAUDE.md` with the instruction file's lines |
| e3 | `prepare.log` in the run folder says "Serena index ready"; `/mcp` shows `serena` connected; `show-run.sh` lists Serena calls under "MCP calls" |
| e4 | "graph ready" in `prepare.log`; `/mcp` shows `code-graph-rag` connected. After the six questions, ask: "Use the code graph: what does `VisitController.processNewVisitForm` call?" The answer should cite `callees` |
| e5 | `/context` lists `AGENTS.md`; the answers carry VERIFIED, INFERRED or UNKNOWN labels |
| e6 | Type `/trace-journey an owner adds a new pet`. After `/exit`, open `files/.harness/diagrams/journey-*.html` from the run folder in a browser |

From rung 3, Serena and the graph start before Claude Code's banner, after 10 to 60 seconds.

**Permission prompts.** Interactive runs keep Claude Code's default mode, so you answer each prompt. Approve read-only commands. Deny anything that writes, builds or reaches the network.

**The model.** `LADDER_MODEL` picks it, `claude-opus-5-5` by default: `LADDER_MODEL=claude-haiku-4-5 ./run.sh e0`.

**Experiment 5.** `./run.sh init` starts a bare session for `/init`. Type `/init`, let it write `CLAUDE.md`, then `/exit`. The draft is saved in `../e5-require-evidence/runs/<date-time>-<model>-init/files/CLAUDE.md`. Every e5 and e6 run commits `../e5-require-evidence/AGENTS.md` into its clone. That file is our audited version; to test your own audit, replace it with yours.

**Experiment 6.** In an interactive `./run.sh e6`, `/arch-map` draws the component view from CodeBoarding's output, into `files/.harness/diagrams/architecture.html`.

## Unattended runs

```bash
./batch.sh claude-haiku-4-5 1         # one round: e0 e2 e3 e4 e5 e6, once each
./batch.sh claude-opus-5-5 5 e2 e3    # five rounds of two rungs
./batch.sh claude-opus-5-5 5 e1       # experiment 1's open run, through its open-run.sh
./scoreboard.sh                        # medians per rung and model
```

- **How a run works.** `batch.sh` runs `run.sh` with `LADDER_BATCH=1`, one run at a time, so the shared Memgraph and Serena caches never see two runs at once. Each run uses `claude -p`: the six questions for e0 to e5, and `/trace-journey an owner adds a new pet` for e6. The answer goes to `answer.txt` in the run folder.
- **Order.** Each round runs every rung once, so a slow hour spreads across rungs.
- **Permissions.** Unattended runs use Claude Code's `dontAsk` mode, except `init`, whose `/init` has to write `CLAUDE.md`. A tool call that would ask for permission is refused; read-only shell commands still run. There is no trust prompt. So an unattended run can be shorter than an interactive one.
- **Time cap.** An unattended run is stopped after `LADDER_TIME_CAP` seconds, 1200 by default, and its folder gets `timed-out.txt`.
- **Usage.** Every run counts against your plan ([SETUP.md](../../SETUP.md) has the tokens per run). If a batch hits a usage limit part way, the finished runs are kept, and a shorter batch tops up.
- **The log** of a batch is in `~/.cache/qcon-he/ladder/batch-<date-time>.log`.

## The scoreboard

`scoreboard.sh` counts only unattended runs, the folders with a `prompt.txt`, so your interactive runs stay out of the medians.

| Command | What it shows |
|---|---|
| `./scoreboard.sh` | Medians per rung and model: minutes, tokens sent, cache share, tool calls, files read, MCP calls, and quality where runs have a score |
| `./scoreboard.sh claude-haiku-4-5` | One model; add a rung, such as `e0`, to limit it further |
| `./scoreboard.sh --runs` | One row per run |
| `./scoreboard.sh --nearest claude-opus-5-5 e0` | The folder of the run nearest the median |
| `./scoreboard.sh --rigged e0 e1` | For each rigged item, in how many runs it happened: contained beside open |

The quality columns come from `score.txt` in each run folder: one line, `q1-5=4 q6=5`, which you write after marking the run against [the answer key](../../reference/answer-key.md).

`./show-run.sh` shows one run in full: the latest run of a rung (`./show-run.sh e3`), or any run folder. Add `--answers`, `--replies` or `--calls` to see only those.

## What a run saves

Each run is saved in `<experiment>/runs/<date-time>-<model>/`, which git ignores:

- `config/` holds Claude Code's state, including the session transcript;
- `etc-claude-code/` holds exactly what the layers put in `/etc/claude-code`;
- `layers/` holds the layer files as they were for the run;
- `prepare.log` holds the indexing output for Serena and the graph;
- `files/` holds whatever the agent created or changed in the clone, `.harness/` included;
- `git-status.txt` records the clone's state when Claude Code exited;
- unattended runs add `prompt.txt`, `answer.txt` and `stderr.txt`.

## Settings

All in `settings.sh`, and each can be set in the environment instead.

| Setting | Default | What it is |
|---|---|---|
| `LADDER_MODEL` | `claude-opus-5-5` | The model for `run.sh` |
| `PETCLINIC` | `https://github.com/qcon-he-course/he-petclinic.git` | The seeded repository each run clones |
| `PETCLINIC_COMMIT` | `b107a86` | The pinned seeding commit |
| `LADDER_WORKDIR` | `~/.cache/qcon-he/ladder` | Where clones and batch logs go, outside this repository |
| `LADDER_TIME_CAP` | `1200` | Seconds an unattended run may take before it is stopped |
| `LADDER_KEEP_CLONE` | unset | `1` keeps the clone after the run |

## Files

| File | What it does |
|---|---|
| `run.sh` | One run of a rung; interactive, or unattended with `LADDER_BATCH=1` |
| `batch.sh` | N unattended rounds of the ladder on one model; `e1` runs experiment 1's open run |
| `scoreboard.sh` | Medians per rung and model from the unattended runs, with their scores |
| `show-run.sh` | Experiment 1's `show-run.sh` on a run of the ladder, with the MCP calls counted |
| `codeboarding.sh` | CodeBoarding's analysis for experiment 6, with a provider API key (optional; [SETUP.md](../../SETUP.md)) |
| `stop.sh` | Stops Memgraph. `--all` also removes the caches and the graph |
| `settings.sh` | The settings above, and which layers each rung loads |
| `compose.yaml` | The session container and Memgraph, on their own network, away from experiment 1's tripwire |
| `image/` | The Dockerfile, the floor, and the start-up scripts: `ladder-assemble` builds `/etc/claude-code`, `ladder-prepare` indexes for Serena and the graph, `cgr` and `ladder-cgr-mcp` start code-graph-rag outside the repository |

## Known limits

- **No firewall.** The container reaches the network directly, as experiment 1's open run does.
- **Serena's index cache.** Serena keeps its index between runs in the `cache` volume. That is tool start-up, outside the agent's tokens. Its memories are removed before every run. `./stop.sh --all` clears the cache.
- **CodeBoarding.** Its component names come from one model run. Every e6 run sees the same output.
- **code-graph-rag and `.env`.** code-graph-rag reads `.env` from the folder it starts in. The MCP wrapper and the `cgr` command start it outside the repository. If you add a script that calls it, do the same.
