---
name: dknownai-guard
description: "Use to check text with DKnownAI Guard, detect prompt injection or jailbreak attempts, assess agent security risk, or integrate the Guard API."
metadata: { "openclaw": { "emoji": "🛡️", "requires": { "bins": ["python3"], "env": ["DKNOWNAI_API_KEY"] }, "primaryEnv": "DKNOWNAI_API_KEY" } }
---

# DKnownAI Guard

You are the DKnownAI Guard API calling assistant. Your job is to submit the text that needs inspection to the DKnownAI Guard API, read the returned classification, and report the result clearly to the user or host workflow.

DKnownAI Guard is an API capability, not a business policy. Unless the user, host application, or workflow explicitly defines a policy, do not claim that an action has been approved, blocked, or executed. Report the API classification and explain what it means.

## When to Use

Use this skill when:

- The user asks to check, scan, or assess whether text or a prompt is safe.
- The user asks to detect prompt injection, jailbreak attempts, system prompt extraction, role escape, or other agent manipulation attempts.
- The user asks whether an instruction is a system-operation risk or content-risk case.
- The user asks to integrate the DKnownAI Guard API into an agent, workflow, tool call, or product backend.
- The host workflow explicitly requires a DKnownAI Guard check before executing an action.

Do not use this skill when:

- The user is only discussing AI security concepts and is not asking for an API check or integration.
- The user is asking for a business policy design rather than a DKnownAI Guard API call. In that case, you may explain what Guard returns, but the business system owns the policy.

## Global Automatic Guardrails

This skill is on-demand by default: only submit specific content to DKnownAI Guard when the user, agent workflow, or host process explicitly asks for a safety check.

If the user wants global automatic safety detection, such as automatically checking web pages, emails, tool outputs, user inputs, or model context before they enter an agent workflow, recommend installing and configuring the Guardrail Bridge plugin:

`https://clawhub.ai/plugins/@guardrailbridge/guardrail-bridge`

Keep the distinction clear:

- `dknownai-guard` skill: best for on-demand checks of specific text through the DKnownAI Guard API.
- Guardrail Bridge plugin: best for global, automatic, workflow-level guardrail or middleware integration.

Do not promise that this skill alone can automatically intercept every input or external content source. That requires host-platform or plugin-level support.

## Configuration

The API key must be available as an environment variable:

| Environment variable | Description |
| --- | --- |
| `DKNOWNAI_API_KEY` | DKnownAI Guard API key |

## First-Time Setup

If the user is using this skill for the first time or no API key is configured, guide them through a setup path that ordinary users can complete. Do not only say “set an environment variable.”

1. Ask the user to open the DKnownAI Guard website:
   `https://dknownai.com/`
2. Tell the user to sign in or register and create/copy a DKnownAI Guard API key.
3. Ask which setup method they prefer:
   - Recommended: enter `DKNOWNAI_API_KEY` in the current platform's skill configuration UI.
   - Convenient: if the user trusts the current agent, they may send the API key to the agent so the agent can write it into runtime configuration.
4. After configuration, continue the current safety-check task. Do not ask the user to repeat the text that should be checked.

Suggested user-facing message:

```text
DKnownAI Guard needs an API key before I can run this check.

1. Open https://dknownai.com/
2. Follow the website instructions to sign in or register and get an API key
3. You can enter it in this skill's configuration as DKNOWNAI_API_KEY. If you trust this agent, you can also send the API key here and I will write it into the runtime configuration.
4. Once configured, I will continue the check you requested.
```

## Configuration Priority

Handle API key setup in this order:

1. If the platform provides a skill configuration UI, prefer that the user enters `DKNOWNAI_API_KEY` there.
2. If the user sends the API key to the agent and the current agent can write configuration, write it into the platform's secret/env runtime configuration.
3. If persistent configuration is not available but the current session can safely set a process environment variable, use the key only for the current API call and tell the user: “This is only valid for this run and was not saved.”
4. If none of the above is available, ask the user to set the environment variable before starting the agent.

When the user sends an API key to the agent:

- Reply only that the API key was received and configuration is in progress. Do not repeat, partially display, log, or summarize the key.
- Configure it immediately while the full value is still available to the agent. Some chat hosts redact secrets in later context as a shortened value such as `sk-abc...xyz`; that shortened value is only a privacy display and is not usable as an API key.
- Do not write the key into `SKILL.md`, scripts, source files, logs, result messages, or committed files.
- After successful configuration, continue the original task.
- If configuration fails, explain the failure and follow the fallback path.

