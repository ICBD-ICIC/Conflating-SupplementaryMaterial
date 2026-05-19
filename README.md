# Conflating Cognitive Tasks in LLMs: Preliminary Study on Misinformation - Supplementary Material

In this repository you will find all the raw logs from the 126 experiments runs, the calculated metrics and corresponding plots, together with the prompt used for the LLM-only agent and 

```
dataset/                # Network analysis of PHEME
plots/                  # Metrics plots  
raw/                    # Logs from the experiments runs
├── 600/                    # NeSy agent model
└── 600_llm/                # LLM-only agent model
results/                # Computed metrics and p_values
prompt.md               # LLM-only agent prompt
probabilities.md        # Parameters to NL translations
```

## PHEME Thread Analysis

Inside the `dataset` directory, each thread has two files: an `original` version and a `joined` version.

The **original** file shows the network built from agents within that thread only. The **joined** file shows the network after merging agents across all three threads. Because threads share users, the joined network expands the susceptible pool - agents unexposed in one thread may be reachable through connections formed in another.

Each thread includes two graphs:

- **Full network graph** - shows all agents and follow relationships in the thread. Nodes are colored by role: the initiator (source of the rumour), reactors (agents who replied or retweeted), and follow-only agents (present in the network but who never posted). Floating agents with no follow edges are also identified.

- **Susceptible subgraph** - isolates the subset of agents who have a directed follow path back to the initiator, meaning they are structurally positioned to receive and potentially propagate the misinformation. This subgraph distinguishes between susceptible reactors (who also actively engaged) and susceptible follow-only agents (who are connected but passive).

## LLM-only agent setup

`probabilities.md` contains the complete mapping from transition probabilities to natural language descriptors used to configure LLM agent personalities. 

`prompt.md` contains the core system prompt that drives LLM-only agent behaviour during the simulation. We use a schema to force Gemini respond with the correct JSON format. 

## Raw Logs 

Inside `raw` directory, each model folder contains subfolders per thread configuration, where the thread ID alone serves as the baseline configuration.

Inside each thread configuration folder you can find:
- **3 run subfolders** named by timestamp (e.g., `1778753440948/`) - these contain the agent log files for that run
- **3 message files** named `messages_<timestamp>.jsonl` - these contain the messages the agents produced

> **Note:** The run folder timestamps and message file timestamps do not match directly. They are paired in chronological order - the first folder pairs with the first message file by proximity in time, and so on.

Each run folder contains one `.jsonl` log file per agent. Log format differs by agent type:

### NeSy Agents (`600/`)

Each line is a JSON object. Key fields include:

| Field | Description |
|---|---|
| `state` | Agent's current belief state (e.g., `"neutral"`) |
| `info` | Human-readable description of the processing step |
| `timestamp` | Unix timestamp in milliseconds |
| `cycle` | Processing cycle number |
| `idle_cycles` | Number of consecutive idle cycles |
| `pnov` | Message's novelty |
| `prpl` | Message's engagement over time |
| `pnw` | Message's cumulative influence |
| `topics` | List of topics identified in the message being processed |
| `u` | Random value |
| `pinf` | Probability of becoming Infected on first contact |
| `pmd` |  Probability of becoming Vaccinated on first contact when not Infected |
| `popi` | Probability of reinforcing one's own opinion when in agreement or after resisting adoption |
| `pad` | Probability of adopting the opposing state when in disagreement with a message  |
| `conversation_id` | ID of the conversation the message belongs to |

### LLM-only Agents (`600_llm/`)

Each line is a JSON object. Key fields include:

| Field | Description |
|---|---|
| `state` | Agent's current belief/opinion state (e.g., `"neutral"`, `"vaccinated"`) |
| `info` | Human-readable description of the processing step |
| `timestamp` | Unix timestamp in milliseconds |
| `cycle` | Processing cycle number |
| `message_id` | ID of the message being processed |
| `llm_result` | Array of objects with the LLM's output (see below) |
| `messages_processed` | Total messages processed in the last cycle |
| `actions_taken` | Total actions (replies, etc.) taken in the last cycle |

**`llm_result` structure:**

| Field | Description |
|---|---|
| `new_state` | The agent's updated belief state after processing the message |
| `reply_content` | Generated text of the reply to post (empty string if no reply) |
| `topics` | List of topics the LLM identified as relevant for the generated reply |


**Message Files (`messages_<timestamp>.jsonl`)**

Each file contains the social media messages that were created during the simulation for a given run. These are the messages the agents read and potentially responded to.

## Simulation Results

Inside `results` directory you can find the following files:

**`results.csv`** - Summary table with one row per thread configuration per variant (rule-based `600` or LLM-based `600_llm`). Each metric is reported as a mean ± standard deviation across all runs of that configuration.

**`results_per_run.csv`** - Granular table with one row per individual run. Contains raw (non-aggregated) values for every metric, useful for inspecting run-to-run variance.

**`results_pooled_delta.csv`** - Output of the pooled delta analysis. For each combination of variant, agent configuration, and metric, it reports the mean delta relative to the baseline (i.e. how much the metric shifted when cautious or credulous agents were introduced) along with a Wilcoxon signed-rank p-value and significance label. Deltas are computed per thread and then pooled across all three threads (n=9 per cell).

The key metrics tracked across all outputs are:

- **% infected / % vaccinated** - share of agents (or susceptible agents specifically) ending in each final state
- **Vaccination effectiveness** - share of state-changed susceptibles who were vaccinated rather than infected
- **State transitions** - percentage breakdown of how agents moved between states (neutral -> infected, neutral -> vaccinated, infected -> vaccinated, vaccinated -> infected)
- **Cycle and message stats** - total simulation cycles, average messages per cycle, and the share of messages posted by infected vs. vaccinated agents

## Plots

Plots are located inside `plots` and organized into three subdirectories. Each plot is produced per thread as well as for an aggregate "all threads" view (mean across threads). Bars show mean ± std across runs, with individual run values overlaid as jitter points when per-run data is available. Significance stars are drawn at the outer end of each bar using pooled p-values from `results_pooled_delta.csv`.

Inside `transitions` there is one plot per transition metric per thread, split into cautious and credulous panels side by side. The y-axis shows the **delta relative to baseline** (in percentage points). The four transitions covered are neutral -> infected, neutral -> vaccinated, infected -> vaccinated, vaccinated -> infected, each with a noted expected direction for cautious vs. credulous configurations.

In `outcomes`, there are two plots per outcome metric per thread:

- **Absolute** - all seven configurations (baseline + 6 variants) shown side by side, useful for comparing raw infection and vaccination rates across conditions.
- **Delta** - cautious and credulous groups in separate panels, showing the shift from baseline. Covers % infected of susceptibles, % vaccinated of susceptibles, and vaccination effectiveness.

In `llm_behaviour` there are two plot types focused on simulation dynamics:

- **Messages per cycle** - absolute bar chart across all configurations per thread, comparing how active agents are across conditions and variants.
- **Total messages vs. max cycles scatter** - a single plot pooling all threads and runs, where each point is one run colored by variant and shaped by thread. Useful for spotting whether differences in message volume are driven by longer simulations or higher per-cycle activity.

