## Scenario:
A shared internal MCP server is deployed on an enterprise intranet. Teams use it for automation (e.g., tool calls for managing test VMs, secrets, or deployments). There’s no strict per-namespace RBAC, so a user from Team A can see tools from Team B.

A curious or malicious developer from Team A calls “fetch_test_log(team=B)” or “get_secret_keys(namespace=B)” and retrieves data they shouldn’t access.

## Threat Landscape:
When namespaces aren’t isolated and RBAC is weak, accidental or intentional misuse becomes trivial. Internal threat actors exploit assumptions of trust in shared environments.

## Assets (A):
* A01: Cross-team secrets, logs, or infra details.
* A02: Tool invocation results for restricted namespaces.

## Threat Actors (TA):
* TA01: Internal developer or tester.
* TA02: Misconfigured internal tool.

## Security Controls (C):
* C01: Enforce namespace-scoped RBAC per user/team.
* C02: Token scoping – issue different tokens per team.
* C03: Redact logs containing other team data.

## Zones:
* Internal Users (trusted but unisolated)
* MCP Server
* Team Resources

```mermaid
flowchart TD
  classDef threatActor fill:#ffdddd,stroke:#ff0000,color:#000;
  classDef asset fill:#ffffcc,stroke:#999900,color:#000;
  classDef control fill:#ddffdd,stroke:#009900,color:#000;
  classDef system fill:#ffffff,stroke:#000000,color:#000;

  subgraph internal [Shared Intranet]
    style internal stroke:#ff0000,stroke-dasharray: 5 5;
    TA01[TA01: Curious Developer]:::threatActor
    TA02[TA02: Misconfigured Tool]:::threatActor
  end

  subgraph mcp [MCP Server]
    style mcp stroke:#ff0000,stroke-dasharray: 5 5;
    A01[A01: Team B Secrets]:::asset
    C01[[C01 Namespace RBAC]]:::control
  end

  TA01 --> A01
  TA02 --> A01
  C01 -. enforces .-> A01
```