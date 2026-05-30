# Ops Agent Hub (n8n + Jira + Microsoft Teams + Confluence + AI Agent)

This document implements the “Ops Agent Hub” plan as a practical, import-and-configure project.

## 1) n8n setup

Use one of:

- **n8n Cloud** (fastest path)
- **Self-hosted Docker**

### Required credentials in n8n

Create and test credentials before activating the workflow:

1. **Jira Software Cloud**
   - API email + API token
   - Base URL (for example `https://your-company.atlassian.net`)
2. **Confluence Cloud**
   - API email + API token
   - Same Atlassian base URL
3. **Microsoft Teams**
   - Incoming webhook URL for channel notifications
   - Optional Microsoft Graph credentials for richer interactive actions
4. **OpenAI / AI provider**
   - API key
   - Model name

## 2) Core workflow behavior

The workflow file is at:

`workflows/incident-response-agent.workflow.json`

### Trigger

- **Webhook** endpoint receives incident request from Teams/chatops form.

### Context collection

- Search Jira for open issues linked to the service/component.
- Search Confluence for matching runbook/runbook section.

### Agent step

- AI node summarizes context and proposes:
  - incident summary
  - recommended actions
  - priority
  - suggested owner

### Action step

- Create Jira issue automatically (you can extend with update logic based on ticket lookup).
- Post adaptive summary message into Teams.
- Create/update Confluence incident page.

## 3) Human-in-the-loop controls

The workflow includes an approval branch:

- **Approve plan**
  - Transition Jira ticket to “In Progress”
  - Assign suggested owner
  - Post approval confirmation to Teams
- **Escalate**
  - Set Jira priority to highest
  - Notify on-call channel in Teams
  - Mark Confluence page as escalated

## 4) Production hardening

Implemented in the workflow structure:

- Error trigger workflow path for failures
- Retry policy in HTTP/API nodes
- Audit log write (JSON line record) for actions and approver
- Credential scoping via dedicated service accounts
- Webhook token guard to prevent unauthorized calls

## 5) Live project: Incident Response Agent

End-to-end flow:

1. Engineer posts incident in Teams form.
2. n8n receives webhook payload.
3. n8n creates/updates Jira issue.
4. n8n fetches Confluence runbook.
5. AI generates triage + resolution checklist.
6. n8n posts action summary to Teams for approval/escalation.
7. On closure, n8n updates Confluence with postmortem draft.

## 6) Required placeholder updates

Before go-live, replace placeholders in the workflow JSON:

- `ATLASSIAN_BASE_URL`
- `JIRA_PROJECT_KEY`
- `JIRA_TRANSITION_IN_PROGRESS_ID`
- `TEAMS_WEBHOOK_URL`
- `CONFLUENCE_SPACE_KEY`
- `OPENAI_MODEL`

Set environment variable in n8n for secure token validation:

- `WEBHOOK_SHARED_TOKEN`

## 7) Jira transition ID note

`JIRA_TRANSITION_IN_PROGRESS_ID` is Jira-instance specific.  
Get the correct value using:

`GET /rest/api/3/issue/{issueKey}/transitions`

Then update the environment variable in n8n before activation.

## 8) Validation checklist

- Webhook test returns 200 and creates execution
- Jira issue is created/updated correctly
- Teams message is delivered to target channel
- Confluence page is created or updated
- Approval and escalation branches both execute as expected
- Error path generates alert on intentional failure test
