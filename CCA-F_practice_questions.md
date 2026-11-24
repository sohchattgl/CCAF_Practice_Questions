# Claude Certified Architect – Foundations: Practice Questions

> Written from my recollection of the exam guide: scenario-based multiple choice, five domains. **Verify the domain list and weights against the official exam guide.** These are practice questions, not real exam items.
> 
> | Domain | Approx. weight |
> |---|---|
> | 1. Agentic architecture and orchestration | 27% |
> | 2. Tool design and MCP integration | 18% |
> | 3. Claude Code configuration and workflows | 20% |
> | 4. Prompt engineering and structured output | 20% |
> | 5. Context management and reliability | 15% |
> 
> Try each question before reading the answer and explanations directly beneath it. A compact answer key is at the end.

---

## Domain 1: Agentic Architecture and Orchestration

**Q1.** In a manual agentic loop using the Messages API, what should determine whether the loop continues?

- A. Parsing the assistant's text for phrases like "I'm done"
- B. A fixed maximum iteration count
- C. The `stop_reason`: continue on `tool_use`, terminate on `end_turn`
- D. Whether the assistant's response contains any text block

**Answer: C**

- **A ✘** — Incorrect. Natural-language text is non-deterministic: the model may say "done" while still wanting a tool, or never use the phrase at all. Parsing prose for completion is a recognised anti-pattern.
- **B ✘** — Incorrect as the primary control. An iteration cap is a useful safety net against runaway loops, but it does not tell you whether the model finished. Used alone it either cuts off valid work or lets finished work keep spinning.
- **C ✔** — Correct. `stop_reason` is the API's explicit control signal. `tool_use` means Claude wants tool results, so execute them, append them to the history and call again. `end_turn` means Claude has finished, so exit the loop.
- **D ✘** — Incorrect. A response can contain text AND a tool_use block, so text presence says nothing about completion. Text blocks often accompany tool calls ("Let me check that...").

**Q2.** A customer-support agent must NEVER issue a refund above $500 without human approval. Prompt instructions say this, but about 3% of runs still violate it. What is the best fix?

- A. Add stronger wording and capitalised warnings to the system prompt
- B. Add a hook or tool-level check that blocks the refund call and routes to human approval
- C. Add few-shot examples showing refusals
- D. Lower the temperature

**Answer: B**

- **A ✘** — Incorrect. Prompts are probabilistic. Stronger wording may reduce the 3% violation rate but can never make it zero, and a financial limit needs a guarantee.
- **B ✔** — Correct. A hook or tool-level check runs in code, so it is deterministic. It intercepts the refund call, rejects amounts above $500 and routes to human approval. Hard business rules belong in enforcement code, not in instructions.
- **C ✘** — Incorrect. Few-shot examples improve the likelihood of correct behaviour but are still probabilistic. They suit ambiguous judgement calls, not zero-tolerance rules.
- **D ✘** — Incorrect. Temperature changes sampling randomness, not rule adherence. A low-temperature model can still consistently misread or ignore the policy.

**Q3.** In a multi-agent research system with a coordinator and subagents, a subagent keeps producing analysis that ignores earlier findings. What is the most likely cause?

- A. Subagents automatically share the coordinator's conversation history, which confuses them
- B. Subagents have isolated context, and the coordinator didn't pass the needed findings in their prompts
- C. The subagent's model is too small
- D. The subagent has too few tools

**Answer: B**

- **A ✘** — Incorrect. This is the opposite of how subagents work. They do NOT automatically inherit the coordinator's history. That isolation is the whole reason the problem occurs.
- **B ✔** — Correct. Subagents run with isolated context. Anything they need (earlier findings, constraints, sources) must be passed explicitly in their prompt by the coordinator.
- **C ✘** — Incorrect. Model size is a speculative cause. Ignoring earlier findings is explained directly by missing context, and nothing in the scenario points to capability limits.
- **D ✘** — Incorrect. Tool count affects what actions are possible, not whether the agent knows prior findings. Missing earlier analysis is an information problem, not a tooling one.

**Q4.** A coordinator needs to research three independent topics. How do you get the lowest latency?

- A. Spawn subagents one per turn so each result informs the next
- B. Emit multiple subagent (Task) tool calls in a single coordinator response so they run in parallel
- C. Run one subagent and ask it to cover all three topics
- D. Have subagents message each other directly

**Answer: B**

- **A ✘** — Incorrect. Sequential spawning makes sense only when each result depends on the previous one. The topics are independent, so this needlessly adds the latencies together.
- **B ✔** — Correct. Emitting several Task (subagent) calls in a single coordinator response lets them execute in parallel. Total time is roughly that of the slowest subagent.
- **C ✘** — Incorrect. One agent covering three topics serialises the work and bloats its context, which usually lowers quality as well as speed.
- **D ✘** — Incorrect. Direct peer-to-peer messaging breaks hub-and-spoke orchestration, removes the coordinator's visibility and error handling, and doesn't reduce latency for independent tasks.

**Q5.** A research coordinator's final report keeps missing a whole sub-area of the topic, even though every subagent did its job well. Where is the likely fault?

- A. The synthesis subagent
- B. The coordinator's task decomposition, which was too narrow
- C. The search tool's rate limiting
- D. Prompt caching

