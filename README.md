# Open-AI-GPT-OSS-20B-120B
Open AI GPT OSS 20B , 120B .gpt oss in Oblivion lab , report from Oblivion lab 
OBLIVION / TECHNICAL RESEARCH GPT-OSS 20B & 120B

INTELLIGENCE. ENGINEERING. SYSTEMS. • 1
OBLIVION / MODEL SYSTEMS REPORT
GPT-OSS 20B & 120B
Architecture, reasoning behaviour, evaluation and deployment engineering
A critical engineering assessment for the Oblivion team
ENGLISH EDITION | 26 October 2026 | Research / 01OBLIVION / TECHNICAL RESEARCH GPT-OSS 20B & 120B
INTELLIGENCE. ENGINEERING. SYSTEMS. • 2
01 / OBLIVION RESEARCH
Executive assessment
Two operating points - one engineering decision framework
gpt-oss is a family of open-weight reasoning models designed for developers who want to inspect, host, adapt and operate
model weights within infrastructure they control. Its significance is not captured by parameter count alone. The family
combines sparse Mixture-of-Experts computation, adjustable reasoning effort, Harmony-formatted tool interaction and
Apache 2.0 licensing, subject to the published usage policy.
The 20B and 120B variants should be treated as different operating points. The smaller model is easier to host and can be
highly competitive on bounded reasoning tasks. The larger model offers more total capacity and stronger results on several
knowledge, coding and tool-use evaluations, but demands substantially more memory and compute. Neither is the
universal winner.
Measure gpt-oss-20b gpt-oss-120b
Total parameters 20.91B 116.83B
Active parameters / token 3.61B 5.13B
Checkpoint size 12.8 GiB 60.8 GiB
Selected experts / token 4 4
 Open-weight does not mean fully reproducible open-source training: the public release does not disclose the complete
pretraining corpus and full training pipeline.
 Active parameters describe selected computation, not the amount of weight data that must be stored.
 gpt-oss is not served through ChatGPT or the OpenAI API; deployment responsibility sits with the operator or hosting
provider.
Engineering verdict: use 20B as the first candidate for constrained local workflows; consider 120B for difficult knowledge,
multi-step coding and tasks where Oblivion measurements prove a quality gain worth the infrastructure cost. Route by
task and evidence, not by brand or parameter count.
Scope update: OpenAI also lists gpt-oss-safeguard-20b and -120b as research-preview safety-reasoning models. They
are policy-classification derivatives, not replacements for the general-purpose 20B/120B models discussed here.OBLIVION / TECHNICAL RESEARCH GPT-OSS 20B & 120B
INTELLIGENCE. ENGINEERING. SYSTEMS. • 3
02 / OBLIVION RESEARCH
Architecture and sparse computation
Why active parameters are not the same as model size
Both models are autoregressive Transformer language models built around Mixture-of-Experts (MoE) blocks. For each
token, a learned router selects four experts from a larger pool. Only the selected experts participate in that token’s expert
computation, while shared layers and attention remain part of the overall forward pass.
This sparse routing improves the ratio between total model capacity and per-token expert computation. It does not shrink
the checkpoint to the active-parameter count: all expert weights must be available in memory, streamed or offloaded by the
runtime.
Figure 1. Conceptual top-4 MoE routing. The diagram is explanatory, not a reproduction of the internal block graph.
Architecture item 20B 120B
Transformer layers 24 36
Experts per MoE block 32 128
Experts selected per token 4 4
Query heads 64 64
Key/value heads 8 8
Attention head dimension 64 64
 Attention alternates local banded-window and dense patterns; the local window bandwidth is 128 tokens.
 Grouped-query attention uses 64 query heads and 8 key/value heads.
 RoPE and YaRN support a reported context length up to 131,072 tokens in dense layers.
 The reported stack includes RMSNorm, Pre-LN, SwiGLU and MXFP4 post-training quantization for MoE weights.
