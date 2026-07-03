# Automation Layer Adviser

> **Related repos** — This is one half of a two-repo project:
> - 🤖 **test-strategy-adviser** (you are here) — the agent: a GitHub Action that recommends test-automation layers for engineering tickets.
> - 🧪 [**prompt-eval-gate**](https://github.com/aastha0208/prompt-eval-gate) — the evaluation system that keeps this agent honest: scores every prompt change against a labelled dataset, blocks regressions in CI, and grows the dataset from human corrections.

The Automation Layer Adviser is a Claude Code-powered GitHub Actions workflow that analyses Jira tickets and recommends the correct test automation layers automatically, every time a ticket is ready to be worked on.

> **Shift-left automation decisions with AI.** When an engineering ticket transitions to Ready state (ready for a sprint), Claude analyzes it and recommends the exact test layer to automate—posted as a comment, grounded in the decision framework aligned to products or teams' established automation guidelines, with no manual intervention required.

---

## The problem

When does your team decide what layer to automate a ticket at?

Usually during development or after the fact. Usually inconsistent, depending on who picks up the ticket. And with no shared record of the reasoning.

That is a timing and an information problem. Automation decisions made mid-sprint are harder to act on, harder to review, and harder to learn from.

## The insight

Most teams pick test layers during dev (too late), inconsistently (depends on who picks up the ticket), and with no audit trail (hard to learn from decisions).

**This tool shifts that decision left to Ready state** — when the team can discuss it before coding starts, grounding every choice in the same framework regardless of who picks up the ticket across teams.

---

## What you get

- ✅ Automated recommendations on every ticket (no one forgets)
- ✅ Consistent framework across all tickets
- ✅ Auditable decision chain (Claude shows its reasoning)
- ✅ Shifts discussion left (before dev starts, not during)
- ✅ ~$0.01 per recommendation (Claude Sonnet pricing)

---

## What this does

When a Jira ticket transitions to Ready, this workflow fires automatically:

1. Reads the ticket: summary, description, acceptance criteria, components, labels
2. Applies a structured five-step decision framework using Claude
3. Posts a formatted recommendation directly as a comment on the Jira ticket
4. Updates the "Automation Required" field on the ticket based on the recommendation

No prompt. No copy-paste. No one needing to remember.

---

## Real example

Here's what the Automation Layer Adviser recommendation looks like on a real Jira ticket:

![Automation Layer Adviser Jira comment - real example](https://github.com/aastha0208/test-strategy-adviser/blob/main/JIRA%20example%20with%20Test%20Adviser.png)

**What you see in the comment:**

- **Primary layer: UNIT** — The recommendation at a glance
- **Rationale** — One sentence explaining why this layer fits the ticket
- **Supporting** — Additional layers recommended as supporting coverage (in this example: INTEGRATION)
- **Skip** — Layers explicitly ruled out and why (in this example: BACKEND_E2E and UI_E2E for a single-transform fix)
- **Decision chain** — Five-step YES/NO reasoning grounded in the ticket content:
  - Step 1: Can unit tests cover it? Analysis of whether this is pure logic testable in isolation
  - Step 2: Does it need real service wiring or DB? If yes, stops here; if no, continues
- The comment is formatted, clear, and immediately actionable for the engineer

---

## Test layers

| Layer | When to use |
|---|---|
| `UNIT` | Pure logic, no live dependencies |
| `INTEGRATION` | Real service wiring, real DB (e.g. Testcontainers) |
| `BACKEND_E2E` | Full API flows with real external dependencies |
| `UI_E2E` | True end-to-end customer journeys only — never for rendering bugs |
| `COMPONENT` | Frontend rendering or state in isolation |
| `MANUAL` | Real hardware, external directory services, exploratory, UX judgment |

The adviser always defaults to the **lowest appropriate layer**.

---

## Why this approach?

- **Claude Code Action** — Uses Anthropic's official GitHub Action for reliable AI inference in workflows
- **Structured JSON schema** — Claude outputs JSON, making output parseable despite non-determinism
- **ADF formatting** — Comments are rich, visible to whole team, easy to override
- **Jira automation trigger** — Fires automatically on ticket transition (no manual prompts)

---

## How it works

```
Jira ticket → Ready
       │
       ▼
Jira Automation
(Send web request)
       │
       ▼
GitHub Actions workflow_dispatch
       │
       ▼
anthropics/claude-code-action
Claude reads ticket + applies decision framework
       │
       ▼
Structured JSON output parsed
       │
       ├──▶ ADF comment posted to Jira ticket
       └──▶ "Automation Required" field updated
```

Claude is called via the official [`anthropics/claude-code-action`](https://github.com/anthropics/claude-code-action). The decision framework is passed as a prompt — Claude reasons through it following a five-step chain, outputs structured JSON, and the workflow parses and formats the result for posting to Jira.

---

## Quick start (2 minutes)

1. **Fork this repo** (or use it as a template)
2. **Add 3 secrets** to your fork (Settings → Secrets and variables → Actions):
   - `CLAUDE_CODE_OAUTH_TOKEN` — [Get from Anthropic](https://claude.ai/account/settings/auth)
   - `JIRA_EMAIL` — Your Jira account email
   - `JIRA_API_TOKEN` — [Generate here](https://id.atlassian.com/manage-profile/security/api-tokens)
3. **Update 2 values** in `.github/workflows/automation-layer-adviser.yml` (lines 159–160):
   - `JIRA_BASE_URL` — Your Jira instance URL
   - `AUTOMATION_FIELD_ID` — Your custom field ID (see [Setup §3](#3--configure-your-jira-instance))
4. **Test it** — Go to Actions → Automation Layer Adviser → Run workflow → fill in a ticket
5. **See the recommendation** posted as a comment on your Jira ticket

That's it. Cost: ~$0.01 per recommendation (Claude Sonnet pricing).

---

## Setup

### 1 — Fork or clone this repo

### 2 — Add GitHub secrets

Go to **Settings → Secrets and variables → Actions** and add:

| Secret | Description |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | [Anthropic Claude Code OAuth token](https://claude.ai/account/settings/auth) |
| `JIRA_EMAIL` | Your Jira account email |
| `JIRA_API_TOKEN` | [Generate here](https://id.atlassian.com/manage-profile/security/api-tokens) |

### 3 — Configure your Jira instance

In `.github/workflows/automation-layer-adviser.yml`, update the env block (around line 159):

```yaml
JIRA_BASE_URL: "https://your-instance.atlassian.net"
AUTOMATION_FIELD_ID: "customfield_XXXXX"
```

**To find your `AUTOMATION_FIELD_ID`:**
1. Open any Jira ticket in your project
2. Add `?expand=renderedFields,names` to the URL
3. Open browser DevTools → Network tab
4. Look at the response JSON for your field name (e.g. `"Automation Required"`)
5. Find its `id` — it will look like `customfield_10042`

### 4 — Trigger manually (test it)

Go to **Actions → Automation Layer Adviser → Run workflow** and fill in a ticket key with its details.

### 5 — Connect Jira automation (optional)

To fire automatically on ticket transitions:

1. Go to your Jira project → **Project settings → Automation → Create rule**
2. Trigger: **Issue transitioned** → To status: `Ready`
3. Action: **Send web request**
   - URL: `https://api.github.com/repos/YOUR_USERNAME/YOUR_REPO/actions/workflows/automation-layer-adviser.yml/dispatches`
   - Method: `POST`
   - Headers: `Authorization: Bearer YOUR_GITHUB_PAT`, `Accept: application/vnd.github+json`
   - Body:
     ```json
     {
       "ref": "main",
       "inputs": {
         "issue_key": "{{issue.key}}",
         "summary": "{{issue.summary}}",
         "issue_type": "{{issue.issueType}}",
         "components": "{{issue.components.name}}",
         "labels": "{{issue.labels}}",
         "team": "{{issue.customfield_XXXXX.name}}",
         "description": "{{issue.description}}",
         "acceptance_criteria": "{{issue.customfield_XXXXX}}"
       }
     }
     ```

---

## Customising the decision framework

The prompt in the workflow contains the full decision framework. To adapt it to your team's guidelines:

- Replace the five-step decision flow with your own rules
- Update the test layer descriptions and repo paths to match your project structure
- Add team-specific rules under "Key rules"
- Add worked examples to the prompt to improve accuracy on edge cases (few-shot prompting)

See `.github/workflows/automation-layer-adviser.yml` lines 97–150.

---

## Why the layer matters

The adviser defaults to the **lowest appropriate layer** because:

- Unit tests are faster, more stable, and cheaper to maintain than E2E tests
- Integration tests catch real wiring bugs without the flakiness of browser automation
- E2E tests should be reserved for true customer journeys — not used as a substitute for missing lower-level coverage

Over-testing at the wrong layer is as much a problem as under-testing.

---

## Troubleshooting

### Workflow runs but no comment appears on Jira

1. Check the Actions log for error messages
2. Verify `JIRA_API_TOKEN` is still valid (tokens expire)
3. Confirm `AUTOMATION_FIELD_ID` matches your Jira instance
4. Check Jira instance URL is correct (typos in `JIRA_BASE_URL`)

### "Could not extract JSON from Claude output"

This means Claude didn't follow the schema. Check the Actions log for Claude's raw response:
- Look for the step "Post comment to Jira"
- If frequent, try adding a worked example to the prompt (lines 97–150 of the workflow)

### Recommendations seem off for my domain

The decision framework is generic — it works well for most software engineering tickets but may need tuning for specialized domains (hardware, data pipelines, etc.).

**Solution:** Edit the decision framework in `.github/workflows/automation-layer-adviser.yml` (lines 97–150) to add your domain-specific rules and examples.

---

## Design decisions

**Why GitHub Actions and not a Jira-native solution?**
Jira automation rules can call webhooks but can't run AI inference. GitHub Actions provides the compute, secrets management, and audit trail needed to run Claude reliably.

**Why structured JSON output?**
Claude is non-deterministic. Requiring a specific JSON schema with markers makes the output parseable regardless of how Claude phrases its reasoning. Defensive jq transforms handle edge cases where Claude varies phrasing.

**Why post to Jira as a comment rather than updating fields directly?**
The comment is visible to the whole team, shows the full reasoning chain, and is easy to override. Field updates alone don't explain why — the comment does.

**Why is the Jira comment posted under a personal account?**
For production use, replace `JIRA_EMAIL` and `JIRA_API_TOKEN` with a dedicated service account so comments aren't attributed to an individual.

---

## Tech stack

- **GitHub Actions** — workflow orchestration (chosen over Lambda for git context access)
- **[Claude Code Action](https://github.com/anthropics/claude-code-action)** — Anthropic's official GitHub Action for running Claude Code
- **Jira REST API v3** — enables audit trail & field updates
- **Atlassian Document Format (ADF)** — structured rich-text comment format
- **Python** — JSON extraction from Claude output
- **jq** — JSON transformation for ADF payload construction

---

## Limitations

- **Recommendation quality depends on ticket content** — thin tickets with no description or acceptance criteria produce weaker recommendations. Keep tickets detailed.
- **Claude is non-deterministic** — the same ticket may produce slightly different output on different runs (though the primary layer usually stays consistent).
- **Third-party integrations over-trigger MANUAL** — Okta, Active Directory, and external identity providers sometimes recommend MANUAL even when automation is possible. Update the decision framework to handle your specific integrations.
- **Comments post under the API token's account** — use a dedicated service account for cleaner audit trails.

---

## Future improvements

- [ ] Multi-step reasoning (Claude can iterate on recommendations)
- [ ] Learning from overrides (track when teams disagree, improve prompts)
- [ ] Cross-team analytics dashboard (visualize automation patterns across all tickets)
- [ ] Support for additional ticket systems (Azure DevOps, GitHub Issues)

---

## License

MIT