**Answer: B**

- **A ✘** — Incorrect. The synthesiser can only combine what it receives. If a sub-area was never researched, it has nothing to synthesise.
- **B ✔** — Correct. Every subagent performed well, yet a sub-area is absent. That points upstream to how the coordinator split the topic. A too-narrow decomposition leaves gaps no subagent was ever assigned to fill.
- **C ✘** — Incorrect. Rate limiting would cause errors or partial results in individual subagents, which the scenario says did their jobs well, and would not produce a systematically missing sub-area.
- **D ✘** — Incorrect. Prompt caching affects cost and latency, not which topics are covered.

**Q6.** Which statement about hub-and-spoke orchestration is correct?

- A. Subagents should communicate peer-to-peer to reduce coordinator load
- B. All inter-subagent communication flows through the coordinator, which gives observability and consistent error handling
- C. The coordinator should never see subagent results
- D. Each subagent should spawn its own subagents by default

**Answer: B**

- **A ✘** — Incorrect. Peer-to-peer communication is the alternative to hub-and-spoke. It sacrifices observability, makes error handling inconsistent and makes the system harder to reason about.
- **B ✔** — Correct. Routing everything through the coordinator gives one place to observe the system, handle errors consistently and control information flow. That is the point of hub-and-spoke.
- **C ✘** — Incorrect. The coordinator must see subagent results, because it synthesises them and decides what to do next.
- **D ✘** — Incorrect. Recursive spawning by default creates uncontrolled fan-out, cost and complexity. Delegation should be deliberate and coordinator-driven.

**Q7.** You resumed a long investigation session a day later. Several tool results in the history are now stale. What is the best approach?

- A. Resume as-is and trust the history
- B. Resume and tell the agent which files or results changed so it re-verifies them. If the staleness is extensive, start fresh with a structured summary.
- C. Always fork the session
- D. Delete all tool results from the history

**Answer: B**

- **A ✘** — Incorrect. After a day, file contents, API data and other tool results may have changed. The agent would reason over stale facts with full confidence.
- **B ✔** — Correct. Telling the agent what changed lets it re-verify just those items and keep the rest of the useful context. If most of the history is stale, a fresh session seeded with a structured summary is cleaner and more reliable.
- **C ✘** — Incorrect. Forking creates independent branches from a baseline for comparing approaches. It does nothing about stale data, since the fork inherits the same stale results.
- **D ✘** — Incorrect. Deleting all tool results strips the evidence behind the agent's conclusions and discards still-valid information. Targeted re-verification is better.

**Q8.** You want to explore two alternative refactoring approaches from the same analysed baseline without the two contaminating each other. Which feature fits?

- A. `--resume` twice on the same session
- B. Session forking (`fork_session`), which creates independent branches from a shared baseline
- C. Plan mode
- D. Compaction

**Answer: B**

- **A ✘** — Incorrect. Resuming the same session twice continues one shared history. The two explorations would contaminate each other instead of staying independent.
- **B ✔** — Correct. `fork_session` creates independent branches from a shared baseline. Each approach starts with the same analysis but evolves separately.
- **C ✘** — Incorrect. Plan mode controls whether Claude plans before executing. It does not create separate branches of a session.
- **D ✘** — Incorrect. Compaction summarises history to save context. It doesn't produce parallel, isolated branches.

**Q9.** Which task is best suited to a fixed prompt chain (a sequential pipeline) rather than dynamic adaptive decomposition?

- A. Open-ended "improve this legacy codebase"
- B. Reviewing a PR in fixed stages: per-file local analysis, then a cross-file integration pass
- C. Investigating an unknown production incident
- D. Exploring a new domain with unknown structure

**Answer: B**

- **A ✘** — Incorrect. An open-ended improvement task has unknown structure, so the steps can't be fixed in advance. It needs dynamic decomposition that adapts to what is discovered.
- **B ✔** — Correct. A PR review with predictable stages (local per-file analysis, then a cross-file integration pass) fits a fixed pipeline. Each stage has a known purpose and input.
- **C ✘** — Incorrect. Incident investigation depends on what each clue reveals, so the path can't be pre-planned. Adaptive decomposition fits.
- **D ✘** — Incorrect. Exploring an unknown domain is discovery work. A fixed chain would impose a structure you don't yet know.

---

## Domain 2: Tool Design and MCP Integration

**Q10.** An agent often calls `get_customer` when it should call `lookup_order`. Both descriptions say "Retrieves information." What is the most effective first fix?

- A. Add a routing classifier in front of the agent
- B. Rewrite the descriptions to state purpose, inputs, example queries and when to use each versus the other
- C. Merge both into one tool
- D. Add few-shot examples only

**Answer: B**

- **A ✘** — Incorrect. A classifier adds components and latency to fix something that lives in the tool descriptions. It's overengineering as a first step.
- **B ✔** — Correct. The model chooses tools mainly from their descriptions. Two near-identical descriptions ("Retrieves information") make them indistinguishable. Stating purpose, inputs, example queries and when to use each versus the other fixes the root cause cheaply.
- **C ✘** — Incorrect. Merging forces one tool to handle two different jobs, hides distinct semantics and complicates parameters. It treats the symptom, not the unclear descriptions.
- **D ✘** — Incorrect. Few-shot examples can help, but they add token overhead and don't address the root cause of unclear descriptions. Descriptions are the first thing to fix.

