# Repository Access

This note documents Abbey's GitHub access for the AI Meal Planner nutritionist agent repository.

Repository SSH URL:

```text
git@github.com:NewRra/nutritionist-agent.git
```

Local clone path:

```text
/home/forge/.openclaw/workspace/repos/nutritionist-agent
```

Public key added to GitHub:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB4JmHj/ZQBEdgSYkfFdbkPULJd9OzVdqHVLx9mi/FCn abbey-nutritionist-agent-openclaw
```

Local private key path:

```text
/home/forge/.openclaw/workspace/.openclaw/keys/abbey_nutritionist_agent_ed25519
```

Do not commit or share the private key. Use the key only for the intended AI Meal Planner repository access unless the owner explicitly says otherwise.

Use this command prefix for Git operations:

```bash
GIT_SSH_COMMAND='ssh -i /home/forge/.openclaw/workspace/.openclaw/keys/abbey_nutritionist_agent_ed25519 -o IdentitiesOnly=yes -o StrictHostKeyChecking=accept-new'
```
