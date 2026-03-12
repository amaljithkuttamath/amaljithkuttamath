# Amaljith Kuttamath

<img src="https://komarev.com/ghpvc/?username=amaljithkuttamath&label=Profile%20Views&color=0e75b6&style=flat" align='right' alt="amaljithkuttamath" />

**AI Engineer** building tools to understand what language models are doing internally, and catching them when they fail.

Working on mechanistic interpretability, LLM evaluation, and trust/safety tooling. Most of my projects are small, sharp, and designed to answer one question well.

---

### Current focus

**[Trust Bench](https://github.com/amaljithkuttamath/trust-bench)** - Interpretability toolkit that probes SAE features, activations, and circuits in Llama 3.1 8B. Three probes (feature survey, hallucination, cross-lingual), statistical analysis, publication-quality viz, and a CLI.

### Interpretability research

| Project | What it does |
|---------|-------------|
| [sae-explorer](https://github.com/amaljithkuttamath/sae-explorer) | Found a single SAE feature (#10543) that fires on "and" across six languages in Gemma 2 2B. Zero false positives. |
| [superposition-viz](https://github.com/amaljithkuttamath/superposition-viz) | Reproduces Anthropic's Toy Models of Superposition. Found phase transition at 0.7 sparsity. |
| [activation-atlas](https://github.com/amaljithkuttamath/activation-atlas) | Layer-by-layer UMAP projections showing how neural networks organize learned representations. |
| [scaling-laws](https://github.com/amaljithkuttamath/scaling-laws) | Train transformers from 100K to 10M params, fit power laws, plot the curves. Do they hold at toy scale? |
| [loss-landscape](https://github.com/amaljithkuttamath/loss-landscape) | 3D surface plots of loss landscapes around trained weights. Sharpness comparison across training configs. |

### LLM evaluation

| Project | What it does |
|---------|-------------|
| [calibration-probe](https://github.com/amaljithkuttamath/calibration-probe) | Measure how well LLMs know what they know. Reliability diagrams and ECE across prompting strategies. |
| [attention-bench](https://github.com/amaljithkuttamath/attention-bench) | Benchmark MHA vs GQA vs MQA vs Sliding Window. Train small transformers, compare perplexity and throughput. |

### Rust CLI tools

| Project | What it does |
|---------|-------------|
| [crux](https://github.com/amaljithkuttamath/crux) | Terminal dashboard for AI coding tool token usage. |
| [tokenizer-arena](https://github.com/amaljithkuttamath/tokenizer-arena) | Compare how different LLM tokenizers handle the same text. Color-coded token boundaries. |
| [gguf-inspect](https://github.com/amaljithkuttamath/gguf-inspect) | Inspect GGUF model files from the terminal. Architecture, quantization, tensors, memory estimates. |

### Writing

Recent posts on [amaljithkuttamath.github.io/work](https://amaljithkuttamath.github.io/work):

- [A Single Neuron for 'And' in Six Languages](https://amaljithkuttamath.github.io/work/sae-explorer) - Cross-lingual SAE features in Gemma 2 2B
- [Why Trust Bench](https://amaljithkuttamath.github.io/work/why-trust-bench) - The case for probing LLM internals
- [How Large Language Models Actually Work](https://amaljithkuttamath.github.io/work/understanding-llms) - From raw text to trained model, in 200 lines of Python

### Tech

<p>
<a href="#"><img alt="Python" src="https://img.shields.io/badge/Python-%2314354C.svg?logo=python&logoColor=white"></a>
<a href="#"><img alt="Rust" src="https://img.shields.io/badge/Rust-%23000000.svg?logo=rust&logoColor=white"></a>
<a href="#"><img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-%23EE4C2C.svg?logo=PyTorch&logoColor=white"></a>
<a href="#"><img alt="Hugging Face" src="https://img.shields.io/badge/Hugging%20Face-FFB200?logo=huggingface&logoColor=white"></a>
<a href="#"><img alt="TransformerLens" src="https://img.shields.io/badge/TransformerLens-4B0082?logoColor=white"></a>
<a href="#"><img alt="SAELens" src="https://img.shields.io/badge/SAELens-2E8B57?logoColor=white"></a>
</p>

### Connect

<p>
<a href="https://www.linkedin.com/in/amaljithk/" target="_blank"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white" alt="LinkedIn"/></a>
<a href="https://twitter.com/amaljithk" target="_blank"><img src="https://img.shields.io/badge/Twitter-1DA1F2?style=flat&logo=twitter&logoColor=white" alt="Twitter"/></a>
<a href="https://substack.com/@amaljithk" target="_blank"><img src="https://img.shields.io/badge/Substack-FF6719?style=flat&logo=substack&logoColor=white" alt="Substack"/></a>
<a href="mailto:kuttamath.amaljith@gmail.com" target="_blank"><img src="https://img.shields.io/badge/Email-D14836?style=flat&logo=gmail&logoColor=white" alt="Email"/></a>
<a href="https://amaljithkuttamath.github.io/" target="_blank"><img src="https://img.shields.io/badge/Website-000000?style=flat&logo=github&logoColor=white" alt="Website"/></a>
</p>
