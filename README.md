# Amaljith Kuttamath

### Now

- **ferrag** (not public yet). A local-first RAG engine in Rust. The idea is a library you embed in your app, the way you'd use SQLite, instead of another server to run.
- **[pg-evidence](https://github.com/amaljithkuttamath/pg-evidence)**. A Postgres extension, also Rust, that stores documents as versions so an agent's citations still point at the exact text after the source changes.
- **[ai-radar](https://github.com/amaljithkuttamath/ai-radar)**. A daily AI digest pulled from about 30 sources. A model writes it, a rubric grades it, and GitHub Actions keeps it running.

### Tools

- **[jira-ops](https://github.com/amaljithkuttamath/jira-ops)**: a Jira Cloud CLI meant to be driven by agents. JSON output, and writes are dry runs unless you say otherwise. `brew install amaljithkuttamath/tap/jira-ops`
- **[crux](https://github.com/amaljithkuttamath/crux)**: a terminal dashboard that shows where your Claude Code and Cursor tokens go.
- **[research-loom](https://github.com/amaljithkuttamath/research-loom)**: a Claude Code and Codex plugin that interviews you, narrows the question, then searches and collects sources.
- **[mcdxkit](https://github.com/amaljithkuttamath/mcdxkit)**: turns ENSOFT reports into Calcpad worksheets and Mathcad files. On PyPI.

### Interpretability

- **[jlens-replication](https://github.com/amaljithkuttamath/jlens-replication)**: I tried to replicate Anthropic's Jacobian lens paper on Qwen 2.5-3B and Gemma-4-E2B with free Kaggle GPUs. Most of the probing result didn't survive the shuffled-corpus control. The numbers are in the repo.
- **[mlx-interp](https://github.com/amaljithkuttamath/mlx-interp)**: train sparse autoencoders on Apple Silicon with MLX.
- **[sae-explorer](https://github.com/amaljithkuttamath/sae-explorer)**: found a Gemma 2 2B feature that fires on "and" in six languages.

For fun I'm building **[ares](https://github.com/amaljithkuttamath/ares)**, a Mars colony game in Godot.

### Writing

[amaljithkuttamath.github.io](https://amaljithkuttamath.github.io/), including [a single neuron for "and" in six languages](https://amaljithkuttamath.github.io/work/sae-explorer) and [how LLMs work in 200 lines of Python](https://amaljithkuttamath.github.io/work/understanding-llms).

[LinkedIn](https://www.linkedin.com/in/amaljithk/) · [Twitter](https://twitter.com/amaljithk) · [Substack](https://substack.com/@amaljithk) · kuttamath.amaljith@gmail.com