**Q11.** You want Claude to always return data through a specific extraction tool and never reply in plain text. Which `tool_choice` setting?

- A. `auto`
- B. `any`
- C. Forced tool selection, `{"type":"tool","name":"extract_invoice"}`
- D. `none`

**Answer: C**

- **A ✘** — Incorrect. `auto` lets Claude decide whether to call a tool at all, so it may reply in plain text.
- **B ✘** — Incorrect. `any` forces Claude to call some tool, but not which one. With multiple tools available it could pick a different one.
- **C ✔** — Correct. Forcing a named tool (`{"type":"tool","name":"extract_invoice"}`) guarantees that exact tool is called, which gives guaranteed structured output.
- **D ✘** — Incorrect. `none` prevents tool use entirely, the opposite of the goal.

**Q12.** A tool fails because of a downstream timeout. What should the tool return so the agent can recover well?

- A. An empty result
- B. A generic "Operation failed"
- C. A structured error (e.g., `isError: true`) with error category, retryability and a human-readable message
- D. Throw an exception that crashes the loop

**Answer: C**

- **A ✘** — Incorrect. An empty result looks like a successful "nothing found". The agent would wrongly conclude no data exists rather than that the check failed.
- **B ✘** — Incorrect. A generic message gives the agent nothing to act on. It can't tell whether to retry, try another route or escalate.
- **C ✔** — Correct. A structured error (with `isError: true`, a category such as timeout, a retryable flag and a readable message) lets the agent recover intelligently, for example retrying a transient timeout.
- **D ✘** — Incorrect. Crashing the loop destroys the chance to recover. Errors should be returned to the agent as data.

**Q13.** What is the key difference between a tool returning an empty result and a tool returning an access-failure error?

- A. None. Both mean "no data".
- B. An empty result is a valid answer ("nothing matched"). An access failure means the answer is unknown, and treating it as empty causes silent wrong conclusions.
- C. Empty results should always be retried
- D. Errors should be hidden from the agent

**Answer: B**

- **A ✘** — Incorrect. They mean very different things. Conflating them is the exact bug the question probes.
- **B ✔** — Correct. An empty result is a valid, successful answer ("nothing matched"). An access failure means the answer is unknown. Treating the latter as empty leads to silently wrong conclusions.
- **C ✘** — Incorrect. A legitimately empty result is a real answer. Blind retries waste calls and don't change it.
- **D ✘** — Incorrect. Hiding errors prevents recovery and produces silent failures. The agent must see them.

**Q14.** A single agent has 18 tools and its tool selection reliability has dropped. What is the recommended remedy?

- A. Add more tools to cover edge cases
- B. Give each agent a focused set (roughly 4–5) of tools relevant to its role, and distribute the rest across specialised subagents
- C. Increase `max_tokens`
- D. Remove all tool descriptions

**Answer: B**

- **A ✘** — Incorrect. More tools worsens the problem. Selection reliability falls as the option space grows.
- **B ✔** — Correct. Scoping each agent to a focused set (about 4–5) relevant to its role improves selection accuracy. Remaining capabilities go to specialised subagents.
- **C ✘** — Incorrect. `max_tokens` limits output length and has no effect on how the model chooses among tools.
- **D ✘** — Incorrect. Descriptions are the main selection signal. Removing them makes selection worse.

**Q15.** Your team wants a shared MCP server config in the repo, with each developer supplying their own API token. Which setup is right?

- A. Hard-code tokens in `.mcp.json`
- B. Project-scoped `.mcp.json` committed to the repo, using environment-variable expansion (e.g., `${GITHUB_TOKEN}`) for secrets
- C. Put everything in each developer's `~/.claude.json`
- D. Store tokens in CLAUDE.md

**Answer: B**

- **A ✘** — Incorrect. Hard-coded tokens in a committed file leak secrets into version control.
- **B ✔** — Correct. A project-scoped `.mcp.json` is committed so everyone gets the same server definitions. `${GITHUB_TOKEN}`-style environment expansion lets each developer supply their own secret locally.
- **C ✘** — Incorrect. `~/.claude.json` is user-level and not shared, so teammates would each need to configure the server manually, and the team config wouldn't be consistent.
- **D ✘** — Incorrect. CLAUDE.md holds instructions, not server config, and putting tokens there would expose secrets and wouldn't configure anything.

**Q16.** When would you expose data as an MCP *resource* rather than a tool?

- A. When Claude must perform a side-effecting action
- B. When you want to give Claude a browsable catalogue of content (e.g., an issue list or documentation hierarchy) for context without exploratory tool calls
- C. When you need authentication
- D. Never, because resources are deprecated

**Answer: B**

- **A ✘** — Incorrect. Side-effecting actions are what tools are for. Resources expose content to read.
- **B ✔** — Correct. Resources expose browsable catalogues (issue lists, documentation hierarchies, schemas) so Claude gets context up front instead of making exploratory tool calls to discover what exists.
- **C ✘** — Incorrect. Authentication is a transport and server concern and doesn't determine whether to use a resource or a tool.
- **D ✘** — Incorrect. Resources are a standard part of MCP and aren't deprecated.

