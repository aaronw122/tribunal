# tribunal

An agent skill that runs a three-agent adversarial loop on any question, claim, bug hypothesis, or design decision:

1. **Investigator** researches the question and writes up its findings with evidence.
2. **Devil's Advocate** reads those findings, checks the code itself, and argues against them.
3. **Judge** reads both sides, verifies the evidence on its own, and gives a verdict.

The orchestrator only reads the final verdict. That keeps the main context clean and avoids anchoring on either side.

## Install

```sh
git clone https://github.com/aaronw122/tribunal ~/.claude/skills/tribunal
```

Any agent harness that loads `SKILL.md` skills will pick it up. Other harnesses can use the same command with their own skills directory.

## Usage

```
/tribunal Why is the API returning 500s on the /users endpoint?
/tribunal Should we use WebSockets instead of polling for real-time updates?
/tribunal Is this race condition actually possible in our auth flow?
```

Outputs go to `notes/` in the project root:

- `notes/tribunal-investigation.md`
- `notes/tribunal-challenge.md`
- `notes/tribunal-verdict.md`

Each run overwrites these files.
