# Agent Skills

Reusable skills for coding agents.

This repository contains agent instructions for making coding workflows more consistent and providing stronger guidance when repository conventions, IDE semantics, or specialized tooling matter.

## Skills

### `mcp-steroid`

Guidance for using MCP Steroid efficiently and safely with IntelliJ IDEA.

The skill deliberately does **not** route every Java/Kotlin task through IntelliJ. Cheap source reading, filename lookup, literal search, simple navigation, and exploratory usage searches should stay with ordinary agent tools. MCP Steroid is used when IntelliJ semantics or IDE capabilities materially improve correctness, including:

- semantic and load-bearing usage analysis;
- rename, move, change-signature, safe-delete, and other structured refactors;
- hierarchy, override, implementation, and overload analysis;
- Java/Kotlin cross-language references;
- generated/light PSI such as Lombok members;
- inspections and debugger workflows;
- IntelliJ project and module models.

It also covers search scoping, generated-Kotlin budgets, IntelliJ threading and EDT requirements, mutation verification, timeout recovery, and choosing between ordinary text search and semantic PSI operations.

**Requires:** MCP Steroid/devrig connected to the agent and a supported IntelliJ IDE.

### `java-conventions`

An opinionated Java code-style and design skill.

It defines conventions for:

- final locals and getter extraction;
- variable grouping and spacing;
- guards and control-flow layout;
- local, field, class, and method naming;
- method-name word-count parity;
- fluent vs. JavaBean accessor consistency;
- package/module organization;
- Lombok usage;
- singleton-service patterns;
- validation of newly introduced or modified code.

When a repository provides `+agents/java-code-style.md` and/or `+agents/java-design-patterns.md`, those repository-specific guides take precedence.

Unlike `mcp-steroid`, this skill is intentionally opinionated and reflects my preferred Java conventions rather than general Java requirements.

## Installation

Clone the repository or copy individual skill directories into your agent's skills directory.

For Kilo CLI, the default setup can use:

```text
~/.config/kilo/skills
```

Configure Kilo to discover that directory:

```json
{
  "skills": {
    "paths": [
      "~/.config/kilo/skills"
    ]
  }
}
```

Other coding agents that support compatible skill files can use the skill directories according to their own installation and discovery mechanisms.

For `mcp-steroid`, configure MCP Steroid as an MCP server for your agent. A local devrig configuration can use:

```json
{
  "mcp": {
    "mcp-steroid": {
      "type": "local",
      "command": [
        "devrig",
        "mcp"
      ]
    }
  }
}
```

Refer to the upstream MCP Steroid/devrig project for current installation and IDE requirements.

## Usage

Skills should be activated when their descriptions match the current task.

They provide policy and operating guidance rather than replacing normal agent judgment. In particular, `mcp-steroid` follows a simple principle:

```text
cheap discovery -> narrow candidates -> semantic verification -> mutation
```

Use text for text, PSI for semantics, and IntelliJ refactoring APIs for structural mutations.
