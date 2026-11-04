# Business Flow: Automated Test Case Pipeline

End-to-end process from tester to production deploy. Business view, no infrastructure details.

---

## Main Flow

```mermaid
flowchart TD
    Tester["Tester"] -->|"Submits test case link<br/>via n8n form"| Form["n8n Form<br/>Entry Point"]

    Form --> MCP["AI Agent reads test case<br/>via MCP<br/>steps, environment, where to run"]

    MCP --> Issue["GitHub Issue created<br/>on team board<br/>labeled auto-test"]

    Issue --> Slack1["Slack: new task<br/>announced"]

    Issue --> Agent["AI Agent picks up issue<br/>writes automated test<br/>self-code review"]

    Agent --> PR["Pull Request opened<br/>test code + review notes"]

    PR --> Slack2["Slack: PR ready<br/>for review<br/>@automation lead tagged"]

    PR --> Lead["Automation Lead<br/>reviews PR"]

    Lead -->|"Approved"| Merge["PR merged"]
    Lead -->|"Changes requested"| Agent

    Merge --> Slack3["Slack: PR completed"]

    Merge --> Jenkins["Jenkins<br/>deploys to server"]

    Jenkins --> Slack4["Slack: deployed<br/>pipeline done"]
```

---

## Roles

| Role | What they do |
|------|-------------|
| **Tester** | Fills the form, pastes test case link. Done. |
| **AI Agent** | Reads the test case via MCP, creates the issue, writes the test, self-reviews, opens the PR. |
| **Automation Lead** | Reviews the PR. Approves or sends back. |
| **Jenkins** | Deploys after merge. Only role outside the dev loop. |

---

## Status Transitions on the Board

```mermaid
stateDiagram-v2
    [*] --> New: Tester submits link
    New --> InProgress: Agent starts work
    InProgress --> InReview: PR opened
    InReview --> Done: Lead approves + merge
    InReview --> InProgress: Changes requested
    Done --> Deployed: Jenkins finishes
    Deployed --> [*]
```

---

## Slack Notification Moments

1. **New task** — issue created, who submitted, link to test case.
2. **PR ready** — PR link, automation lead mentioned.
3. **PR completed** — merged, who approved.
4. **Deployed** — Jenkins result, success or failure.

All messages use Slack blocks: title, context, button to the relevant GitHub page.
