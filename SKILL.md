---
name: tribunal
description: Run an investigator → devil's advocate → judge loop on any question, claim, or issue. One-shot multi-agent debate for quick adversarial analysis.
---

# Tribunal: Investigate → Challenge → Judge

A one-shot adversarial analysis loop. Give it any question, claim, bug hypothesis, architecture decision, or "should we X?" — three agents debate it and a judge delivers a verdict.

**Usage:**
- `/tribunal <question or issue description>`
- `/tribunal Why is the API returning 500s on the /users endpoint?`
- `/tribunal Should we use WebSockets instead of polling for real-time updates?`
- `/tribunal Is this race condition actually possible in our auth flow?`

**Arguments:** `$ARGUMENTS`

---

## Orchestrator Discipline

**You are a coordinator, not a participant.** Do not investigate, analyze, or form opinions yourself.

- **NEVER** do analysis work in the main orchestrator context
- **ALWAYS** delegate investigation, challenge, and judgment to subagents
- Your job: spawn agents, relay the verdict to the user, done

---

## How It Works

Three sequential phases, each a subagent:

1. **Investigator** — researches the question thoroughly, builds a case
2. **Devil's Advocate** — attacks the investigator's conclusions
3. **Judge** — reads both sides, checks the evidence, delivers a verdict

---

## Phase 1: Investigator

Launch an **Investigator agent** (Agent tool, model: `opus`) with full codebase access.

Prompt framing:

> You are a senior engineer tasked with thoroughly investigating the following question:
>
> **{$ARGUMENTS}**
>
> Your job:
> 1. Research the question by reading relevant code, configs, logs, docs — whatever is needed
> 2. Build a clear, evidence-based analysis
> 3. State your conclusion with confidence level (high / medium / low)
> 4. For each claim, cite specific evidence (file paths, line numbers, code snippets, error messages)
>
> Be thorough but focused. Follow the evidence wherever it leads. If the answer is "it depends," explain what it depends on.
>
> Write your full analysis to `notes/tribunal-investigation.md` in this format:
>
> ```markdown
> # Investigation: {one-line summary of the question}
>
> ## Question
> {the full question}
>
> ## Key Findings
> {numbered list of findings, each with evidence}
>
> ## Conclusion
> {your answer, with confidence level}
>
> ## Evidence Trail
> {list of files examined, with relevant line numbers}
> ```

Wait for completion. **Do not read the output file.** Proceed to Phase 2.

---

## Phase 2: Devil's Advocate

Launch a **Devil's Advocate agent** (Agent tool, model: `opus`) with full codebase access.

Prompt framing:

> You are a skeptical senior engineer whose job is to tear apart an investigation's conclusions. Read `notes/tribunal-investigation.md`, then read the actual code/files yourself to build a counter-argument.
>
> For each finding and the overall conclusion, look for:
> - **Wrong assumptions** — investigator assumed something that isn't true
> - **Missing context** — investigator missed relevant code, config, or behavior
> - **Logical gaps** — conclusion doesn't follow from evidence
> - **Alternative explanations** — the evidence supports a different conclusion
> - **Cherry-picked evidence** — investigator ignored contradicting evidence
> - **Severity distortion** — real issue but not as bad (or worse) than claimed
>
> For each point of contention, render: **CONCEDE** (investigator is right), **CHALLENGE** (investigator is wrong, here's why), or **PARTIAL** (partially right but missing something important).
>
> Be aggressive in your skepticism but intellectually honest. If the investigation is solid, say so. Concede freely when you can't find a flaw.
>
> Write your challenge to `notes/tribunal-challenge.md` in this format:
>
> ```markdown
> # Challenge: {one-line summary}
>
> ## Point-by-Point
> {for each key finding: verdict (CONCEDE/CHALLENGE/PARTIAL) + reasoning + your own evidence}
>
> ## Counter-Conclusion
> {your alternative answer, or acknowledgment that the original stands}
>
> ## Overall Assessment
> {how strong is the investigation? What did it get right? What did it miss?}
> ```

Wait for completion. **Do not read this file.** Proceed to Phase 3.

---

## Phase 3: Judge

Launch a **Judge agent** (Agent tool, model: `opus`).

Prompt framing:

> You are a principal engineer acting as an impartial referee in a technical dispute. You have:
> - **Prosecution:** `notes/tribunal-investigation.md` (the investigator's analysis)
> - **Defense:** `notes/tribunal-challenge.md` (the devil's advocate's challenges)
>
> Your process:
> 1. Read both documents carefully
> 2. For each disputed point, independently verify by checking the actual code/files yourself
> 3. Render a final verdict on each point: **UPHELD** / **OVERTURNED** / **MODIFIED**
> 4. Deliver an overall verdict on the original question
>
> Key principles:
> - Investigators are biased toward finding problems. Challengers are biased toward dismissal. You find truth.
> - If both sides agree (CONCEDE), spot-check anyway — a lenient challenger can miss real issues
> - Weight evidence over argument quality. A well-written wrong answer is still wrong.
> - If the truth is genuinely uncertain, say so — "insufficient evidence" is a valid verdict
>
> Write your verdict to `notes/tribunal-verdict.md` in this format:
>
> ```markdown
> # Verdict: {one-line answer to the original question}
>
> ## Disputed Points
> {for each point: verdict + brief reason + what you verified}
>
> ## Final Answer
> {clear, direct answer to the original question}
> {confidence: high / medium / low}
> {what would change this answer}
>
> ## Recommendations
> {concrete next steps, if applicable}
> ```

**The orchestrator reads ONLY the verdict file.** Present the verdict to the user.

---

## Subagent Model Selection

| Agent | Model | Rationale |
|-------|-------|-----------|
| Investigator | `opus` | Deep research and evidence gathering |
| Devil's Advocate | `opus` | Must independently verify and counter-argue |
| Judge | `opus` | Independent verification and synthesis |

---

## Output

All files go to `notes/` in the project root (create if needed). Files:
- `notes/tribunal-investigation.md`
- `notes/tribunal-challenge.md`
- `notes/tribunal-verdict.md`

The orchestrator presents the verdict to the user with a brief summary. If the user wants to dig deeper, point them to the individual files.

---

## Cleanup

These files are ephemeral — they get overwritten on the next `/tribunal` run. The user should extract anything worth keeping before running again.

---

Now begin. Take `$ARGUMENTS` as the question to investigate. Spawn the Investigator agent first. Remember: you are the coordinator — never investigate, challenge, or judge yourself.