**Q17.** In Claude Code, you need to find every file that calls `processPayment`. Which built-in tool is best?

- A. Glob
- B. Grep
- C. Read
- D. Edit

**Answer: B**

- **A ✘** — Incorrect. Glob matches file names and paths, not their contents. It can't find who calls a function.
- **B ✔** — Correct. Grep searches file contents, so it finds every occurrence of `processPayment`.
- **C ✘** — Incorrect. Read opens a known file. It's useful after you've located candidates, not for discovery across the codebase.
- **D ✘** — Incorrect. Edit modifies files and doesn't search.

**Q18.** You need to find all files matching `**/*.test.tsx`. Which built-in tool is best?

- A. Grep
- B. Glob
- C. Bash `cat`
- D. Edit

**Answer: B**

- **A ✘** — Incorrect. Grep searches contents. Using it to match file names is clumsy and unreliable.
- **B ✔** — Correct. Glob matches path patterns such as `**/*.test.tsx`.
- **C ✘** — Incorrect. `cat` prints a file's contents and doesn't discover files by pattern. A dedicated tool is also preferable to shelling out.
- **D ✘** — Incorrect. Edit changes file contents and doesn't find files.

---

## Domain 3: Claude Code Configuration and Workflows

**Q19.** Where do you put coding conventions that apply only to you, across all projects, and should not be committed?

- A. `.claude/CLAUDE.md` in the repo
- B. `~/.claude/CLAUDE.md`
- C. A subdirectory CLAUDE.md
- D. `.mcp.json`

**Answer: B**

- **A ✘** — Incorrect. `.claude/CLAUDE.md` inside the repo is project-level and is meant to be committed and shared. It would impose your personal preferences on teammates.
- **B ✔** — Correct. `~/.claude/CLAUDE.md` is user-level. It applies across all your projects and lives outside any repo, so it is never committed.
- **C ✘** — Incorrect. A subdirectory CLAUDE.md applies only to that part of one project and is normally version-controlled.
- **D ✘** — Incorrect. `.mcp.json` configures MCP servers, not coding conventions.

**Q20.** A new teammate's Claude Code isn't following the team's conventions. The conventions are in a CLAUDE.md in the *original developer's home directory*. What is the root cause?

- A. The model is outdated
- B. The instructions are user-level, not project-level. They should be in the project's CLAUDE.md or `.claude/` and committed.
- C. Teammates need a different plan
- D. `--resume` was not used

**Answer: B**

- **A ✘** — Incorrect. Model version doesn't explain why conventions from one specific file aren't applied. The scenario points to where the instructions live.
- **B ✔** — Correct. A CLAUDE.md in one person's home directory is user-level and never reaches teammates. Moving the conventions into the project's CLAUDE.md or `.claude/` and committing them shares them with everyone.
- **C ✘** — Incorrect. Plans or licences don't control which instruction files are loaded.
- **D ✘** — Incorrect. `--resume` continues a prior session. It has no bearing on which configuration files exist on a teammate's machine.

**Q21.** Test-file conventions apply to `**/*.test.ts` files scattered through many directories. What is the best mechanism?

- A. A CLAUDE.md in every directory
- B. A `.claude/rules/` file with YAML frontmatter `paths: ["**/*.test.ts"]` so it loads only when matching files are edited
- C. Put all conventions into the root CLAUDE.md
- D. A slash command

**Answer: B**

- **A ✘** — Incorrect. Duplicating CLAUDE.md across many directories is hard to maintain and still wouldn't cleanly target a file pattern that cuts across directories.
- **B ✔** — Correct. A `.claude/rules/` file with `paths: ["**/*.test.ts"]` frontmatter loads only when matching files are being worked on. It handles scattered files in one place and saves context otherwise.
- **C ✘** — Incorrect. Putting everything in the root CLAUDE.md loads test conventions into every task, wasting context and diluting focus.
- **D ✘** — Incorrect. Slash commands are invoked explicitly. They aren't applied automatically based on which files are edited.

**Q22.** You want a reusable `/review` command shared with the whole team via version control. Where does it go?

- A. `~/.claude/commands/`
- B. `.claude/commands/` in the project repo
- C. `.mcp.json`
- D. In your system prompt

**Answer: B**

- **A ✘** — Incorrect. `~/.claude/commands/` is personal and not version-controlled, so teammates wouldn't get it.
- **B ✔** — Correct. Commands in the repo's `.claude/commands/` are committed, so the whole team shares and updates them through version control.
- **C ✘** — Incorrect. `.mcp.json` defines MCP servers, not slash commands.
- **D ✘** — Incorrect. A system prompt isn't a shared, versioned command mechanism.

**Q23.** A skill performs verbose codebase exploration, and you don't want its output to pollute the main conversation. Which skill frontmatter option helps?

- A. `context: fork` to run the skill in an isolated sub-context
- B. `allowed-tools: []`
- C. `model: haiku`
- D. `argument-hint`

**Answer: A**

- **A ✔** — Correct. `context: fork` runs the skill in an isolated sub-context. The verbose exploration stays there and only the result returns, which keeps the main conversation clean.
- **B ✘** — Incorrect. `allowed-tools` restricts which tools a skill can use. It doesn't isolate output, and an empty list would likely cripple an exploration skill.
- **C ✘** — Incorrect. `model` picks which model runs the skill. It doesn't separate context.
- **D ✘** — Incorrect. `argument-hint` only documents the expected arguments in the UI.

