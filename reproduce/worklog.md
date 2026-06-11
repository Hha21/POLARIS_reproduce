# POLARIS Reproducibility Work Log

**Paper:** POLARIS: Proactive Optimization & Learning Architecture for Resilient Intelligent Systems (SEAMS 2025)  
**Repo:** https://github.com/prakhar479/POLARIS

---

## 11/06/26

### Getting the system running

**Prerequisites:**
- Docker Desktop running (on WSL, must be started from Windows side)
- Python venv at `.venv/` — activate before running anything
- Gemini API key set as `GEMINI_API_KEY` env var

**Steps to start a run:**

1. Start the SWIM Docker container (if not already running):
   ```bash
   docker run -d --name swim -p 4242:4242 -p 5901:5901 -p 6901:6901 gabrielmoreno/swim
   ```

2. Start the NATS message broker:
   ```bash
   docker run -d --name polaris-nats -p 4222:4222 nats:latest
   ```

3. Start the SWIM OMNeT++ simulation inside the container (must be done manually — the container only auto-starts a VNC desktop, not the simulation itself):
   ```bash
   docker exec -d swim bash -c "cd /headless/seams-swim/swim/simulations/swim && PATH=/opt/omnetpp-5.4.1/bin:\$PATH LD_LIBRARY_PATH=/opt/omnetpp-5.4.1/lib:\$LD_LIBRARY_PATH ./run.sh sim 0"
   ```

4. Start all POLARIS components via the startup script:
   ```bash
   cd polaris_poc
   ./start_polaris_swim_system.sh
   ```
   This creates a tmux session (`polaris-swim`) with one window per component. Attach with:
   ```bash
   tmux attach-session -t polaris-swim
   ```

**Installed missing dependencies (not in original requirements):**
```bash
pip install google-generativeai filterpy
```

---

### Issues encountered

**`gemini-2.0-flash` no longer available (404 NOT_FOUND)**

The paper's codebase uses `gemini-2.0-flash` for the agentic reasoner. As of June 2026, Google has removed this model:

```
404 NOT_FOUND: This model models/gemini-2.0-flash is no longer available.
Please update your code to use a newer model.
```

**Fix:** Updated `polaris_poc/src/config/agentic_reasoner_config.yaml` to use `gemini-2.5-flash` instead, which is the current equivalent free-tier model.

```yaml
# Before
model: "gemini-2.0-flash"
# After
model: "gemini-2.5-flash"
```

The meta-learner (`meta_learner_llm.py`) already defaults to `gemini-2.5-flash` so no change needed there.
