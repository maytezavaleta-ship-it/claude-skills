# Claude Skills

Shared Claude Code skills for the team. Install any skill by copying its folder to `~/.claude/skills/` on your machine — it becomes available globally in every Claude Code session.

---

## Available Skills

### `/power-ed-prototype`
Builds and maintains the **Power Education** live interactive demo — a standalone HTML prototype that recreates the look, feel, and interaction patterns of the Power Ed AI assistant. Use it to test new feature concepts with customers without touching production.

**Triggers:** "build the prototype", "add a feature to the demo", "update the Power Ed site", "test this concept in the prototype"

---

### `/validate-feature-request`
Validates a customer issue or feature request against real customer signal from VOC org 62, IdeaExchange, Slack, and customer meeting transcripts. Produces a **0–10 validation score**.

**Triggers:** "validate this feature", "check VOC for X", "is this a real problem", "how validated is this", "do customers want this"

---

## How to install a skill

1. Clone this repo (or just download the folder you want):
   ```bash
   git clone https://github.com/maytezavaleta-ship-it/claude-skills.git
   ```

2. Copy the skill folder to your Claude skills directory:
   ```bash
   cp -r claude-skills/power-ed-prototype ~/.claude/skills/
   # or
   cp -r claude-skills/validate-feature-request ~/.claude/skills/
   ```

3. That's it — the skill is now available in every Claude Code session. Invoke it with `/skill-name`.

> **Note:** `~/.claude/skills/` may not exist yet. Run `mkdir -p ~/.claude/skills/` first if needed.

---

## How to update a skill

Pull the latest version and re-copy:
```bash
cd claude-skills && git pull
cp -r power-ed-prototype ~/.claude/skills/
```