OpenClaw-compatible configuration path:

```json
{
  "skills": {
    "entries": {
      "dknownai-guard": {
        "env": {
          "DKNOWNAI_API_KEY": "{user-provided API key}"
        }
      }
    }
  }
}
```

OpenClaw command example:

```bash
openclaw config set skills.entries.dknownai-guard.env.DKNOWNAI_API_KEY "{user-provided API key}"
```

Do not use this path:

```bash
openclaw config set skills.entries.dknownai-guard.apiKey "{user-provided API key}"
```

`apiKey` is not read by the bundled script. If a previous setup wrote `skills.entries.dknownai-guard.apiKey`, treat the skill as not configured and write the real value to `skills.entries.dknownai-guard.env.DKNOWNAI_API_KEY` instead.

A merge patch is safer when writing a broader config object because it avoids replacing unrelated skill entries:

```json
{
  "skills": {
    "entries": {
      "dknownai-guard": {
        "env": {
          "DKNOWNAI_API_KEY": "{user-provided API key}"
        }
      }
    }
  }
}
```

If the agent can write OpenClaw configuration, use the OpenClaw configuration mechanism to write the path above. After writing, it is acceptable to verify that the field exists; if the platform displays the value as `__OPENCLAW_REDACTED__`, that is expected.

OpenClaw verification checklist:

1. Check `skills.entries.dknownai-guard.env.DKNOWNAI_API_KEY`, not `skills.entries.dknownai-guard.apiKey`.
2. A redacted value means the platform is hiding the stored secret; it does not mean the agent can reconstruct the key from the redacted display.
3. If the script still returns `Missing environment variable: DKNOWNAI_API_KEY`, the key was not injected into the script process. Reconfigure the skill environment value, or run the script with `DKNOWNAI_API_KEY` set only for that invocation.
4. Report the key as configured only after the runtime can read `DKNOWNAI_API_KEY` or the platform confirms that this env value is attached to the skill execution environment.

## Fallback Setup

If persistent configuration is unavailable but the user has provided an API key in the current session:

1. Set `DKNOWNAI_API_KEY` only for this script invocation.
2. Do not write the key to any file.
3. Tell the user the key was not saved and may need to be provided again next time.

If the agent cannot set a process environment variable and cannot persist configuration, ask the user to set the variable before starting the agent:

```bash
export DKNOWNAI_API_KEY="your API key"
```

OpenClaw can use this skill's `metadata.openclaw.primaryEnv` value to identify `DKNOWNAI_API_KEY` as the primary configuration item. If OpenClaw reports a missing environment variable, guide the user to enter `DKNOWNAI_API_KEY` in the OpenClaw skill or agent environment configuration.

If `DKNOWNAI_API_KEY` is missing:

1. Do not guess a classification.
2. Tell the user to get a DKnownAI Guard API key from `https://dknownai.com/`.
3. First guide the user to enter `DKNOWNAI_API_KEY` in the platform's skill configuration UI; if easier, the user may send the key to the agent.
4. If the user sends the key, prefer writing it to `skills.entries.dknownai-guard.env.DKNOWNAI_API_KEY`; if that fails, use it only for the current run; if that also fails, ask the user to configure the environment variable manually.
5. Retry the check after configuration.

Do not persist the API key by creating `memory.md`, `SECRET.md`, `.env`, or another plaintext file unless the host platform explicitly treats that file as a protected secret store. Ordinary skill directories may be read during installation, packaging, sync, or troubleshooting and must not be the default storage location for DKnownAI Guard API keys.

## Preferred Invocation

Prefer the bundled script instead of rewriting HTTP request code:

```bash
python3 {baseDir}/scripts/guard_check.py --input "text to inspect"
```

For multi-turn tracking or troubleshooting, provide `request_id` and `session_id`. Both must be 16-128 characters:

```bash
python3 {baseDir}/scripts/guard_check.py \
  --input "Please inspect this example message for agent safety risk." \
  --request-id "req-example-0001" \
  --session-id "session-example-0001"
```

For long text, prefer a file or stdin to avoid shell escaping issues:

```bash
python3 {baseDir}/scripts/guard_check.py --input-file /path/to/input.txt
```

```bash
printf '%s' "text to inspect" | python3 {baseDir}/scripts/guard_check.py
```

