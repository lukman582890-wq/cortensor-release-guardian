# What We Learned — Actionable Feedback for OpenMinion 0.1 & Cortensor Builders

1. **Bug Found (`openminion 0.1.24`): Missing `default_channel` Crashes Programmatic `Agent.run()`:**
   - **Location:** `openminion/services/runtime/ingress/requests.py:116` & `openminion/services/gateway/security.py:132`
   - **Reproduction:** Create a minimal `agents.json` (like the example in `README.md` without `default_channel`), instantiate `APIRuntime.from_config_path(...)` + `Agent(runtime=runtime, ...)` and call `agent.run("...")`.
   - **Actual behavior:** `Agent._build_payload()` does not pass `"channel"`, `agent_profile.default_channel` resolves to `""`, and `GatewaySecurity.enforce_inbound_authenticity()` raises `RuntimeError: inbound authenticity denied (reason=invalid_channel, mode=warn)` even in `warn` mode.
   - **Expected behavior:** Programmatic `Agent.run()` calls should default `channel` to `"api"` or `"cli"` when `default_channel` is omitted in `agents.json`.
   - **Workaround:** Explicitly set `"default_channel": "cli"` under each `agents.<id>` profile. Our `check_openminion_config` tool now automatically flags this for other Cortensor builders.

2. **Windows PowerShell Bootstrap Friction:**
   - `docs/getting-started.md` leans heavily on POSIX `make` / `export` commands. Adding a first-class 4-line Windows PowerShell snippet (using `$env:OPENMINION_HOME = $PWD` and `python -m pip install -e .`) would immediately cut onboarding time for Windows builders.

3. **Dependency Range Strictness (`pillow`):**
   - Installing `openminion 0.1.24` into an existing Python media/automation environment triggered a `pillow` version conflict (`pillow==12.3.0`). Isolating image-only dependencies behind an optional extra (e.g., `openminion[vision]`) would make the core Python SDK lighter and conflict-free.

4. **Clearer Config Validation for `action_policy` vs `permission_mode`:**
   - Builders naturally try placing `permission_mode` inside `agents.<id>` in `agents.json`, even though it is only valid at runtime/session level while `action_policy` lives in config. Emitting a loud config validation warning on unknown keys prevents silent misconfiguration.

5. **Cortensor Portal Retry Ergonomics:**
   - `openminion setup --list-providers` warns that `cortensor-portal` requests do not automatically retry on transient latency/timeouts. Offering an opt-in `retry_policy` block inside `agents.<id>` (`max_retries`, `backoff_seconds`) would make Portal-backed hackathon demos far more reliable.
