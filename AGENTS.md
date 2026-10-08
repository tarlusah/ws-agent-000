# ws-agent-000 — Agent Factory

This workspace builds agents for **other** workspaces. Nothing here is the end product;
the output always lands outside this directory.

## Standard

Every workspace uses **AGENTS.md** as the single source of truth for agent rules.
`CLAUDE.md` only contains `@AGENTS.md` so Claude Code loads the same rules.

## Structure

```
ws-agent-000/
├── AGENTS.md                          # this file
├── CLAUDE.md                          # imports AGENTS.md
├── .claude/
│   └── agents/
│       └── agent-builder.md           # the agent that creates agents elsewhere
└── templates/
    └── target-workspace/
        ├── AGENTS.md                  # template for a target workspace's AGENTS.md
        ├── CLAUDE.md                  # "@AGENTS.md" import
        └── agent.md                   # optional Claude Code subagent template
```

## How to use

Ask: "Use agent-builder to create an agent in `../ws-agent-001` that <does X>."

## Rules

- Rules and goals for agent-builder live in `.claude/agents/agent-builder.md` — edit that file to evolve it.
- Do not commit or push with git unless a human explicitly asks.