**Q24.** Which scenario favours **plan mode** over direct execution?

- A. Fixing a typo in one file
- B. A large migration touching dozens of files with multiple valid approaches and architectural decisions
- C. Adding a single log line
- D. Renaming one variable

**Answer: B**

- **A ✘** — Incorrect. A one-file typo has an obvious solution. Planning adds overhead for no benefit, so direct execution is right.
- **B ✔** — Correct. A large, multi-file migration with several valid approaches and architectural trade-offs benefits from exploring and agreeing a plan before any edits.
- **C ✘** — Incorrect. A single log line is trivial and needs no plan.
- **D ✘** — Incorrect. A one-variable rename is small and well-defined, so execute directly.

**Q25.** A CI pipeline needs Claude Code to run without interactive input and emit machine-parseable output. What do you use?

- A. Run the interactive TUI and scrape the terminal
- B. `claude -p "prompt"` with `--output-format json` (and optionally `--json-schema`)
- C. `claude --resume` with stdin tricks
- D. Use plan mode

**Answer: B**

- **A ✘** — Incorrect. Scraping a terminal UI is brittle and not designed for automation.
- **B ✔** — Correct. `claude -p` runs non-interactively, and `--output-format json` (optionally with `--json-schema`) gives machine-parseable output suited to CI.
- **C ✘** — Incorrect. `--resume` continues an earlier session and isn't a non-interactive mechanism. Stdin tricks are fragile.
- **D ✘** — Incorrect. Plan mode is for planning before execution, not for headless output.

**Q26.** A CI code-review step uses the same session that generated the code and finds few issues. What's the better design?

- A. Ask the same session to "try harder"
- B. Use an independent review instance without the generator's reasoning context, which is less biased toward its own output
- C. Skip review
- D. Use a larger context window

**Answer: B**

- **A ✘** — Incorrect. "Try harder" doesn't remove the session's bias. It still holds the reasoning that produced the code and tends to defend it.
- **B ✔** — Correct. An independent instance sees only the output, without the generator's reasoning context, so it is less anchored to its own choices and finds more real issues.
- **C ✘** — Incorrect. Skipping review removes a quality control and doesn't fix the weakness.
- **D ✘** — Incorrect. A larger context window doesn't change the self-review bias and may add more of the generator's reasoning to anchor on.

**Q27.** Which approach reduces duplicate comments when CI re-reviews a PR after new commits?

- A. Delete previous comments each run
- B. Include the prior review findings in context and instruct Claude to report only new or unresolved issues
- C. Lower the review strictness
- D. Run review only once ever

**Answer: B**

- **A ✘** — Incorrect. Deleting earlier comments erases history and context for reviewers and doesn't stop the model from re-raising the same points.
- **B ✔** — Correct. Supplying prior findings and telling Claude to report only new or unresolved issues directly prevents re-flagging resolved items.
- **C ✘** — Incorrect. Lowering strictness reduces useful findings overall without targeting duplicates.
- **D ✘** — Incorrect. Reviewing once ignores new commits, which defeats the purpose of CI review.

---

## Domain 4: Prompt Engineering and Structured Output

**Q28.** A prompt says "be conservative and flag only high-confidence issues." False positives remain high. What is most effective?

- A. Add "be very conservative"
- B. Replace vague guidance with explicit categorical criteria: what to flag, what to skip, with concrete code examples of each
- C. Ask for a confidence number and filter at 0.9
- D. Increase temperature

**Answer: B**

- **A ✘** — Incorrect. More adjectives ("very conservative") are still vague. The model has no concrete boundary to apply.
- **B ✔** — Correct. Explicit categorical criteria (what to flag, what to skip) with concrete examples of each define the decision boundary, which is what reduces false positives.
- **C ✘** — Incorrect. Self-reported confidence is poorly calibrated, so thresholding at 0.9 filters on a noisy signal.
- **D ✘** — Incorrect. Higher temperature increases randomness and tends to make output less consistent.

**Q29.** When are few-shot examples most valuable?

- A. For trivially clear tasks
- B. For ambiguous cases and judgement calls, showing the reasoning for why one action was chosen over a plausible alternative
- C. Only for formatting JSON
- D. Never alongside tools

**Answer: B**

- **A ✘** — Incorrect. Trivially clear tasks don't need examples, which only add tokens.
- **B ✔** — Correct. Few-shot examples shine on ambiguous cases and judgement calls. Showing why one action was chosen over a plausible alternative teaches the decision boundary.
- **C ✘** — Incorrect. Formatting is one use but far from the only or most valuable one. Examples help with reasoning and edge cases too.
- **D ✘** — Incorrect. Few-shot examples work fine alongside tools, for instance to show when to call which tool.

**Q30.** What does forcing output through `tool_use` with a JSON schema guarantee, and what does it NOT?

- A. Guarantees syntactically valid, schema-conforming JSON. It does not guarantee semantic correctness (e.g., line items summing to the stated total).
- B. Guarantees both syntax and semantics
- C. Guarantees neither
- D. Guarantees semantics but not syntax

