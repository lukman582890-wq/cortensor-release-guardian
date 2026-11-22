# Building Blocks Tested — OpenMinion `0.1.24`

Environment: Windows 11, PowerShell 5.1, Python `3.14.1`, `openminion==0.1.24`, Ollama local model (`qwen2.5:1.5b`) + offline `echo` provider for CI tests.

| Building Block | API / Surface Tested | Status | Notes |
|---|---|---|---|
| **Package Install & CLI** | `pip install openminion`, `openminion --version`, `openminion setup --list-providers` | Worked (with caveat) | Cleanly exposes 19+ providers including `cortensor-portal`, `cortensor-router`, and `ollama`. Note: `openminion` pins `pillow<13`, causing a pip resolver warning alongside `instagrapi` (`pillow>=13`). |
| **Application Runtime** | `APIRuntime.from_config_path(...)` + `Agent(runtime=...)` | Bug Found + Workaround | Clean lifecycle separation, **however** omitting `"default_channel"` in `agents.<id>` causes `Agent.run()` to crash with `RuntimeError: inbound authenticity denied (reason=invalid_channel, mode=warn)` at `requests.py:116` / `security.py:132`. Setting `"default_channel": "cli"` resolves it cleanly. |
| **Custom `@tool` Registration** | `@tool(min_scope="READ_ONLY", idempotent=True)` + `forced_tools` | Worked | All 4 read-only audit tools registered and executed cleanly without interactive approval prompts under `allow_read_only_without_prompt: true`. |
| **Action Policy** | `action_policy.mode = "auto"`, `default_action = "require_confirm"` | Worked | Prevents accidental write/exec effects during read-only repository audits. |
| **Durable Sessions** | `Agent(..., session_id=...)` across `audit` and `followup` | Worked | Session state persisted cleanly under `OPENMINION_DATA_ROOT` and carried audit context into the follow-up turn. |
| **Structured Output** | `output_type=ReleaseVerdict` + `AgentOutputValidationError` | Mixed (Model-dependent) | Works cleanly with strong JSON-following models; small local models (`1.5B`) occasionally wrap JSON in markdown fences, making the `AgentOutputValidationError` fallback essential. |
| **Streaming Progress** | `agent.run_stream(..., on_delta=...)` | Worked | Emits incremental events cleanly to stdout during live CLI runs. |
| **Provider Switching** | `configs/agents.ollama.json`, `configs/agents.cortensor-portal.json`, `configs/agents.echo.json` | Worked | Zero code changes needed to swap between offline CI (`echo`), local real model (`ollama`), and hosted `cortensor-portal`. |
