# Handlebar Skills

This repository contains skills to setup and configure [Handlebar][handlebar-website]
governance on a user's AI agents.
It contains the following skills:

| Skill | Description |
| --- | --- |
| [handlebar-setup](./skills/setup) | Connect an AI agent in code to Handlebar governance platform |
| [handlebar-rule-generation](./skills/rule_generation) | Generate governance rules for an AI agent |

## Using these skills

Add these skills locally by running

```bash
npx skills add gethandlebar/agent-skills
```

To get started, you will need to be in a codebase that has an agent defined in code (i.e. not claude code itself).
1. Run `/handlebar-setup` to connect the agent to Handlebar
2. Run `/handlebar-rule-generation` to apply relevant rules and controls to your agent 

[handlebar-website]: https://gethandlebar.com
[handlebar-platform]: https://app.gethandlebar.com
