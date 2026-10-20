# Week 1: the experiments

The Week 1 experiments of the InfoQ Certified AI-Assisted Engineering Program. This is the same ladder we ran in class, so you can run it again on your own Claude token, change a layer, and see what changes.

Every run asks the same six questions about a seeded copy of Spring PetClinic ([he-petclinic](https://github.com/qcon-he-course/he-petclinic) at commit `b107a86`). Each experiment changes one thing around the agent and keeps the rest.

## The ladder

| Experiment | What it changes |
|---|---|
| [E0: the naked agent](experiments/e0-the-naked-agent/) | Nothing. Claude Code on the floor only |
| [E1: limit what it can see and do](experiments/e1-limit-what-it-can-do/) | The floor taken away: plain Claude Code with the repository's own configuration trusted, beside a tripwire |
| [E2: tell it how to behave](experiments/e2-tell-it-how-to-behave/) | An instruction file about behaviour |
| [E3: give it navigation](experiments/e3-give-it-navigation/) | Serena, a Java language server, over MCP |
| [E4: give it code knowledge](experiments/e4-give-it-code-knowledge/) | A code graph (code-graph-rag with Memgraph), over MCP |
| [E5: require evidence](experiments/e5-require-evidence/) | The evidence rule, and an audited `AGENTS.md` |
| [E6: build your mental model](experiments/e6-build-your-mental-model/) | Diagram skills, and CodeBoarding's component analysis |

Each rung keeps the layers of the rungs below it. E1 stands beside the ladder: it compares an open run with E0's contained one.

## Quick start

Do [SETUP.md](SETUP.md) first: Docker, your token, and the first image build. Then:

```bash
cd ~/course/he-experiments/week-1/experiments/ladder
read -rs CLAUDE_CODE_OAUTH_TOKEN && export CLAUDE_CODE_OAUTH_TOKEN
./run.sh e0                 # trust the folder, paste the questions, /exit when done
./show-run.sh e0            # what the run did: time, tokens, tool calls, files read
```

On macOS, `pbcopy < ../questions.txt` puts the questions on the clipboard before you paste. The [ladder README](experiments/ladder/README.md) explains every rung, unattended runs and the scoreboard.

## Our results

- [results/summary.md](results/summary.md): the medians of our stored runs, five per rung on Opus 5.5 and five on Haiku 4.5, with the outcome of each question. `summary.json` holds the same figures.
- [results/findings.md](results/findings.md): what the runs showed, by experiment, with the evidence for each finding.
- [results/runs/](results/runs/): each of our 60 stored runs, trimmed: the prompt, the final answer, the marks and `show-run.sh`'s summary. Also E6's two diagrams.

A model (Claude Fable 5.1) marked every stored run against [reference/answer-key.md](reference/answer-key.md). We then confirmed every mark by hand.

## Score your own run

Mark your answers against [reference/answer-key.md](reference/answer-key.md). An answer is right when it gives the outcome and the reason that decides it. Question 6 scores one point per step, out of 6.

Then write one line in the run's folder, for example `experiments/e0-the-naked-agent/runs/<run>/score.txt`:

```text
q1-5=4 q6=5
```

`./scoreboard.sh` reads it and adds your run to the quality columns. It counts only unattended runs, from `batch.sh`; an interactive run's folder has no `prompt.txt`, so the scoreboard leaves it out.

## Layout

| Path | What it is |
|---|---|
| [SETUP.md](SETUP.md) | Machines, Docker, your token, the quota each run uses |
| `experiments/ladder/` | The container that runs E0 and E2 to E6, and its scripts |
| `experiments/e0-...` to `e6-...` | One folder per experiment: `README.md`, `layer/` (what the rung adds), `runs/` (your runs, git-ignored) |
| `experiments/e1-limit-what-it-can-do/` | E1's open run, tripwire and `show-run.sh` |
| `experiments/questions.txt` | The six questions every run asks |
| `reference/questions.md` | The question sheet from class, with the confidence tables |
| `reference/answer-key.md` | The answers, with the evidence for each |
| `results/` | Our stored runs' summary and findings |

## License

MIT, in [LICENSE](../LICENSE). The tools the image installs, and the seeded PetClinic, keep their own licenses.