Interpretation: 3.61B and 5.13B are useful measures of per-token active computation. They are not reliable estimates of
RAM/VRAM requirements. For memory planning, start with checkpoint size, then add KV cache, runtime buffers,
batching and operating-system overhead.OBLIVION / TECHNICAL RESEARCH GPT-OSS 20B & 120B
INTELLIGENCE. ENGINEERING. SYSTEMS. • 4
03 / OBLIVION RESEARCH
Evaluation: what the published scores say
A comparative reading of high-reasoning results
The table below reproduces selected high-reasoning values from OpenAI’s Model Card. Tool-assisted and no-tool rows are
different evaluation settings. Scores from unrelated benchmarks are not directly comparable, and vendor-reported scores
are not independent replications by Oblivion.
Figure 2. Selected scores reproduced from OpenAI’s published Model Card. The AIME 2025 tool-assisted result is the notable row where
20B is higher.
Benchmark / setting 120B 20B Higher score
AIME 2025 · tools 97.9 98.7 20B
GPQA Diamond · no tools 80.1 71.5 120B
SWE-bench Verified 62.4 60.7 120B
τ-Bench Retail 67.8 54.8 120B
Aider Polyglot 44.4 34.2 120B
MMLU 90.0 85.3 120B
 120B has larger reported advantages on GPQA Diamond, τ-Bench Retail and Aider Polyglot.
 20B slightly exceeds 120B on the published AIME 2025 tool-assisted row; this is task-specific, not evidence of a general
reversal.
 Codeforces Elo is reported as 2,622 vs 2,516 with tools and 2,463 vs 2,230 without tools (120B vs 20B).
Do not compress this into a single leaderboard. Model choice should be based on the Oblivion task mix: coding
repository changes, local extraction, multilingual support, tool reliability and latency may rank the models differently.OBLIVION / TECHNICAL RESEARCH GPT-OSS 20B & 120B
INTELLIGENCE. ENGINEERING. SYSTEMS. • 5
04 / OBLIVION RESEARCH
Reasoning, Harmony and tool use
A model that can call tools is not yet a dependable agent
Reasoning effort is configurable as low, medium or high. Greater effort can improve results on difficult problems, but it
usually increases generated tokens, latency and compute. OpenAI reports that gpt-oss-20b averaged more than 20,000
chain-of-thought tokens per AIME problem in a high-reasoning setting. This is a warning about cost and latency, not a
target budget for production.
The models use Harmony, a serialized conversation format that carries message boundaries, roles, channels and controlflow markers. Tool calls are assistant messages directed to a tool; tool responses return under the tool’s name. An
integration must parse control flow correctly rather than treating the entire stream as plain text.
Figure 3. Qualitative relationship between reasoning budget and compute. This is an explanatory illustration, not a measured curve.
Control surface Engineering implication
Reasoning effort Set per task; cap tokens and time.
Harmony parser Validate roles, channels, recipients and hand-off markers.
Tool schemas Validate arguments and reject unknown functions.
State machine Track tool calls, retries, timeouts and completion.
 Keep user data, tool output and developer instructions in distinct trust domains.
 Treat retrieved documents and web pages as untrusted content, including any embedded instructions.
 Use bounded retries, explicit stop conditions and a maximum tool-call budget.
Never expose raw internal reasoning as a guaranteed explanation. OpenAI warns that raw CoT can contain hallucinated
or policy-inconsistent content. Present concise rationales, tool outputs, citations and test evidence instead.OBLIVION / TECHNICAL RESEARCH GPT-OSS 20B & 120B
INTELLIGENCE. ENGINEERING. SYSTEMS. • 6
05 / OBLIVION RESEARCH
Inference, memory and deployment
Checkpoint size is only the first line of the resource budget
OpenAI reports checkpoint sizes of 12.8 GiB for 20B and 60.8 GiB for 120B. These are weight-storage figures, not
complete runtime requirements. Actual deployment also depends on KV cache, context length, concurrent sequences,
allocator fragmentation, quantization format, offload policy and runtime implementation.
A 131K context ceiling does not mean that every prompt should use the full context. Long contexts increase KV-cache
pressure and can reduce throughput. A model that technically loads may still be too slow, unstable or expensive for a
product workflow.
Resource factor What to measure
Weights Resident VRAM/RAM; format; offload ratio.
KV cache Peak memory by context length and concurrency.
Latency Time to first token; p50/p95 completion time.
Throughput Input/output tokens per second at target batch size.
Reliability OOM rate, restarts, malformed tool calls.
Cost / energy GPU time and energy per accepted task.
 OpenAI positions 120B for a single 80 GB GPU; confirm exact runtime and context conditions before treating that as an
