# DKnownAI Guard Skill

`dknownai-guard` is an agent skill for on-demand safety checks with the DKnownAI Guard API.

It helps an agent submit selected text to DKnownAI Guard and interpret returned classifications such as prompt injection, jailbreak, system-operation risk, content risk, or safe content.

## Files

- `SKILL.md`: Skill instructions for agent platforms such as OpenClaw, ClawHub, and SkillHub.
- `scripts/guard_check.py`: Standard-library Python helper for calling the DKnownAI Guard API.
- `_meta.json`: Skill package metadata.

## Configuration

Get a DKnownAI Guard API key from:

https://dknownai.com/

Configure it as:

```bash
export DKNOWNAI_API_KEY="your API key"
```

Or enter `DKNOWNAI_API_KEY` in the host platform's skill configuration UI.

## Usage

```bash
python3 scripts/guard_check.py --input "text to inspect"
```

For long text:

```bash
python3 scripts/guard_check.py --input-file /path/to/input.txt
```

## Distribution

The skill is published on ClawHub as `dknownai-guard`.

Data operations and daily public metrics are tracked separately at:

https://github.com/dylanzhangzx/dknownai-guard-data-ops

## License

MIT-0
