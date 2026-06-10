# Automation Layer Adviser

A Claude Code-powered GitHub Actions workflow that analyses Jira tickets and recommends the correct test automation layer — automatically, every time a ticket is ready to be worked on.

---

## The problem

When does your team decide what layer to automate a ticket at?

Usually during development. Usually inconsistently, depending on who picks up the ticket. And with no shared record of the reasoning.

That's a timing problem and an information problem. Automation decisions made mid-sprint are harder to act on, harder to review, and harder to learn from.

---

## What this does

When a Jira ticket transitions to Ready, this workflow fires automatically:

1. Reads the ticket — summary, description, acceptance criteria, components, labels
2. Applies a structured five-step decision framework using Claude
3. Posts a formatted recommendation directly as a comment on the Jira ticket
4. Updates the "Automation Required" field on the ticket based on the recommendation

No prompt. No copy-paste. No one needing to remember.

---

## What the recommendation looks like

Every comment contains:

- **Primary layer** — the correct test layer with a one-sentence rationale
- **Decision chain** — five-step YES/NO reasoning, grounded in the ticket content
- **Unit tests** — evaluated independently of the decision chain, even when the primary layer is Integration or E2E
- **What to automate** — specific scenarios with repo paths
- **What still needs manual testing** — never left blank
- **Deferred coverage** — anything blocked by missing infrastructure

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

The adviser always defaults to the lowest appropriate layer.

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

Claude is called via the official [`anthropics/claude-code-action`](https://github.com/anthropics/claude-code-action). The decision framework is passed as a prompt — Claude reasons through it fresh for every ticket. Nothing is hardcoded.

---

## Setup

### 1 — Fork or clone this repo

### 2 — Add GitHub secrets

Go to **Settings → Secrets and variables → Actions** and add:

| Secret | Description |
|---|---|
| `CLAUDE_CODE_OAUTH_TOKEN` | Anthropic Claude Code OAuth token |
| `JIRA_EMAIL` | Your Jira account email |
| `JIRA_API_TOKEN` | Your Jira API token ([generate here](https://id.atlassian.com/manage-profile/security/api-tokens)) |

### 3 — Configure your Jira instance

In `.github/workflows/automation-layer-adviser.yml`, update the env block:

```yaml
JIRA_BASE_URL: "https://your-instance.atlassian.net"
AUTOMATION_FIELD_ID: "customfield_XXXXX"
```

To find your `AUTOMATION_FIELD_ID`: go to any Jira ticket → add `?expand=renderedFields,names` to the URL → search for your field name in the JSON.

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

---

## Why the layer matters

The adviser defaults to the **lowest appropriate layer** because:

- Unit tests are faster, more stable, and cheaper to maintain than E2E tests
- Integration tests catch real wiring bugs without the flakiness of browser automation
- E2E tests should be reserved for true customer journeys — not used as a substitute for missing lower-level coverage

Over-testing at the wrong layer is as much a problem as under-testing.

---

## Design decisions

**Why GitHub Actions and not a Jira-native solution?**
Jira automation rules can call webhooks but can't run AI inference. GitHub Actions provides the compute, secrets management, and audit trail needed to run Claude reliably.

**Why structured JSON output?**
Claude is non-deterministic. Requiring a specific JSON schema with markers makes the output parseable regardless of how Claude phrases its reasoning. Defensive jq transforms handle edge cases where Claude returns a string instead of an array.

**Why post to Jira as a comment rather than updating fields directly?**
The comment is visible to the whole team, shows the full reasoning chain, and is easy to override. Field updates alone don't explain why — the comment does.

**Why is the Jira comment posted under a personal account?**
For production use, replace `JIRA_EMAIL` and `JIRA_API_TOKEN` with a dedicated service account so comments aren't attributed to an individual.

---

## Tech stack

- **GitHub Actions** — workflow orchestration
- **[Claude Code Action](https://github.com/anthropics/claude-code-action)** — Anthropic's official GitHub Action for running Claude Code
- **Jira REST API v3** — comment posting and field updates
- **Atlassian Document Format (ADF)** — structured rich-text comment format
- **Python** — JSON extraction from Claude output
- **jq** — JSON transformation for ADF payload construction

---

## Limitations

- Recommendation quality depends on ticket content — thin tickets with no description or AC produce weaker recommendations
- Claude is non-deterministic — the same ticket may produce slightly different output on different runs
- Third-party integrations (Okta, AD, external identity providers) sometimes over-trigger MANUAL — the framework benefits from team-specific rules for these scenarios
- Comments post under whichever Jira account the API token belongs to

---

## Licence

MIT