**Answer: A**

- **A ✔** — Correct. A forced schema guarantees syntactically valid, schema-conforming JSON. It cannot guarantee the values are right. For example, line items can still fail to sum to the stated total, so you still need semantic validation.
- **B ✘** — Incorrect. Semantics are not guaranteed. A well-formed object can contain wrong or inconsistent data.
- **C ✘** — Incorrect. Syntax and schema conformance are guaranteed, which is the main benefit.
- **D ✘** — Incorrect. It is the reverse: syntax is guaranteed, semantics are not.

**Q31.** A source document sometimes lacks a "contract end date," and the model fabricates one. Which schema change helps most?

- A. Make the field required
- B. Make the field optional or nullable so the model can return null, and instruct it to do so when absent
- C. Make it a free-text string
- D. Remove the field

**Answer: B**

- **A ✘** — Incorrect. Making a field required pressures the model to produce a value even when none exists, which encourages fabrication.
- **B ✔** — Correct. A nullable or optional field gives the model a legitimate way to say "not present". Pair it with an instruction to return null when the information is absent.
- **C ✘** — Incorrect. Free text doesn't stop fabrication and weakens validation and downstream parsing.
- **D ✘** — Incorrect. Removing the field loses data that is legitimately present in other documents.

**Q32.** Extraction output failed validation (the total doesn't match the sum of line items). Which retry strategy is best?

- A. Retry the identical prompt
- B. Retry with the original document, the failed extraction and the specific validation error, so the model can self-correct
- C. Switch models
- D. Give up and route to human review immediately in all cases

**Answer: B**

- **A ✘** — Incorrect. The identical prompt gives the model no new information, so it will likely repeat the mistake.
- **B ✔** — Correct. Giving the model the original document, its failed extraction and the specific validation error lets it see precisely what to correct. This is retry with error feedback.
- **C ✘** — Incorrect. Switching models doesn't address the specific error and adds cost and variability.
- **D ✘** — Incorrect. Many validation failures are fixable by retry. Routing to humans is for cases that can't be resolved automatically.

**Q33.** When will retrying with error feedback NOT help?

- A. Format mismatches
- B. When the required information simply isn't in the source document
- C. Structural errors
- D. Unit normalisation errors

**Answer: B**

- **A ✘** — Incorrect. Format mismatches are the kind of error feedback fixes well, because the model can reformat.
- **B ✔** — Correct. If the information isn't in the source, no amount of feedback can produce it. Retrying only tempts fabrication. Return null or escalate.
- **C ✘** — Incorrect. Structural errors are correctable given a specific error message.
- **D ✘** — Incorrect. Unit normalisation mistakes are fixable once the model is told what's wrong.

**Q34.** Which workload is best suited to the Message Batches API?

- A. A blocking pre-merge check that developers wait on
- B. An overnight job to extract data from 50,000 archived documents
- C. A live chat support bot
- D. An interactive coding assistant

**Answer: B**

- **A ✘** — Incorrect. A blocking pre-merge check needs a prompt response, and batches can take up to 24 hours.
- **B ✔** — Correct. Overnight extraction over 50,000 archived documents is bulk and latency-tolerant, which is ideal for batches and their roughly 50% cost saving.
- **C ✘** — Incorrect. A live chat bot needs real-time responses.
- **D ✘** — Incorrect. An interactive assistant needs low latency and multi-turn interaction.

**Q35.** Which statement about the Message Batches API is TRUE?

- A. It gives about a 50% cost saving, with processing that can take up to 24 hours and no guaranteed latency SLA
- B. It supports multi-turn tool calling within a single request
- C. It returns results in the same order as the requests
- D. It is real-time

**Answer: A**

- **A ✔** — Correct. Batches cost about 50% less, can take up to 24 hours to complete, and have no guaranteed latency SLA.
- **B ✘** — Incorrect. A batch request can't run a multi-turn tool-calling loop, because you can't return tool results mid-request.
- **C ✘** — Incorrect. Results can come back in any order, which is why `custom_id` exists.
- **D ✘** — Incorrect. Batches are asynchronous, not real-time.

**Q36.** In a batch run, results arrive unordered and some requests failed. How do you correlate and recover?

- A. By array index
- B. Using `custom_id` to match results, then resubmit only the failed items
- C. Resubmit the entire batch
- D. Discard failures

**Answer: B**

- **A ✘** — Incorrect. Results are not guaranteed to match request order, so index-based matching can mislabel results.
- **B ✔** — Correct. `custom_id` ties each result to its request regardless of order. Resubmit only the failed items to avoid paying twice.
- **C ✘** — Incorrect. Resubmitting everything wastes cost and time on items that already succeeded.
- **D ✘** — Incorrect. Discarding failures loses data. They are often recoverable, for example by splitting oversized documents.

**Q37.** A single-pass review of a 14-file PR gives inconsistent depth: detailed on some files, shallow on others, and it contradicts itself. What's the best restructuring?

- A. A larger model alone
- B. Per-file local analysis passes, then a separate cross-file integration pass
- C. One giant prompt with more instructions
- D. Randomly sample files

**Answer: B**

- **A ✘** — Incorrect. A larger model doesn't fix the structural cause, which is attention dilution when one pass handles many files.
- **B ✔** — Correct. Per-file passes give each file full attention, and a separate integration pass checks cross-file concerns. That yields consistent depth and avoids contradictions.
- **C ✘** — Incorrect. More instructions in one giant prompt add noise and don't solve attention spreading.
- **D ✘** — Incorrect. Sampling files means some code is never reviewed, which is unacceptable for a PR review.

---

## Domain 5: Context Management and Reliability

**Q38.** In a long support conversation, early facts (order ID, refund amount) get lost as history grows. What is the best mitigation?

- A. Summarise everything into prose each turn
- B. Extract transactional facts into a persistent "case facts" block included in every prompt, outside the summarised history
- C. Raise `max_tokens`
- D. Remove the tool results

**Answer: B**

- **A ✘** — Incorrect. Repeated prose summarisation is lossy and tends to drop exact details such as IDs and amounts.
- **B ✔** — Correct. A persistent case-facts block carries exact transactional data in every prompt, outside the summarised history, so it can't be lost through compression.
- **C ✘** — Incorrect. `max_tokens` limits output length and does nothing for what is kept in input context.
- **D ✘** — Incorrect. Removing tool results deletes the very facts you need.

**Q39.** What is the "lost in the middle" effect, and how do you mitigate it?

- A. Models ignore content at the start. Mitigate by moving it to the end.
- B. Models may under-attend to content in the middle of long inputs. Mitigate by placing key findings at the start or end and using explicit section headers.
- C. Models drop tool calls in the middle of loops
- D. Tokens in the middle cost more

**Answer: B**

- **A ✘** — Incorrect. The effect concerns the middle of long inputs, not the start. Moving content to the end doesn't describe the phenomenon.
- **B ✔** — Correct. Models can under-attend to information buried in the middle of long contexts. Put key findings at the start or end and use explicit section headers to help the model locate content.
- **C ✘** — Incorrect. This isn't about tool calls in loops.
- **D ✘** — Incorrect. Token cost doesn't vary by position.

**Q40.** A tool returns 40 fields per order but the agent needs 5. What should you do?

- A. Pass everything. More context is better.
- B. Trim or filter the tool output to the relevant fields before it accumulates in context
- C. Ask the user to filter
- D. Disable the tool

**Answer: B**

- **A ✘** — Incorrect. Extra fields waste tokens and distract the model. More context is not automatically better.
- **B ✔** — Correct. Trim or filter outputs to the relevant fields before they accumulate in context. This preserves room and focus.
- **C ✘** — Incorrect. Users shouldn't have to filter machine-level data. It's an engineering responsibility.
- **D ✘** — Incorrect. The tool is useful. The fix is to shape its output.

**Q41.** Which is a *reliable* escalation trigger for a support agent?

- A. Negative sentiment detected in the message
- B. The customer explicitly requests a human, or a policy gap or exception exists that the agent can't resolve
- C. The model's self-reported confidence is below 8/10
- D. The conversation exceeds 5 turns

**Answer: B**

- **A ✘** — Incorrect. Sentiment is a poor proxy. A frustrated customer may have an easy issue, and a calm one may have an impossible one.
- **B ✔** — Correct. An explicit request for a human, or a policy gap or exception the agent can't resolve, are concrete, reliable signals that a human is needed.
- **C ✘** — Incorrect. Self-reported confidence is poorly calibrated and unreliable as a trigger.
- **D ✘** — Incorrect. Turn count is arbitrary and unrelated to whether the agent can help.

**Q42.** A customer says "I want to talk to a person" while the agent could resolve the issue. What should the agent do?

- A. Try to resolve first and escalate only if it fails
- B. Honour the request and escalate immediately
- C. Ask them to justify the request
- D. Offer a discount

**Answer: B**

- **A ✘** — Incorrect. Delaying defies the customer's explicit request and risks frustration, even if the agent could have solved it.
- **B ✔** — Correct. An explicit request for a human should be honoured immediately. It's the customer's right and a reliable escalation trigger.
- **C ✘** — Incorrect. Making the customer justify the request is adversarial and delays help.
- **D ✘** — Incorrect. Offering a discount sidesteps the request and may be unauthorised.

**Q43.** A lookup returns multiple customers matching "John Smith." What should the agent do?

- A. Pick the most recently active
- B. Ask for additional identifiers (email, order number) to disambiguate rather than guess
- C. Pick the first result
- D. Escalate immediately in all cases

**Answer: B**

- **A ✘** — Incorrect. Picking by recency is a guess, and acting on the wrong customer's data is a serious error.
- **B ✔** — Correct. Ask for further identifiers such as an email or order number to disambiguate. Don't guess between matches.
- **C ✘** — Incorrect. The first result is arbitrary, and the same risk of acting on the wrong account applies.
- **D ✘** — Incorrect. Escalating always is excessive when a simple clarifying question resolves it.

**Q44.** In a multi-agent pipeline, a subagent's web search times out. What error propagation is best?

- A. Return an empty result so the pipeline continues
- B. Fail the whole pipeline silently
- C. Return structured error context (what was attempted, partial results, alternatives) so the coordinator can retry, route around the failure or flag a coverage gap
- D. Suppress the error and let the synthesiser guess

**Answer: C**

- **A ✘** — Incorrect. An empty result masquerades as success, so the coordinator would assume nothing was found and hide a coverage gap.
- **B ✘** — Incorrect. Failing silently loses all partial work and gives no signal about what went wrong.
- **C ✔** — Correct. Structured error context (what was attempted, partial results, possible alternatives) lets the coordinator retry, route around the failure or flag the gap honestly.
- **D ✘** — Incorrect. Suppressing the error and letting the synthesiser guess invites fabrication.

**Q45.** A synthesised report merges claims from many sources. To preserve attribution, you should:

- A. Strip citations to save tokens
- B. Require subagents to return structured claim-source mappings (URL, excerpt, date) that carry through synthesis, and annotate conflicts rather than silently picking one
- C. Average conflicting values
- D. Keep only the most recent source

**Answer: B**

- **A ✘** — Incorrect. Stripping citations destroys provenance and makes claims unverifiable.
- **B ✔** — Correct. Structured claim-source mappings (URL, excerpt, date) that persist through synthesis preserve attribution. Annotate conflicting claims instead of silently choosing.
- **C ✘** — Incorrect. Averaging conflicting values invents a number that no source states.
- **D ✘** — Incorrect. Keeping only the most recent source discards valid evidence and doesn't handle genuine disagreements.

**Q46.** Your extraction system reports 97% aggregate accuracy, so you plan to automate it fully. What's the risk and the fix?

- A. No risk
- B. Aggregate accuracy can hide poor performance on specific document types or fields. Validate per type and field with stratified sampling before reducing human review.
- C. Increase the batch size
- D. Remove validation

**Answer: B**

- **A ✘** — Incorrect. 97% aggregate accuracy can hide serious failures on specific document types or fields, so there is a risk.
- **B ✔** — Correct. Aggregates mask segment-level problems. Validate per document type and field with stratified sampling before reducing human review.
- **C ✘** — Incorrect. Batch size is unrelated to accuracy validation.
- **D ✘** — Incorrect. Removing validation takes away the safeguard that catches errors.

**Q47.** A very long codebase investigation is degrading as the context fills. Which technique helps?

- A. Keep going
- B. Persist findings to scratchpad files and spawn subagents for specific questions, summarising key results back and using `/compact` where appropriate
- C. Disable tools
- D. Start a new unrelated chat

**Answer: B**

- **A ✘** — Incorrect. Continuing as context fills degrades quality as earlier details are lost.
- **B ✔** — Correct. Persist findings to scratchpad files, delegate focused questions to subagents that return summaries, and use `/compact` where appropriate. This keeps the main context lean while preserving knowledge.
- **C ✘** — Incorrect. Disabling tools prevents the investigation from continuing.
- **D ✘** — Incorrect. A new unrelated chat loses all accumulated findings.

**Q48.** Which approach to confidence-based routing is best?

- A. Trust the model's self-reported confidence as-is
- B. Have the model output field-level confidence, then calibrate those scores against a labelled validation set before using them as routing thresholds
- C. Route everything to humans
- D. Route nothing to humans

**Answer: B**

- **A ✘** — Incorrect. Self-reported confidence is often poorly calibrated, so using it as-is gives unreliable routing.
- **B ✔** — Correct. Field-level confidence calibrated against a labelled validation set turns raw scores into thresholds you can trust for routing.
- **C ✘** — Incorrect. Routing everything to humans removes the efficiency gain of automation.
- **D ✘** — Incorrect. Routing nothing to humans ignores risk and sends errors through unchecked.

---

## Answer Key

| Q | Ans | Q | Ans | Q | Ans | Q | Ans |
|---|---|---|---|---|---|---|---|
| 1 | C | 13 | B | 25 | B | 37 | B |
| 2 | B | 14 | B | 26 | B | 38 | B |
| 3 | B | 15 | B | 27 | B | 39 | B |
| 4 | B | 16 | B | 28 | B | 40 | B |
| 5 | B | 17 | B | 29 | B | 41 | B |
| 6 | B | 18 | B | 30 | A | 42 | B |
| 7 | B | 19 | B | 31 | B | 43 | B |
| 8 | B | 20 | B | 32 | B | 44 | C |
| 9 | B | 21 | B | 33 | B | 45 | B |
| 10 | B | 22 | B | 34 | B | 46 | B |
| 11 | C | 23 | A | 35 | A | 47 | B |
| 12 | C | 24 | B | 36 | B | 48 | B |

---

## Study Tips

- The exam is **scenario-based**. For each scenario (support agent, multi-agent research, Claude Code in CI, data extraction, developer productivity), ask: *what is deterministic, and what is probabilistic?*
- The recurring answer patterns:
  - **Hard rules** go to hooks or code, not prompts.
  - **Vague guidance** gets replaced by explicit criteria and examples.
  - **Subagent context** must be passed explicitly.
  - **Errors** should be structured, and failure must be distinguished from empty.
  - **Bulk, non-urgent work** goes to batches. **Blocking work** uses the synchronous API.
- Practice with Claude Code hands-on (CLAUDE.md, rules, commands, skills, `-p` mode, `.mcp.json`) and build a small agentic loop with the Agent SDK.
- Ask me for more questions on any single domain, a harder set, or a timed mock exam.