operational guarantee.
 For 20B, do not infer smooth execution from a nominal 16 GB target alone; check whether the capacity refers to VRAM,
unified memory or system RAM.
 Record GPU model, driver, runtime version, quantization, batch size, context, prompt length and generation budget for
every benchmark.
Deployment rule: benchmark the full product path, not just model loading. Measure successful task completion per unit
of latency, memory, energy and operator intervention.OBLIVION / TECHNICAL RESEARCH GPT-OSS 20B & 120B
INTELLIGENCE. ENGINEERING. SYSTEMS. • 7
06 / OBLIVION RESEARCH
Safety, reliability and openness boundaries
The weights do not enforce product policy by themselves
OpenAI’s Model Card notes that open-weight deployment shifts substantial responsibility to downstream developers. Finetuning and system composition can alter behaviour, and the publisher cannot centrally enforce the safeguards of every
derivative deployment. Apache 2.0 supports broad use and modification, but the published usage policy remains relevant.
The Model Card reports that the models generally underperform o4-mini on instruction-hierarchy evaluations and on
SimpleQA/PersonQA factuality tests. The report also cautions that raw internal reasoning may contain hallucinations or
policy-inconsistent language. These limits matter more when a model has tools or access to local files.
Risk Required Oblivion control
Prompt injection Treat retrieved content as data, never as policy; isolate tool
instructions.
Over-privileged tools Allowlist capabilities; read-only defaults; explicit approval for
writes.
Factual errors Require source citations, retrieval checks and claim
verification.
Unsafe code execution Sandbox, resource limits, no unrestricted shell or secrets.
Silent failures Structured logs, event IDs, retries with limits and alerting.
 OpenAI reports parity with o4-mini on some StrongREJECT jailbreak evaluations, but weaker resistance to user
attempts to override system instructions.
 The default 120B did not reach the report’s indicative “High” thresholds in tracked Preparedness categories; this is a
bounded evaluation result, not a universal safety guarantee.
 Open weights can be modified after release. A product must enforce its own permissions, monitoring, moderation and
audit trail.
Security position: safety is a system property. It depends on the model, prompts, runtime, tools, permissions, data
sources, human review and monitoring—not on the model checkpoint alone.OBLIVION / TECHNICAL RESEARCH GPT-OSS 20B & 120B
INTELLIGENCE. ENGINEERING. SYSTEMS. • 8
07 / OBLIVION RESEARCH
Oblivion reference architecture
Keep the model replaceable; keep policy outside the model
Oblivion should treat gpt-oss as an interchangeable reasoning backend. A separate orchestration layer should own
authentication, user permissions, model routing, context assembly, tool authorization, execution isolation, evidence
checking and audit events. This architecture remains valid if the underlying model changes.
Figure 4. Suggested separation of concerns for an Oblivion agent harness.
Layer Responsibility
Identity and policy User/session identity, tenant boundaries, role checks.
Model router Select 20B/120B using task class, risk, budget and measured
quality.
Agent harness Harmony adapter, state machine, token/time budgets, bounded
retries.
Tool gateway Allowlist, schema validation, sandbox, approval gate.
Verifier Tests, citations, diff review, output constraints.
Observability Immutable audit events, metrics, trace IDs and incident review.
 20B candidate role: extraction, summarization, first-pass code drafts, bounded local analysis and low-risk tasks with
