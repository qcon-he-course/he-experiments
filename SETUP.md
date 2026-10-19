# Setup

About 30 minutes, most of it the first image build.

## Supported machines

- macOS, on Apple Silicon or Intel.
- Linux on x86-64.
- Windows through WSL 2 should work, with every step done in the Ubuntu terminal. We haven't tested it.

You need:

- **Docker** with Compose v2: Docker Desktop on macOS, Docker Desktop or Docker Engine on Linux. Check with `docker compose version`.
- **About 15 GB of free disk.** The images take about 11 GB: the ladder image 5.5 GB, Memgraph 2.8 GB and E1's image 2.2 GB.
- **At least 8 GB of memory for Docker.** In Docker Desktop, set it under Settings, Resources. This figure is inferred: from rung 3 a run starts Serena's Java language server, and from rung 4 Memgraph as well.
- **git, bash and python3** on your machine. The scripts that summarise a run use python3.

## Get the repository

```bash
mkdir -p ~/course
git clone https://github.com/qcon-he-course/he-experiments.git ~/course/he-experiments
```

## Your token

The runs use your own Claude plan. You need a plan that includes Claude Code: Pro, Max, Team or Enterprise. With a Claude Console account, use an API key instead (the end of this section).

1. On your machine, run `claude setup-token` and sign in. It prints a long-lived token.
2. In each terminal window you run experiments from, read it into the environment:

   ```bash
   read -rs CLAUDE_CODE_OAUTH_TOKEN && export CLAUDE_CODE_OAUTH_TOKEN
   ```

   Paste the token and press Enter. `read -s` doesn't echo what you paste, and the token stays out of your shell history.

Only Claude Code uses the token, inside the container, talking directly to Anthropic. Never put it in a file in this repository, in a chat, or in an issue. With a Claude Console API key instead, read it the same way into `LADDER_ANTHROPIC_API_KEY` and export that. E0 and E2 to E6 accept it; E1's `open-run.sh` needs the token.

## The quota each run uses

Every run counts against your plan. Our stored runs sent these tokens per run (medians from [results/summary.md](results/summary.md)):

| Model | E0 | E1 | E2 | E3 | E4 | E5 |
|---|---|---|---|---|---|---|
| Opus 5.5 | 188k | 219k | 271k | 422k | 320k | 220k |
| Haiku 4.5 | 1042k | 723k | 841k | 1180k | 934k | 382k |

Most of it is served from cache: 82% to 89% on Opus, 93% to 96% on Haiku. Haiku made more tool calls, a median of 21 to 43 per run against Opus's 12 to 30. An interactive run may send more than these (inferred): it can ask for permission and carry on where an unattended run is refused.

A batch of five rounds is 30 runs: every rung, five times. Start with one run, and check `/usage` before a batch.

## Open PetClinic only through run.sh

The seeded PetClinic carries rigged prompt-injection items and a fake token in its `.env`. They are harmless, and they are the point: you see what an agent does with them in a contained run. `run.sh` clones PetClinic fresh for every run, into `~/.cache/qcon-he/ladder/`, and removes the clone afterwards.

Don't open the seeded PetClinic in your own agent on your own machine. Treat its files as data.

## Build the images

```bash
cd ~/course/he-experiments/experiments/ladder
docker compose build session && docker compose pull memgraph
```

Expect 5 to 10 minutes the first time. `run.sh` rebuilds before each run, from cache, in a second or two. E1's image builds the first time you run `./open-run.sh` in `experiments/e1-limit-what-it-can-do/`.

Then run E0 once, as in the [quick start](README.md#quick-start).

## CodeBoarding (optional)

E6 uses CodeBoarding's component analysis of PetClinic. Our output is already in `experiments/e6-build-your-mental-model/codeboarding/`, so you don't need to run it.

To make your own, CodeBoarding calls a model for every component, so it needs a provider API key: exactly one of `ANTHROPIC_API_KEY` (a Claude Console key), `OPENAI_API_KEY`, `OPENROUTER_API_KEY` or `GOOGLE_API_KEY`. Never use your subscription token for it; the script refuses one. Then run `./codeboarding.sh` in `experiments/ladder/`. It replaces our output in the E6 folder.

## If something fails

| What you see | Likely cause | What to do |
|---|---|---|
| `run: run 'claude setup-token' on this machine` | No token in this window | Export it again, as above |
| `Cannot connect to the Docker daemon` | Docker isn't running | Start Docker Desktop, or `sudo systemctl start docker` |
| The build stops with "no space left on device" | Docker's disk is full | `docker system prune`, or give Docker more disk |
| E3 or E4 hangs before Claude Code starts, or `prepare.log` has no "Serena index ready" | Too little memory for Docker | Give Docker 8 GB or more and run again |
| `git clone` of he-petclinic fails | No network, or a proxy | Check that `git ls-remote https://github.com/qcon-he-course/he-petclinic.git` works |
| A run hits a usage limit | Your plan's quota | Wait for it to reset; finished runs are kept |