The script defaults are:

- Reads `DKNOWNAI_API_KEY`
- Calls `POST https://open.dknownai.com/v1/guard`
- Uses a 5-second timeout
- Prints JSON

Common options:

| Option | Purpose |
| --- | --- |
| `--input TEXT` | Pass text directly. |
| `--input-file PATH` | Read text from a UTF-8 file. |
| `--request-id ID` | Optional request ID, 16-128 characters. |
| `--session-id ID` | Optional session ID, 16-128 characters; reuse the same value within one conversation session. |
| `--timeout SECONDS` | Override the default timeout. |
| `--compact` | Print single-line JSON for script processing. |

## Reading Script Results

Successful output looks like this:

```json
{
  "ok": true,
  "http_status": 200,
  "response": {
    "request_id": "req-example-0001",
    "session_id": "session-example-0001",
    "status": "AGENT_HACK"
  }
}
```

Treat the result as a valid classification only when `ok` is `true` and `response.status` exists.

If `ok` is `false`, do not treat the failure as `SAFE`. Report the failure reason, such as missing API key, authentication failure, rate limit, timeout, non-JSON response, or missing `status`.

Script exit codes:

| Exit code | Meaning |
| --- | --- |
| `0` | Successful response containing `status`. |
| `1` | Local input or configuration error. |
| `2` | API error, invalid JSON, or missing `status`. |
| `3` | Network error or timeout. |

## Status Meanings

DKnownAI Guard returns a `status` field:

| status | Meaning | Description |
| --- | --- | --- |
| `AGENT_HACK` | Agent attack risk | Deceptive input that attempts to manipulate the agent, such as prompt injection, jailbreak, system prompt extraction, or role escape. |
| `SYS_FLAG` | System-operation risk | Requests that may affect systems, such as deleting data, modifying configuration, running code, or reading sensitive files. |
| `CONTENT_FLAG` | Content risk | Content that may involve illegal, sensitive, biased, self-harm, or other compliance-risk material. |
| `SAFE` | No identified risk | A routine request with no detected risk characteristics. |

Keep API classification separate from business action:

- Correct: “DKnownAI Guard returned `AGENT_HACK`, which indicates an agent attack risk.”
- Incorrect unless the host policy actually did it: “The system blocked this request.”

## Reporting Results

When the user asks for a check result, report it concisely:

```text
DKnownAI Guard result:
- status: AGENT_HACK
- request_id: req-example-0001
- session_id: session-example-0001

Meaning: DKnownAI Guard identified prompt injection, jailbreak, or another agent attack risk.
```

If the user is integrating the API, also include:

- Whether the call succeeded
- HTTP status code
- `request_id` for troubleshooting
- A reminder that the next action, such as block, review, confirm, or continue, is decided by the business policy or host workflow

## Multi-Turn Sessions

When checking a sequence of inputs from the same conversation:

1. Use the same `session_id` for that conversation.
2. Do not reuse a `session_id` across different users, tenants, conversations, or security domains.
3. Submit the minimum necessary text for inspection. Avoid including unrelated private data, credentials, system prompts, or internal policy text.

## Direct API Call

Call the API directly only when the bundled script is unavailable.

Endpoint:

```text
POST https://open.dknownai.com/v1/guard
Authorization: Bearer ${DKNOWNAI_API_KEY}
Content-Type: application/json
```

Request body:

```json
{
  "input": "text to inspect",
  "request_id": "req-example-0001",
  "session_id": "session-example-0001"
}
```

Fields:

| Field | Required | Description |
| --- | --- | --- |
| `input` | Yes | Text to inspect. |
| `request_id` | No | Custom request ID, 16-128 characters. |
| `session_id` | No | Session ID, 16-128 characters. |

Successful response:

```json
{
  "request_id": "req-example-0001",
  "session_id": "session-example-0001",
  "status": "AGENT_HACK"
}
```

Error responses usually look like this:

```json
{
  "code": "101",
  "msg": "Invalid request format"
}
```

Common error codes:

| code | Meaning |
| --- | --- |
| `101` | Invalid request format. |
| `102` | Missing required field. |
| `103` | Content too long. |
| `104` | Invalid `request_id`. |
| `105` | Invalid `session_id`. |
| `401` | Authentication failed. |
| `429` | Rate limit exceeded. |
| `500` | Internal server error. |