validation.
 120B candidate role: difficult planning, complex debugging, deep code review and tasks where internal evaluation
demonstrates meaningful uplift.
 Require human approval for deletion, publishing, external messaging, credential changes, repository merges and other
state-changing actions.
The model may propose an operation; the tool gateway decides whether that operation is authorized. A model score is
never a permission grant.OBLIVION / TECHNICAL RESEARCH GPT-OSS 20B & 120B
INTELLIGENCE. ENGINEERING. SYSTEMS. • 9
08 / OBLIVION RESEARCH
Three-pass critical review
Claims challenged, evidence checked, design revised
Pass 1 - Challenge parameter-count assumptions. The first draft implicitly treated 20B as “small” and 120B as “better”. Both
are incomplete. The 20B checkpoint is 12.8 GiB before runtime overhead; 20B also exceeds 120B on one AIME 2025 toolassisted row while trailing on several knowledge and agentic tasks. Revision: describe task-specific operating points, not a
universal ranking.
Pass 2 - Challenge benchmark interpretation. Published scores do not establish identical prompt, harness, repetition or
hardware conditions for every task. High reasoning raises compute use. Revision: label every score with its setting, record
uncertainty where available and reproduce results on an Oblivion task suite before selecting a production default.
Pass 3 - Challenge safety-by-prompt. A prompt cannot reliably enforce policy when weights are modifiable and tool
execution is external. Revision: put permission checks, sandboxing, output validation and human approval in code outside
the model.
 Build a minimum 100-task evaluation set from anonymized Oblivion workflows.
 Test Persian, English, German, Japanese and Korean separately; do not extrapolate from English-only scores.
 Compare identical prompts, runtime, tools, budgets and hardware; report low/medium/high reasoning separately.
 Measure task pass rate, unit tests, hallucination, injection resistance, tool misuse, p50/p95 latency, peak memory and
energy per accepted task.
 Keep the deployment read-only by default; require approval for side effects and preserve complete audit logs.
Final position: 20B is a resource-conscious candidate with meaningful reasoning capability; 120B offers greater capacity
and stronger results on several demanding evaluations. Neither is a complete product. Oblivion’s value must come from
orchestration, verification, security and measured reliability.OBLIVION / TECHNICAL RESEARCH GPT-OSS 20B & 120B
INTELLIGENCE. ENGINEERING. SYSTEMS. • 10
09 / OBLIVION RESEARCH
Methodology and primary references
Evidence provenance, reproducibility and limits
This report was rewritten as an engineering assessment for the Oblivion team. Architecture and benchmark figures are
attributed to OpenAI’s published gpt-oss Model Card. Current hosting and Safeguard-family notes are based on OpenAI
Help Center guidance available on 26 October 2026. Harmony behaviour is cross-checked against OpenAI’s public format
documentation.
No benchmark in this report was independently run by Oblivion. Reported values can vary with harness version, prompt
design, reasoning effort, tool implementation, quantization, hardware, decoding settings and evaluation protocol. The
diagrams are explanatory editorial illustrations; they are not source diagrams from OpenAI.
 [1] OpenAI, gpt-oss-120b & gpt-oss-20b Model Card, 5 August 2025. https://deploymentsafety.openai.com/gpt-oss/a2
 [2] OpenAI, Introducing gpt-oss, 5 August 2025. https://openai.com/index/introducing-gpt-oss/
 [3] OpenAI Help Center, OpenAI open-weight models; current guidance accessed 26 October 2026.
https://help.openai.com/
 [4] OpenAI, Harmony format documentation and public repository. https://github.com/openai/harmony
 [5] OpenAI Cookbook, Harmony format. https://cookbook.openai.com/articles/openai-harmony
 [6] Hugging Face, official OpenAI gpt-oss model repositories. https://huggingface.co/openai
Interpretation boundary: this is not an OpenAI endorsement, a security certification or an independent benchmark report.
Licensing and usage-policy decisions should be checked against the current source text before product release.
